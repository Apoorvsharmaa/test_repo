# Application Setup and Initialization

## Entry Point: main.dart

The application starts in `lib/main.dart`, which handles all initialization before showing any UI to the user.

### Main Function Flow

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // ... initialization steps
  runApp(const MyApp());
}
```

### Initialization Steps

The `main()` function executes several critical setup tasks in sequence:

#### 1. Widget Binding Initialization

```dart
WidgetsFlutterBinding.ensureInitialized();
```

This ensures Flutter is ready before running asynchronous initialization code. Required when using `async` in `main()`.

#### 2. Shared Preferences Initialization

```dart
await MySharedPrefernces.init();
```

Initializes the local storage system for persisting:
- User authentication state
- User type (child, parent, or police)
- User profile data

#### 3. Hive Database Setup

```dart
final appDocumentDir = await path_provider.getApplicationDocumentsDirectory();
Hive.init(appDocumentDir.path);
Hive.registerAdapter(ContactModelAdapter());
await Hive.openBox<ContactModel>('contactsData');
```

**Purpose**: Sets up local NoSQL database for storing emergency contacts

- **Hive.init()** - Initializes Hive with the app's document directory
- **registerAdapter()** - Registers the `ContactModel` type adapter for serialization
- **openBox()** - Opens/creates the 'contactsData' box (similar to a table)

**Why Hive?** Fast, lightweight local database perfect for storing contacts that need to be accessible even offline.

#### 4. Permission Handling

```dart
Future handlePermissions() async {
  await Permission.location.request();
  await Permission.sms.request();
}

await handlePermissions();
```

Requests critical permissions:
- **Location** - Required for tracking user location during emergencies
- **SMS** - Required for sending emergency text messages to contacts

#### 5. Background Services

```dart
await initiallizedLocalNotification();
```

Sets up local notification system for:
- Alerting police of new distress signals
- Notifying users of important events
- Background monitoring

#### 6. SMS Permission Verification

```dart
await BottomSheetControllers().grantedPermission();
```

Double-checks SMS permissions are granted, as they're critical for emergency messaging.

#### 7. Firebase Initialization

```dart
await Firebase.initializeApp(
  options: DefaultFirebaseOptions.currentPlatform,
);
```

Connects to Firebase backend services:
- **Firebase Authentication** - User login/registration
- **Cloud Firestore** - Real-time database
- **Firebase Storage** - Image/media storage

The `DefaultFirebaseOptions.currentPlatform` automatically selects the correct configuration for the current platform (Android, iOS, Web, etc.).

#### 8. User Data Loading

```dart
try {
  userData = await MySharedPrefernces().getUserData();
} catch (e) {
  log("Error fetching user data from shared preferences");
}
```

Loads previously saved user data from local storage. If this is the first run or user is logged out, this will fail gracefully.

#### 9. User Data Update

```dart
try {
  await updateUserData(userData["id"]);
  userData = await MySharedPrefernces().getUserData();
} catch (e) {}
```

Syncs local user data with Firebase to ensure it's up-to-date. This refreshes profile information from the cloud.

## The MyApp Widget

After initialization, the app creates the `MyApp` widget:

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      title: 'Woman Safety App',
      theme: ThemeData(
        textTheme: GoogleFonts.figtreeTextTheme(ThemeData.light().textTheme),
      ),
      themeMode: ThemeMode.light,
      home: FutureBuilder(...),
      debugShowCheckedModeBanner: false,
    );
  }
}
```

### Key Components:

1. **GetMaterialApp** - GetX version of MaterialApp, provides:
   - Navigation management
   - State management
   - Dependency injection

2. **Google Fonts** - Custom typography using the Figtree font family

3. **FutureBuilder** - Determines initial screen based on stored user type

## Initial Route Determination

The app uses a `FutureBuilder` to decide which screen to show first:

```dart
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
```

### Routing Logic:

1. **Empty String (`''`)** - No user type saved → Show `AuthSelectionScreen`
   - New user or logged-out user
   - User selects whether to register as child, parent, or police

2. **'child'** - Child user → Show `BottomNavPagesScreen`
   - Main app interface with bottom navigation
   - Access to home, contacts, chat, accounts

3. **'parent'** - Parent user → Show `ParentHomeScreen`
   - Parent dashboard
   - List of connected children
   - Chat access

4. **'police'** - Police user → Show `PoliceDashboard`
   - Active distress signals
   - Emergency response interface

5. **Loading State** - While checking user type → Show `loadingIndicator()`

## Global Application Data

Defined in `lib/data/global/appdata.dart`:

```dart
Map<String, dynamic> userData = {};
Map<String, dynamic> policeData = {};
```

These global maps store:
- **userData** - Current logged-in user's profile information
- **policeData** - Police-specific data when applicable

Accessed throughout the app to get user information without repeated database queries.

## Firebase Constants

Defined in `lib/res/const/firebase_const.dart`:

