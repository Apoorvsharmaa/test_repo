# Chapter 2: Architecture and Project Structure

## Overall Architecture

The Public Safety Application follows the **Model-View-ViewModel (MVVM)** architectural pattern, which is a common approach in Flutter development for separating concerns and maintaining clean code.

### MVVM Pattern Breakdown

```
┌─────────────────────────────────────────────────┐
│                    VIEW                         │
│  (UI Screens - LoginScreen, HomeScreen, etc.)   │
│           Flutter Widgets & Layouts              │
└─────────────────┬───────────────────────────────┘
                  │
                  │ User Interactions
                  │ (Button Clicks, Form Submits)
                  ↓
┌─────────────────────────────────────────────────┐
│                 VIEW MODEL                       │
│  (Business Logic Controllers)                    │
│  - LoginViewModel                                │
│  - ChatViewModel                                 │
│  - BottomSheetControllers                        │
│  Uses GetX for State Management                  │
└─────────────────┬───────────────────────────────┘
                  │
                  │ Data Operations
                  ↓
┌─────────────────────────────────────────────────┐
│                  MODEL                           │
│  (Data & Services)                               │
│  - ContactModel, UserModel                       │
│  - Firebase Services                             │
│  - Hive Database                                 │
└─────────────────────────────────────────────────┘
```

### Why MVVM?

1. **Separation of Concerns**: UI code is separate from business logic
2. **Testability**: ViewModels can be tested independently
3. **Maintainability**: Changes to UI don't affect business logic and vice versa
4. **Reusability**: ViewModels can be reused across different views

## Project Folder Structure

The repository is organized as follows:

```
Public_Safety_Application/
│
├── android/              # Android platform-specific code
├── ios/                  # iOS platform-specific code
├── linux/                # Linux desktop configuration
├── macos/                # macOS configuration
├── windows/              # Windows desktop configuration
├── web/                  # Web platform files
├── test/                 # Test files
│
├── lib/                  # Main application code (Dart)
│   ├── main.dart         # Application entry point
│   ├── firebase_options.dart  # Firebase configuration
│   │
│   ├── data/             # Data layer
│   │   ├── background/   # Background services
│   │   ├── global/       # Global app data
│   │   ├── hive db/      # Local database (Hive)
│   │   ├── services/     # API and service classes
│   │   └── shared_preferences/  # Local storage helpers
│   │
│   ├── model/            # Data models
│   │   ├── contact_model.dart
│   │   ├── child_user_model.dart
│   │   └── distress_signal_model.dart
│   │
│   ├── view_model/       # Business logic controllers
│   │   ├── auth/         # Authentication ViewModels
│   │   ├── distress/     # Distress signal handling
│   │   ├── police_distress_actions/  # Police-specific logic
│   │   ├── bottom_sheat_view_model.dart
│   │   ├── chat_view_model.dart
│   │   └── home_view_model.dart
│   │
│   ├── views/            # UI screens and widgets
│   │   ├── child/        # Child user screens
│   │   ├── parents/      # Parent user screens
│   │   ├── police/       # Police user screens
│   │   ├── distress/     # Distress-related screens
│   │   └── selection/    # User type selection
│   │
│   └── res/              # Resources
│       ├── colors/       # Color definitions
│       ├── components/   # Reusable UI components
│       ├── const/        # Constants
│       ├── theme/        # Theme configuration
│       └── utils/        # Utility functions
│
├── pubspec.yaml          # Project dependencies
├── firebase.json         # Firebase configuration
└── README.md             # Project readme
```

## Layer-by-Layer Breakdown

### 1. Data Layer (`lib/data/`)

Handles data operations and external services.

**Key Files:**

- **`data/global/appdata.dart`**: Global application state
  ```dart
  Map<String, dynamic> userData = {};
  Map<String, dynamic> policeData = {};
  ```

- **`data/hive db/`**: Local database operations using Hive
  - `boxes.dart` - Database box accessors
  - `hive_db.dart` - Database operations

- **`data/services/`**: Service classes for Firebase operations
  - `accounts_firestore_services.dart` - User account operations
  - `chat_firestore_services.dart` - Chat database queries
  - `chat_message_services.dart` - Message sending/receiving
  - `chat_image_services.dart` - Image upload/download
  - `chat_audio_services.dart` - Audio message handling
  - `review_services.dart` - Review/feedback operations

- **`data/shared_preferences/`**: Persistent local storage for user sessions

- **`data/background/`**: Background service implementations

### 2. Model Layer (`lib/model/`)

Defines data structures.

**Key Models:**

- **`contact_model.dart`**: Emergency contact structure
  ```dart
  @HiveType(typeId: 0)
  class ContactModel extends HiveObject {
    @HiveField(0)
    final String name;
    
    @HiveField(1)
    final String phoneNumber;
  }
  ```

- **`child_user_model.dart`**: User information structure
- **`distress_signal_model.dart`**: Emergency signal data structure

### 3. ViewModel Layer (`lib/view_model/`)

Contains business logic controllers using GetX.

**Key ViewModels:**

- **`auth/login_view_model.dart`**: Authentication logic (covered in Chapter 4)
- **`bottom_sheat_view_model.dart`**: Location services (covered in Chapter 5)
- **`chat_view_model.dart`**: Chat functionality (covered in Chapter 6)
- **`home_view_model.dart`**: Home screen logic
- **`contacts_view_model.dart`**: Contact management
- **`accounts_view_model.dart`**: User account management

**ViewModel Pattern Example:**

From `lib/view_model/chat_view_model.dart`:

