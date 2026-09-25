# Core Features and View Models

## Introduction to View Models

This application uses the **MVVM (Model-View-ViewModel)** pattern. View Models sit between the UI (Views) and the data layer, handling business logic and state management.

### Why View Models?

- **Separation of Concerns**: UI code separate from business logic
- **Testability**: Logic can be tested without UI
- **Reusability**: Same logic can be used across multiple views
- **State Management**: Centralized state using GetX

All view models in this app extend `GetxController` from the GetX package, enabling reactive state management.

## Core Feature 1: Location Tracking and Sharing

### BottomSheetControllers

**File**: `lib/view_model/bottom_sheat_view_model.dart`

**Purpose**: Manages location services and SMS emergency messaging

**Key Responsibilities**:
1. Get current GPS location
2. Convert coordinates to human-readable address
3. Send emergency SMS messages
4. Manage location permissions

### Location Tracking Implementation

#### Getting Current Location

```dart
getCurrentLocation() async {
  isLoading.value = true;
  requestPermissions();
  permission = await Geolocator.checkPermission();
  
  if (permission == LocationPermission.denied) {
    permission = await Geolocator.requestPermission();
    if (permission == LocationPermission.denied) {
      return Future.error('Location permissions are denied');
    }
  }

  try {
    currentPosition = await Geolocator.getCurrentPosition(
        forceAndroidLocationManager: true,
        desiredAccuracy: LocationAccuracy.high);
    
    if (currentPosition != null) {
      getCurrentAddress();
    }
  } catch (e) {
    openLocationSettings();
    showError('Failed to get current position: $e');
  }
  isLoading.value = false;
}
```

**Flow**:
1. Set loading state to true
2. Check location permission status
3. Request permission if denied
4. Get high-accuracy GPS coordinates
5. Convert coordinates to address
6. Handle errors gracefully

#### Reverse Geocoding (Coordinates to Address)

```dart
getCurrentAddress() async {
  try {
    List<Placemark> placemarks = await placemarkFromCoordinates(
        currentPosition!.latitude, currentPosition!.longitude);

    if (placemarks.isNotEmpty) {
      Placemark place = placemarks[0];
      currentAddress = "${place.locality}, ${place.street}, ${place.postalCode}, ${place.name}, ${place.subAdministrativeArea}";
      realtimeAddress.value = currentAddress!;
    }
  } catch (e) {
    showError('Failed to get current address: $e');
  }
}
```

**What it does**:
- Takes latitude/longitude coordinates
- Calls geocoding API
- Constructs readable address from components
- Updates observable `realtimeAddress` (UI updates automatically)

### Emergency Location Sharing

```dart
sendLocation() async {
  final contactBox = Boxes.getContacts();
  if (contactBox == null || contactBox.isEmpty) {
    showError('No contacts available to send location');
    return;
  }

  String msgBody = "https://maps.google.com/?daddr=${currentPosition!.latitude},${currentPosition!.longitude}, $currentAddress";

  final contactList = contactBox.values.toList();
  List<Future> smsSendingTasks = [];

  if (await grantedPermission()) {
    for (var element in contactList) {
      smsSendingTasks.add(sendMsg(element.phoneNumber, "I am in trouble $msgBody"));
    }

    try {
      await Future.wait(smsSendingTasks);
      showSuccess('All messages sent successfully', '');
    } catch (e) {
      showError('Failed to send some messages: $e');
    }
  }
}
```

**Process**:
1. Retrieve emergency contacts from Hive database
2. Create Google Maps link with coordinates and address
3. Build message: "I am in trouble [location link]"
4. Send SMS to all contacts simultaneously using `Future.wait()`
5. Show success/error feedback

### SMS Sending

```dart
sendMsg(String phoneNumber, String msg, {int? sim}) async {
  var result = await BackgroundSms.sendMessage(
      phoneNumber: phoneNumber, message: msg, simSlot: sim);
  if (result == SmsStatus.sent) {
    showSuccess('Message sent', "");
  } else {
    showError('Failed to send');
  }
}
```

Uses the `background_sms` package to send SMS even when app is in background.