```dart
FirebaseAuth auth = FirebaseAuth.instance;
FirebaseFirestore firestore = FirebaseFirestore.instance;

var usercollection = 'users';
var chatcollection = 'chat';
var reviewcollection = 'review';
var msgcollection = 'message';
var allmsgcollection = 'allmessage';

User? currentUser = auth.currentUser;
var currentId = currentUser!.uid;
var logger = Logger();
```

### Global Instances:

- **auth** - Firebase Authentication instance
- **firestore** - Cloud Firestore database instance
- **currentUser** - Currently authenticated user
- **currentId** - Current user's unique ID
- **logger** - Logging utility for debugging

### Collection Names:

Firestore database collections:
- **users** - User profiles (child, parent, police)
- **chat** - Chat room metadata
- **message** - Individual chat messages
- **review** - User reviews/feedback
- **allmessage** - Global message archive

These constants ensure consistent database access throughout the app.

## Shared Preferences System

The `MySharedPrefernces` class (in `lib/data/shared_preferences/shared_preferences.dart`) handles local data persistence:

### Key Methods:

1. **Save User Type**
```dart
MySharedPrefernces.userSaveType('child');
```
Stores whether user is child, parent, or police.

2. **Get User Type**
```dart
String userType = await MySharedPrefernces.getUserType();
```
Retrieves saved user type.

3. **Save User Data**
```dart
MySharedPrefernces().saveUserData(userDataMap);
```
Stores complete user profile locally.

4. **Get User Data**
```dart
Map<String, dynamic> data = await MySharedPrefernces().getUserData();
```
Retrieves saved user profile.

5. **Delete User Data**
```dart
await MySharedPrefernces().deleteUserData(userData);
```
Clears saved user data (used during logout).

## Permission Flow

The application requires two critical permissions:

### Location Permission

**Purpose**: Track user location during emergencies

**When Requested**: 
- During app initialization
- When user first accesses location features
- In `BottomSheetControllers.getCurrentLocation()`

**Permission Levels**:
- `denied` - User denied permission
- `deniedForever` - User permanently denied (must enable in settings)
- `granted` - Permission granted

### SMS Permission

**Purpose**: Send emergency SMS to saved contacts

**When Requested**:
- During app initialization  
- Before sending emergency messages

**Critical for**:
- Distress signal system
- Location sharing via SMS

## Color System

Defined in `lib/res/colors/colors.dart`:

```dart
Color white = Colors.white;
Color black = Colors.black;
List<Color> gradien = const [
  Color(0xFFFD8080),
  Color(0xFFFB8580),
  Color(0xFFFBD079),
];

Color primaryColor = const Color(0xfffc3b77);  // Pink for child users
Color policePrimary = const Color.fromARGB(255, 15, 0, 125);  // Dark blue for police
```

**Design Pattern**: Different primary colors for different user types helps users quickly identify their interface.

## Utility Functions

Defined in `lib/res/utils/utils.dart`, used throughout the app:

### Loading Indicator
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
Shows a loading spinner during async operations.

### Error Display
```dart
showError(message) {
  Get.snackbar("Error", message,
      backgroundColor: Colors.red,
      colorText: Colors.white,
      snackPosition: SnackPosition.BOTTOM);
}
```
Displays error messages to the user.

### Success Display
```dart
showSuccess(title, message) {
  Get.snackbar(title, message,
      backgroundColor: Colors.white,
      colorText: Colors.black,
      snackPosition: SnackPosition.BOTTOM);
}
```
Displays success confirmations.

### Location Dialog
```dart
void showLocationDialog(currentAddress, currentPosition)
```
Displays formatted location information including address, latitude, and longitude.

## Platform-Specific Entry Points

While `lib/main.dart` is the Dart entry point, each platform has its own native entry point:

### Linux: `linux/main.cc`
```cpp
int main(int argc, char** argv) {
  g_autoptr(MyApplication) app = my_application_new();
  return g_application_run(G_APPLICATION(app), argc, argv);
}
```

### Windows: `windows/runner/main.cpp`
```cpp
int APIENTRY wWinMain(_In_ HINSTANCE instance, ...) {
  // Windows initialization
  FlutterWindow window(project);
  // ...
}
```

These native entry points bootstrap the Flutter engine and then hand control to `lib/main.dart`.

## Initialization Summary

The complete initialization sequence:

1. ✅ Flutter engine starts
2. ✅ Native platform code runs
3. ✅ Dart `main()` function begins
4. ✅ Shared Preferences initialized
5. ✅ Hive database set up
6. ✅ Permissions requested
7. ✅ Background services started
8. ✅ Firebase connected
9. ✅ User data loaded
10. ✅ Initial route determined
11. ✅ First screen displayed

Total initialization time: Typically 2-5 seconds depending on network speed and device performance.

## Next Chapter

Now that you understand how the application initializes, proceed to [Core Features and View Models](03_core_features_and_view_models.md) to learn about the main functionality.