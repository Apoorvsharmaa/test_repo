# Chapter 3: Application Entry Point and Initialization

## The Application Entry Point

Every Flutter application starts with a `main()` function. Let's explore how this app initializes.

### Location

The main entry point is located at: **`lib/main.dart`**

### The main() Function

Here's the complete initialization sequence:

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await MySharedPrefernces.init();
  final appDocumentDir = await path_provider.getApplicationDocumentsDirectory();
  Hive.init(appDocumentDir.path);
  // Register adapter
  Hive.registerAdapter(ContactModelAdapter());
  await Hive.openBox<ContactModel>('contactsData');
  await handlePermissions();
  await initiallizedLocalNotification();
  await BottomSheetControllers().grantedPermission();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  try {
    userData = await MySharedPrefernces().getUserData();
  } catch (e) {
    log("Error fetching user data from shared preferences");
  }
  try {
    await updateUserData(userData["id"]);
    userData = await MySharedPrefernces().getUserData();
  } catch (e) {}
  runApp(const MyApp());
}
```

## Initialization Steps Explained

### Step 1: Ensure Flutter Binding

```dart
WidgetsFlutterBinding.ensureInitialized();
```

**What it does:**
- Ensures Flutter framework is properly initialized before running async operations
- Required when using async code in `main()` before `runApp()`

### Step 2: Initialize Shared Preferences

```dart
await MySharedPrefernces.init();
```

**What it does:**
- Initializes the shared preferences system
- This is used to store user session data (like login state, user type)
- Allows the app to remember logged-in users

### Step 3: Initialize Hive Database

```dart
final appDocumentDir = await path_provider.getApplicationDocumentsDirectory();
Hive.init(appDocumentDir.path);
Hive.registerAdapter(ContactModelAdapter());
await Hive.openBox<ContactModel>('contactsData');
```

**What it does:**
- Gets the app's document directory path
- Initializes Hive (local NoSQL database)
- Registers the `ContactModel` adapter (tells Hive how to store/retrieve contacts)
- Opens the 'contactsData' box (like a table in traditional databases)

**Why Hive for contacts?**
- Emergency contacts must be available offline
- Fast local access for emergency situations
- No network dependency when sending emergency SMS

### Step 4: Handle Permissions

```dart
Future handlePermissions() async {
  await Permission.location.request();
  await Permission.sms.request();
}

await handlePermissions();
```

**What it does:**
- Requests critical permissions from the user:
  - **Location**: For GPS tracking and location sharing
  - **SMS**: For sending emergency messages

**Note**: These are requested at startup because they're essential for the app's core safety features.

### Step 5: Initialize Notifications

```dart
await initiallizedLocalNotification();
```

**What it does:**
- Sets up local notification system
- Used for background alerts and emergency notifications
- Implementation is in `lib/data/background/background_services.dart`

### Step 6: Grant SMS Permission Check

```dart
await BottomSheetControllers().grantedPermission();
```

**What it does:**
- Double-checks if SMS permission is granted
- From `BottomSheetControllers` class which manages location and SMS features

### Step 7: Initialize Firebase

```dart
await Firebase.initializeApp(
  options: DefaultFirebaseOptions.currentPlatform,
);
```

**What it does:**
- Connects the app to Firebase services
- Uses platform-specific configuration from `firebase_options.dart`
- Enables Firebase Auth and Firestore

### Step 8: Load User Data

```dart
try {
  userData = await MySharedPrefernces().getUserData();
} catch (e) {
  log("Error fetching user data from shared preferences");
}

try {
  await updateUserData(userData["id"]);
  userData = await MySharedPrefernces().getUserData();
} catch (e) {}
```

**What it does:**
- Attempts to load previously saved user data
- If user was logged in before, retrieves their information
- Updates user data from Firebase (sync latest changes)
- Stores in global `userData` variable (from `lib/data/global/appdata.dart`)

### Step 9: Run the App

```dart
runApp(const MyApp());
```

**What it does:**
- Launches the Flutter application widget tree
- Renders the initial screen

## The MyApp Widget

After initialization, the `MyApp` widget defines the app structure:

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      title: 'Woman Safety App',
      theme: ThemeData(
        textTheme: GoogleFonts.figtreeTextTheme(ThemeData.light().textTheme),
      ),
      themeMode: ThemeMode.light,
      home: FutureBuilder(
        future: MySharedPrefernces.getUserType(),
        builder: (context, AsyncSnapshot snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return loadingIndicator();
          }

          switch (snapshot.data) {
            case '':
              return const AuthSelectionScreen();
            case 'child':
              return const BottomNavPagesScreen();
            case 'parent':
              return const ParentHomeScreen();
            case 'police':
              return const PoliceDashboard();
            default:
              return const AuthSelectionScreen();
          }
        },
      ),
      debugShowCheckedModeBanner: false,
    );
  }
}
```

## App Configuration

### GetMaterialApp vs MaterialApp

```dart
return GetMaterialApp(
```

The app uses **GetMaterialApp** (from GetX package) instead of the standard MaterialApp. This provides:
- Built-in route management
- State management capabilities
- Snackbar and dialog utilities
- Easier navigation

### Theme Configuration

