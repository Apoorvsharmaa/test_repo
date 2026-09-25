# Chapter 4: Authentication and User Management

## Overview

The Public Safety Application supports three distinct user types: **Child** (protected person), **Parent** (guardian), and **Police** (emergency responder). Each type has different capabilities and interfaces. This chapter explains how authentication works.

## User Type System

The app distinguishes between user types using a simple string-based system:

```dart
// Stored in Firebase user document and local preferences
'child'  - Protected person who can send distress signals
'parent' - Guardian who monitors children
'police' - Emergency responder who receives distress signals
```

## Authentication Architecture

### Firebase Authentication

The app uses **Firebase Authentication** for user management:

```dart
// From lib/res/const/firebase_const.dart
FirebaseAuth auth = FirebaseAuth.instance;
FirebaseFirestore firestore = FirebaseFirestore.instance;

User? currentUser = auth.currentUser;
var currentId = currentUser!.uid;
```

### User Data Storage

User information is stored in two places:

1. **Firebase Firestore** - Cloud database (persistent, synchronized)
2. **Shared Preferences** - Local device storage (quick access, offline)

**Firestore Structure:**
```
users/
  └── {userId}/
        ├── name: "John Doe"
        ├── email: "john@example.com"
        ├── type: "child" (or "parent", "police")
        ├── phone: "+1234567890"
        └── ... (other user fields)
```

## Login System

### LoginViewModel Class

The login logic is handled by `LoginViewModel` in `lib/view_model/auth/login_view_model.dart`:

```dart
class LoginViewModel extends GetxController {
  var isPassword = false.obs;  // Toggle password visibility
  var isLoading = false.obs;   // Show loading indicator

  final formkey = GlobalKey<FormState>();
  final formdata = <String, Object>{};

  onSaveValue() async {
    // Login logic (explained below)
  }

  Future<void> signOut() async {
    // Logout logic (explained below)
  }
}
```

### Login Process

Here's the complete login flow from `onSaveValue()` method:

#### Step 1: Form Validation

```dart
if (formkey.currentState!.validate()) {
  formkey.currentState!.save();
  // Proceed with login
}
```

**What happens:**
- Checks if email and password fields are valid
- If valid, saves form data to `formdata` map

#### Step 2: Firebase Authentication

```dart
try {
  isLoading(true);
  UserCredential userCredential = await auth.signInWithEmailAndPassword(
      email: formdata['email'].toString(),
      password: formdata['password'].toString());

  if (userCredential.user != null) {
    // User authenticated successfully
  }
}
```

**What happens:**
- Sets loading state to true (shows spinner on UI)
- Calls Firebase `signInWithEmailAndPassword()`
- If successful, receives a `UserCredential` object

#### Step 3: Fetch User Type from Firestore

```dart
firestore
    .collection(usercollection)  // 'users' collection
    .doc(auth.currentUser!.uid)  // Current user's document
    .get()
    .then((value) {
      if (value.exists) {
        // Check user type and route accordingly
      }
    });
```

**What happens:**
- Queries Firestore for user's profile document
- Retrieves user data including the `type` field

#### Step 4: Route Based on User Type

**For Child Users:**

```dart
if (value['type'] == 'child') {
  isLoading.value = true;
  // Save identifier
  MySharedPrefernces.userSaveType('child');
  // Save user data locally
  MySharedPrefernces().saveUserData(value.data()!);
  userData = value.data()!;
  isLoading.value = false;
  Get.off(() => const BottomNavPagesScreen());
}
```

**For Police Users:**

```dart
else if (value['type'] == 'police') {
  isLoading.value = true;
  MySharedPrefernces.userSaveType('police');
  MySharedPrefernces().saveUserData(value.data()!);
  userData = value.data()!;
  PoliceDistressHandler().registerPoliceLocation();
  isLoading.value = false;
  Get.off(() => const PoliceDashboard());
}
```

**Special note for police**: The `registerPoliceLocation()` call registers the police officer's location for distress signal proximity matching.

**For Parent Users:**

```dart
else {
  isLoading.value = true;
  MySharedPrefernces.userSaveType('parent');
  MySharedPrefernces().saveUserData(value.data()!);
  userData = value.data()!;
  isLoading.value = false;
  Get.off(() => const ParentHomeScreen());
}
```