### State Variables

```dart
RxBool isLoading = false.obs;               // Loading indicator
Position? currentPosition;                   // GPS coordinates
String? currentAddress = 'Your Address/Location';  // Street address
RxString realtimeAddress = "Your Address/Location".obs;  // Observable address
LocationPermission? permission;              // Permission status
```

- `Rx` prefix = Observable (UI automatically updates when value changes)
- `.obs` = Makes the variable observable
- `.value` = Access/modify observable value

## Core Feature 2: Chat and Messaging

### ChatViewModel

**File**: `lib/view_model/chat_view_model.dart`

**Purpose**: Manages real-time chat between users (children and parents)

**Key Services Used**:
1. `ChatFirestoreService` - Database operations
2. `ChatMessageService` - Send/delete messages
3. `ChatImageService` - Handle image uploads
4. `ChatAudioService` - Handle audio messages

### Chat Functionality

#### Getting User Status

```dart
getstatus() async {
  return await firestore
      .collection(usercollection)
      .doc(auth.currentUser!.uid)
      .get()
      .then((value) {
    status.value = value.data()!['type'];  // 'child', 'parent', or 'police'
    name.value = value.data()!['name'];
  }).catchError((e) {
    return showError(e.toString());
  });
}
```

Retrieves current user's type and name from Firestore.

#### Real-Time Chat Stream

```dart
Stream<QuerySnapshot> getChat(friendid) {
  return firestoreService.getChat(friendid);
}
```

Returns a stream of messages between current user and specified friend. The UI listens to this stream and updates automatically when new messages arrive.

**Stream**: A sequence of asynchronous events. Perfect for real-time chat where messages arrive at unpredictable times.

#### Sending Messages

```dart
sendMessage({friendId, message, type}) {
  return messageService.sendMessage(
      friendId: friendId, message: message, type: type);
}
```

**Parameters**:
- `friendId` - Recipient's user ID
- `message` - Message content
- `type` - Message type (text, image, audio)

#### Deleting Messages

```dart
deleteMsg({friendid, docId}) {
  return messageService.deleteMessage(friendId: friendid, docId: docId);
}
```

Allows users to delete sent messages.

### Image Handling in Chat

#### Selecting an Image

```dart
selectImage(ImageSource source) async {
  try {
    final path = await imageService.selectImage(source: source);
    if (path != null) {
      imagePath.value = path;
    } else {
      showError("No image selected.");
    }
  } catch (e) {
    showError("Error selecting image: $e");
  }
}
```

**ImageSource** can be:
- `ImageSource.camera` - Take new photo
- `ImageSource.gallery` - Select from gallery

#### Uploading Image

```dart
Future<void> uploadImage() async {
  try {
    var value = await imageService.uploadImage(imagePath: imagePath.value);
    if (value != null) {
      imageUrls.value = value;  // URL of uploaded image
    }
  } catch (e) {
    showError("Error uploading image: $e");
  }
}
```

**Process**:
1. User selects image from camera/gallery
2. Local path stored in `imagePath`
3. Image uploaded to Firebase Storage
4. Download URL returned and stored in `imageUrls`
5. URL sent in chat message

### Getting Connected Users

```dart
Stream<QuerySnapshot> getChildData() {
  return firestoreService.getChildData();
}

Stream<QuerySnapshot> getParentData() {
  return firestoreService.getParentData();
}
```

Retrieve lists of children or parents to chat with, depending on current user type.

## Core Feature 3: Authentication

### LoginViewModel

**File**: `lib/view_model/auth/login_view_model.dart`

**Purpose**: Handle user login and logout

### Login Process