```dart
theme: ThemeData(
  textTheme: GoogleFonts.figtreeTextTheme(ThemeData.light().textTheme),
),
themeMode: ThemeMode.light,
```

**What it does:**
- Sets "Figtree" as the app's font family (via Google Fonts)
- Enforces light theme mode

## User Routing Logic

The most important part of the initialization is routing users to the correct screen:

### FutureBuilder for Async Routing

```dart
home: FutureBuilder(
  future: MySharedPrefernces.getUserType(),
  builder: (context, AsyncSnapshot snapshot) {
    // Routing logic
  },
),
```

**What is FutureBuilder?**
- A widget that builds itself based on a Future's state
- Waits for `MySharedPrefernces.getUserType()` to complete
- Rebuilds when data is available

### Routing Decision Tree

```dart
if (snapshot.connectionState == ConnectionState.waiting) {
  return loadingIndicator();  // Show loading while checking user type
}

switch (snapshot.data) {
  case '':
    return const AuthSelectionScreen();  // No user logged in
  case 'child':
    return const BottomNavPagesScreen(); // Child user logged in
  case 'parent':
    return const ParentHomeScreen();     // Parent user logged in
  case 'police':
    return const PoliceDashboard();      // Police user logged in
  default:
    return const AuthSelectionScreen();  // Fallback
}
```

**How it works:**

1. **Check connection state**: If still loading, show a spinner
2. **Empty string ('')**: No user type saved → User not logged in → Show login/selection screen
3. **'child'**: Previously logged in as child → Go directly to child home screen
4. **'parent'**: Previously logged in as parent → Go to parent dashboard
5. **'police'**: Previously logged in as police → Go to police dashboard
6. **Default**: Any unexpected value → Safe fallback to login screen

## Loading Indicator

The loading indicator is defined in `lib/res/utils/utils.dart`:

```dart
Widget loadingIndicator() {
  return Center(
    child: CircularProgressIndicator(
      backgroundColor: primaryColor,
      color: Colors.red,
      strokeWidth: 7,
    ),
  );
}
```

## User Type Storage

User types are stored in shared preferences:

```dart
// Saving user type (from LoginViewModel)
MySharedPrefernces.userSaveType('child');   // or 'parent', 'police'

// Reading user type
String userType = await MySharedPrefernces.getUserType();
```

## Initialization Flow Diagram

```
App Launch (main())
      ↓
[1] Initialize Flutter Binding
      ↓
[2] Initialize Shared Preferences
      ↓
[3] Initialize Hive Database
      ↓
[4] Register ContactModel Adapter
      ↓
[5] Open 'contactsData' Box
      ↓
[6] Request Location Permission
      ↓
[7] Request SMS Permission
      ↓
[8] Initialize Local Notifications
      ↓
[9] Check SMS Permission Status
      ↓
[10] Initialize Firebase
      ↓
[11] Load User Data (if exists)
      ↓
[12] Run MyApp Widget
      ↓
[13] GetMaterialApp builds
      ↓
[14] FutureBuilder checks user type
      ↓
┌─────────────┬──────────────┬──────────────┬──────────────┐
│             │              │              │              │
No Type      'child'       'parent'      'police'
    ↓            ↓              ↓              ↓
Auth        Bottom Nav    Parent Home   Police
Selection    Screen        Screen        Dashboard
Screen
```

## Global Data Variables

From `lib/data/global/appdata.dart`:

```dart
Map<String, dynamic> userData = {};
Map<String, dynamic> policeData = {};
```

These global maps store:
- **userData**: Current user's information (name, email, type, etc.)
- **policeData**: Police-specific information when a police user is logged in

**Accessed throughout the app:**
```dart
import 'package:women_safety_app/data/global/appdata.dart';

// Use anywhere
String userName = userData['name'];
String userType = userData['type'];
```

## Firebase Constants

From `lib/res/const/firebase_const.dart`:

```dart
FirebaseAuth auth = FirebaseAuth.instance;
FirebaseFirestore firestore = FirebaseFirestore.instance;

var usercollection = 'users';
var chatcollection = 'chat';
var reviewcollection = 'review';
var msgcollection = 'message';

User? currentUser = auth.currentUser;
var currentId = currentUser!.uid;
var logger = Logger();
```

These constants are imported throughout the app for consistent Firebase access.

## Error Handling During Initialization

Note the try-catch blocks:

```dart
try {
  userData = await MySharedPrefernces().getUserData();
} catch (e) {
  log("Error fetching user data from shared preferences");
}
```

**Purpose:**
- If user data doesn't exist (first launch), the app continues gracefully
- Logs the error for debugging
- Doesn't crash the app

## Summary

The initialization process:

1. **Sets up local storage** (Shared Preferences, Hive)
2. **Requests critical permissions** (Location, SMS)
3. **Initializes backend services** (Firebase)
4. **Loads user session** (if exists)
5. **Routes to appropriate screen** based on user type

This ensures:
- All required services are ready before the UI appears
- Emergency features (SMS, location) have necessary permissions
- Users see a seamless experience (auto-login if previously logged in)
- Offline features (contacts) are available immediately

In the next chapter, we'll explore the authentication system and how users log in and out.