#### Step 5: Error Handling

```dart
on FirebaseAuthException catch (e) {
  if (e.code == 'user-not-found') {
    showError('No user found for that email.');
  } else if (e.code == 'wrong-password') {
    showError('Wrong password provided for that user.');
  } else if (e.code == 'invalid-email') {
    showError('Invalid email format.');
  } else {
    showError('An error occurred while logging in.');
  }
  isLoading(false);
}
```

**Common Firebase error codes:**
- `user-not-found` - Email not registered
- `wrong-password` - Incorrect password
- `invalid-email` - Malformed email address

## Login Screen UI

The login screen is at `lib/views/child/auth/login_screen.dart`:

### Form Structure

```dart
Form(
  key: loginViewModel.formkey,
  child: Column(
    children: [
      CustomTextField(
        hintText: 'Enter email',
        textInputAction: TextInputAction.next,
        keyboardtype: TextInputType.emailAddress,
        prefix: const Icon(Icons.email),
        validate: (email) {
          if (email!.isEmpty || email.length < 3 && email.contains('@')) {
            return 'Enter correct email';
          }
          return null;
        },
        onsave: (email) {
          loginViewModel.formdata['email'] = email ?? "";
        },
      ),
      // Password field...
    ],
  ),
)
```

### Email Field Validation

```dart
validate: (email) {
  if (email!.isEmpty || email.length < 3 && email.contains('@')) {
    return 'Enter correct email';
  }
  return null;
},
```

Returning `null` means validation passed. Any string returned is shown as an error.

### Password Field with Toggle

```dart
CustomTextField(
  isPassword: !loginViewModel.isPassword.value,
  hintText: 'Enter password',
  prefix: const Icon(Icons.lock),
  validate: (password) {
    if (password!.isEmpty || password.length < 7) {
      return 'Enter correct password';
    }
    return null;
  },
  onsave: (password) {
    loginViewModel.formdata['password'] = password ?? "";
  },
  suffix: Obx(
    () => IconButton(
      onPressed: () {
        loginViewModel.isPassword.value =
            !loginViewModel.isPassword.value;
      },
      icon: loginViewModel.isPassword.value
          ? const Icon(Icons.visibility)
          : const Icon(Icons.visibility_off),
    ),
  ),
),
```

**Features:**
- Password hidden by default
- Eye icon toggles visibility
- Minimum 7 characters required

### Login Button

```dart
PrimaryButton(
  title: 'LOGIN',
  onPressed: () {
    loginViewModel.onSaveValue();
  },
),
```

### Loading State

The entire screen is wrapped in an `Obx` widget to react to loading state:

```dart
Obx(
  () => loginViewModel.isLoading.value
      ? loadingIndicator()
      : SafeArea(
          child: SingleChildScrollView(
            // Login form
          ),
        ),
)
```

When `isLoading` is true, the form is replaced with a loading spinner.

## User Session Management

### Saving User Data

After successful login, user data is saved locally:

```dart
// Save user type
MySharedPrefernces.userSaveType('child');

// Save complete user data
MySharedPrefernces().saveUserData(value.data()!);

// Store in global variable for app-wide access
userData = value.data()!;
```

The `userData` global variable (from `lib/data/global/appdata.dart`) is used throughout the app:

```dart
import 'package:women_safety_app/data/global/appdata.dart';

// Access user info anywhere
String userName = userData['name'];
String userEmail = userData['email'];
String userType = userData['type'];
```

### Updating User Data

The `updateUserData()` function synchronizes local data with Firebase:

```dart
Future<void> updateUserData(String userID) async {
  try {
    firestore.collection(usercollection).doc(userID).get().then((value) {
      if (value.exists) {
        // Save updated data
        MySharedPrefernces().saveUserData(value.data()!);
      }
    });
    userData = await MySharedPrefernces().getUserData();
  } on FirebaseAuthException catch (e) {
    // Handle errors
  }
}
```

## Logout System

### Sign Out Logic

```dart
Future<void> signOut() async {
  await auth.signOut();  // Firebase logout
  MySharedPrefernces.userSaveType('');  // Clear user type
  await MySharedPrefernces().deleteUserData(userData);  // Clear local data
  Get.offAll(() => const AuthSelectionScreen());  // Navigate to login
}
```