```dart
class ChatViewModel extends GetxController {
  var status = ''.obs;  // Observable state
  var name = ''.obs;
  var controller = TextEditingController();
  
  @override
  onInit() {
    getstatus();
    super.onInit();
  }
  
  getstatus() async {
    // Business logic here
  }
}
```

The `GetxController` provides:
- Reactive state management (`.obs` makes variables observable)
- Lifecycle methods (`onInit`, `onClose`)
- Easy dependency injection

### 4. View Layer (`lib/views/`)

UI screens organized by user type.

**Structure:**

- **`child/`**: Screens for child/protected person users
  - `auth/` - Login, registration
  - `home/` - Home screen with emergency button
  - `chat/` - Chat interface
  - `contact/` - Emergency contact management
  - `accounts/` - Profile settings
  - `bottom_nav_bar.dart` - Bottom navigation

- **`parents/`**: Screens for parent/guardian users
  - `parent_home_screen.dart` - Parent dashboard
  - `register_parent_screen.dart` - Parent registration

- **`police/`**: Screens for police users
  - `police_dashboard.dart` - Police control panel

- **`distress/`**: Emergency-related screens
  - `distress_screen.dart` - Distress signal interface

- **`selection/`**: User type selection
  - `auth_selection_screen.dart` - Choose user type

### 5. Resources Layer (`lib/res/`)

Reusable resources and utilities.

**Key Components:**

- **`colors/colors.dart`**: Color definitions
  ```dart
  Color primaryColor = const Color(0xfffc3b77);
  Color policePrimary = const Color.fromARGB(255, 15, 0, 125);
  ```

- **`components/`**: Reusable UI widgets
  - `common/` - Generic components (buttons, text fields)
  - `chat/` - Chat-specific components
  - `emergency/` - Emergency UI components
  - `home/` - Home screen components

- **`const/firebase_const.dart`**: Firebase constants
  ```dart
  FirebaseAuth auth = FirebaseAuth.instance;
  FirebaseFirestore firestore = FirebaseFirestore.instance;
  var usercollection = 'users';
  var chatcollection = 'chat';
  ```

- **`utils/utils.dart`**: Utility functions (loading indicators, error messages, dialogs)

## Key Dependencies

From `pubspec.yaml` (observed from imports):

### State Management
- **get** - GetX for state management, navigation, and dependency injection

### Firebase Integration
- **firebase_core** - Firebase initialization
- **firebase_auth** - Authentication
- **cloud_firestore** - Cloud database

### Location Services
- **geolocator** - GPS location tracking
- **geocoding** - Address from coordinates
- **google_maps_flutter** - Map display

### Messaging
- **background_sms** - Send SMS messages

### Local Storage
- **hive** - NoSQL local database
- **hive_flutter** - Flutter integration for Hive
- **shared_preferences** - Key-value storage

### Permissions
- **permission_handler** - Request app permissions

### UI
- **google_fonts** - Custom fonts

### Image Handling
- **image_picker** - Select images from gallery/camera

### Other
- **path_provider** - File system paths
- **logger** - Logging utility

## Firebase Configuration

The app uses Firebase for backend services:

```dart
// From lib/firebase_options.dart
class DefaultFirebaseOptions {
  static FirebaseOptions get currentPlatform {
    // Platform-specific Firebase configuration
  }
}
```

**Firebase Collections Structure:**

```
Firestore Database
├── users/           # User profiles
│   └── {userId}
│       ├── name
│       ├── email
│       ├── type (child/parent/police)
│       └── ...
│
├── chat/            # Chat conversations
│   └── {chatId}
│       └── message/  # Messages subcollection
│
└── review/          # User reviews/feedback
```

## State Management with GetX

The app uses **GetX** for reactive state management:

### Observable State
```dart
var isLoading = false.obs;  // Observable boolean
```

### Updating State
```dart
isLoading.value = true;  // Update triggers UI rebuild
```

### Observing in UI
```dart
Obx(() => isLoading.value ? loadingIndicator() : actualContent())
```

### Navigation
```dart
Get.to(() => NextScreen());        // Navigate
Get.off(() => NewScreen());        // Replace
Get.offAll(() => HomeScreen());    // Clear stack
```

### Dependency Injection
```dart
var loginViewModel = Get.put(LoginViewModel());  // Register
var viewModel = Get.find<LoginViewModel>();      // Retrieve
```

## Design Patterns Used

1. **MVVM (Model-View-ViewModel)**: Overall architecture
2. **Repository Pattern**: Service classes abstract data access
3. **Singleton Pattern**: Firebase instances, shared preferences
4. **Observer Pattern**: GetX reactive state
5. **Factory Pattern**: Model constructors (e.g., `UserModel.fromJson()`)

## Platform-Specific Code

The app includes platform-specific entry points:

- **Linux**: `linux/main.cc` (C++)
- **Windows**: `windows/runner/main.cpp` (C++)
- **Android**: `android/app/src/main/kotlin/` (Kotlin)
- **iOS**: `ios/Runner/AppDelegate.swift` (Swift)

These handle platform initialization and bridge to Flutter code.

## Summary

The Public Safety Application uses a clean MVVM architecture with:
- **Data layer** for services and storage
- **Model layer** for data structures
- **ViewModel layer** for business logic (using GetX)
- **View layer** for UI components
- **Resources layer** for shared utilities

This structure promotes:
- Code reusability
- Easy testing
- Clear separation of concerns
- Scalability for future features

In the next chapter, we'll examine how the application initializes and routes users to appropriate screens.