```dart
onSaveValue() async {
  if (formkey.currentState!.validate()) {
    formkey.currentState!.save();

    try {
      isLoading(true);
      UserCredential userCredential = await auth.signInWithEmailAndPassword(
          email: formdata['email'].toString(),
          password: formdata['password'].toString());

      if (userCredential.user != null) {
        firestore.collection(usercollection)
            .doc(auth.currentUser!.uid)
            .get()
            .then((value) {
          if (value.exists) {
            if (value['type'] == 'child') {
              MySharedPrefernces.userSaveType('child');
              MySharedPrefernces().saveUserData(value.data()!);
              userData = value.data()!;
              Get.off(() => const BottomNavPagesScreen());
            } else if (value['type'] == 'police') {
              MySharedPrefernces.userSaveType('police');
              MySharedPrefernces().saveUserData(value.data()!);
              userData = value.data()!;
              PoliceDistressHandler().registerPoliceLocation();
              Get.off(() => const PoliceDashboard());
            } else {
              MySharedPrefernces.userSaveType('parent');
              MySharedPrefernces().saveUserData(value.data()!);
              userData = value.data()!;
              Get.off(() => const ParentHomeScreen());
            }
          }
        });
      }
    } on FirebaseAuthException catch (e) {
      // Error handling
      isLoading(false);
    }
  }
}
```

**Login Flow**:
1. Validate form (email and password format)
2. Authenticate with Firebase
3. Retrieve user document from Firestore
4. Check user type (child/parent/police)
5. Save user type and data locally
6. Navigate to appropriate home screen
7. For police: Also register their location for distress monitoring

### Error Handling

The login handles specific Firebase errors:

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
}
```

### Logout Process

```dart
Future<void> signOut() async {
  await auth.signOut();
  MySharedPrefernces.userSaveType('');
  await MySharedPrefernces().deleteUserData(userData);
  Get.offAll(() => const AuthSelectionScreen());
}
```

**Steps**:
1. Sign out from Firebase
2. Clear saved user type
3. Delete local user data
4. Navigate to authentication selection screen
5. `Get.offAll()` clears navigation stack (can't press back)

### Update User Data Function

```dart
Future<void> updateUserData(String userID) async {
  try {
    firestore.collection(usercollection).doc(userID).get().then((value) {
      if (value.exists) {
        if (value['type'] == 'child') {
          MySharedPrefernces().saveUserData(value.data()!);
        } else if (value['type'] == 'police') {
          MySharedPrefernces().saveUserData(value.data()!);
        } else {
          MySharedPrefernces().saveUserData(value.data()!);
        }
      }
    });
    userData = await MySharedPrefernces().getUserData();
  } catch (error) {
    showError(error.toString());
  }
}
```

This function syncs local user data with Firebase, ensuring profile information stays current.

## Core Feature 4: Contact Management

### ContactModel

**File**: `lib/model/contact_model.dart`

```dart
@HiveType(typeId: 0)
class ContactModel extends HiveObject {
  @HiveField(0)
  final String name;

  @HiveField(1)
  final String phoneNumber;