**What happens:**
1. Signs out from Firebase (clears authentication token)
2. Clears user type from shared preferences
3. Deletes user data from local storage
4. Navigates to auth selection screen (clears navigation stack)

### Logout from UI

Logout is typically triggered from account screens or navigation drawers:

```dart
ElevatedButton(
  onPressed: () {
    Get.find<LoginViewModel>().signOut();
  },
  child: Text('Logout'),
)
```

## Authentication Utilities

### Error Messages

From `lib/res/utils/utils.dart`:

```dart
showError(message) {
  Get.snackbar("Error", message,
      backgroundColor: Colors.red,
      colorText: Colors.white,
      snackPosition: SnackPosition.BOTTOM);
}
```

Used throughout authentication to display user-friendly errors.

### Success Messages

```dart
showSuccess(title, message) {
  Get.snackbar(
      title,
      message,
      backgroundColor: Colors.white,
      colorText: Colors.black,
      snackPosition: SnackPosition.BOTTOM);
}
```

## User Registration

While not shown in the provided files, the app includes registration screens:

- `lib/views/child/auth/register_child_screen.dart` - Child registration
- `lib/views/parents/register_parent_screen.dart` - Parent registration
- `lib/views/child/auth/register_police_screen.dart` - Police registration

These follow a similar pattern:
1. Collect user information (name, email, password, phone, etc.)
2. Create Firebase Auth account
3. Store user profile in Firestore with appropriate `type` field
4. Navigate to role-specific home screen

## Auth Selection Screen

Before login, users choose their type at `lib/views/selection/auth_selection_screen.dart`:

```
┌─────────────────────────┐
│  Choose User Type       │
├─────────────────────────┤
│  [Register as Child]    │
│  [Register as Parent]   │
│  [Register as Police]   │
│  [Login]                │
└─────────────────────────┘
```

This ensures users create the appropriate account type.

## Authentication Flow Diagram

```
App Launch
    ↓
Check Shared Preferences
    ↓
┌───────────────┬──────────────────┐
│               │                  │
User Type       No User Type
Exists          Saved
    ↓               ↓
Auto-login      Auth Selection
to role screen  Screen
                    ↓
                Choose:
                - Register Child
                - Register Parent  
                - Register Police
                - Login
                    ↓
                Login Screen
                    ↓
                Enter Email/Password
                    ↓
                Firebase Auth
                    ↓
                ┌─────────┬─────────┬─────────┐
                │         │         │         │
            Success    Error      Error     Error
            (Valid)    (Email)   (Pass)    (Other)
                │         ↓         ↓         ↓
                │      Show Error Messages
                ↓
            Fetch User Type from Firestore
                ↓
            ┌───────────┬──────────┬──────────┐
            │           │          │          │
        Child       Parent      Police
            ↓           ↓          ↓
    Save 'child'  Save 'parent' Save 'police'
            ↓           ↓          ↓
    BottomNav     ParentHome  PoliceDash
    Screen        Screen      board
```

## Security Considerations

### Password Requirements

The app enforces:
- Minimum 7 characters (validated in UI)
- Firebase enforces additional requirements (complexity, etc.)

### Data Storage

- **Passwords**: Never stored locally, handled only by Firebase Auth
- **User data**: Stored in Firestore with Firebase security rules
- **Session tokens**: Managed automatically by Firebase SDK

### Type Verification

The app trusts the `type` field in Firestore. In a production app, Firebase Security Rules should enforce:

```javascript
// Firestore security rule example (not in repository)
allow read: if request.auth != null && request.auth.uid == userId;
allow write: if request.auth != null && request.auth.uid == userId 
             && request.resource.data.type == resource.data.type; // Prevent type change
```

## Summary

The authentication system:

1. **Uses Firebase Authentication** for secure user management
2. **Supports three user types** with different capabilities
3. **Stores user data** in both Firestore (cloud) and Shared Preferences (local)
4. **Routes users** to appropriate screens based on their type
5. **Handles errors gracefully** with user-friendly messages
6. **Manages sessions** with auto-login on app restart

The LoginViewModel class centralizes authentication logic, making it reusable and testable. The MVVM pattern separates UI concerns from business logic, resulting in clean, maintainable code.

In the next chapter, we'll explore the location services and emergency features powered by the BottomSheetControllers class.