  ContactModel({required this.name, required this.phoneNumber});
}
```

**Hive Annotations**:
- `@HiveType(typeId: 0)` - Registers this class with Hive
- `@HiveField(0)` - Marks field for storage, with position index
- `extends HiveObject` - Enables features like `.delete()` and `.save()`

**Why Local Storage?**
Emergency contacts stored locally ensure they're accessible even without internet connection - critical during emergencies.

### Accessing Contacts

The `Boxes` class (in `lib/data/hive db/boxes.dart`) provides easy access:

```dart
class Boxes {
  static Box<ContactModel> getContacts() => Hive.box<ContactModel>('contactsData');
}
```

Usage example from `BottomSheetControllers`:

```dart
final contactBox = Boxes.getContacts();
if (contactBox == null || contactBox.isEmpty) {
  showError('No contacts available');
  return;
}
final contactList = contactBox.values.toList();
```

## UI Components

### PrimaryButton

**File**: `lib/res/components/common/primary_button.dart`

```dart
class PrimaryButton extends StatelessWidget {
  final String title;
  final Function onPressed;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: 60,
      width: double.infinity,
      child: ElevatedButton(
        onPressed: () { onPressed(); },
        style: ElevatedButton.styleFrom(
            backgroundColor: primaryColor,
            shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(30))),
        child: Text(
          title,
          style: const TextStyle(fontSize: 18, color: Colors.white),
        ),
      ),
    );
  }
}
```

**Features**:
- Full width button (responsive)
- Fixed height (60px)
- Rounded corners (30px radius)
- Consistent styling across app
- Uses primary color from theme

**Usage**:
```dart
PrimaryButton(
  title: 'LOGIN',
  onPressed: () {
    loginViewModel.onSaveValue();
  },
)
```

### CustomTextField

**File**: `lib/res/components/common/custom_textfield.dart`

A reusable text input field with:
- Validation support
- Password visibility toggle
- Prefix/suffix icons
- Custom keyboard types
- Rounded borders
- Consistent styling

**Example Usage** (from LoginScreen):

```dart
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
)
```

## Login Screen Example

**File**: `lib/views/child/auth/login_screen.dart`

Demonstrates how View Models are used in practice:

```dart
class LoginScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    var loginViewModel = Get.put(LoginViewModel());

    return Scaffold(
      body: Obx(() => loginViewModel.isLoading.value
          ? loadingIndicator()
          : SafeArea(
              child: SingleChildScrollView(
                child: Form(
                  key: loginViewModel.formkey,
                  child: Column(
                    children: [
                      // Logo and title
                      CustomTextField(...),
                      CustomTextField(...),
                      PrimaryButton(
                        title: 'LOGIN',
                        onPressed: () {
                          loginViewModel.onSaveValue();
                        },
                      ),
                      // Forgot password link
                    ],
                  ),
                ),
              ),
            ),
      ),
    );
  }
}
```

**Pattern**:
1. `Get.put(LoginViewModel())` - Creates and registers view model
2. `Obx(() => ...)` - Observes `isLoading` state
3. Shows loading indicator when `isLoading.value == true`
4. Shows login form when `isLoading.value == false`
5. Button calls `loginViewModel.onSaveValue()` which handles login logic

## State Management Pattern

GetX provides reactive programming:

### In ViewModel:
```dart
RxBool isLoading = false.obs;  // Observable

void someAsyncTask() async {
  isLoading.value = true;   // Set to true
  await doWork();
  isLoading.value = false;  // Set to false
}
```

### In View:
```dart
Obx(() => controller.isLoading.value
    ? loadingWidget()
    : contentWidget())
```

When `isLoading.value` changes, the UI automatically rebuilds.

## Execution Flow Example: Emergency Location Share

1. **User Action**: Presses "Share Location" button
2. **View**: Calls `bottomSheetController.sendLocation()`
3. **ViewModel** (`BottomSheetControllers`):
   - Gets current location
   - Converts to address
   - Retrieves contacts from Hive
   - Creates Google Maps link
   - Sends SMS to all contacts
4. **Services**: `BackgroundSms` sends messages
5. **Feedback**: Shows success/error message to user
6. **Result**: All emergency contacts receive SMS with location

## Important Dependencies Summary

Based on the code analyzed:

1. **firebase_auth** - User authentication
2. **cloud_firestore** - Real-time database
3. **get** - State management and navigation  
4. **hive** - Local NoSQL database
5. **geolocator** - GPS location services
6. **geocoding** - Address lookup
7. **background_sms** - SMS messaging
8. **permission_handler** - Permission management
9. **image_picker** - Camera/gallery access
10. **google_fonts** - Custom typography

## Best Practices Observed

1. **Separation of Concerns**: Views don't contain business logic
2. **Error Handling**: Try-catch blocks with user-friendly messages
3. **Loading States**: Visual feedback during async operations
4. **Permission Checks**: Proper permission handling before operations
5. **Null Safety**: Null checks and safe navigation operators
6. **Reactive UI**: Automatic updates using observable variables
7. **Reusable Components**: Shared widgets like `PrimaryButton` and `CustomTextField`

## Conclusion

The application's core features revolve around three main view models:

1. **BottomSheetControllers** - Location and emergency SMS
2. **ChatViewModel** - Real-time messaging
3. **LoginViewModel** - Authentication and user management

These work together with Firebase services, local storage, and UI components to create a comprehensive safety application. The MVVM pattern keeps the code organized and maintainable, while GetX provides reactive state management that automatically updates the UI when data changes.