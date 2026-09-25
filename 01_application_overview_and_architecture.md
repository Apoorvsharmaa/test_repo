# Application Overview and Architecture

## What Does This Application Do?

The Public Safety Application is a Flutter-based mobile app designed to enhance personal safety through real-time communication, location tracking, and emergency response features. It creates a safety network connecting children, parents/guardians, and police officers.

### Main Features

1. **Distress Signal System** - Users can trigger emergency alerts that notify emergency contacts and nearby police
2. **Location Sharing** - Real-time location tracking and sharing with trusted contacts
3. **Emergency SMS** - Automatic SMS messages with location details sent to saved contacts
4. **Chat Messaging** - Real-time communication between children and parents
5. **Contact Management** - Save and manage emergency contacts
6. **Multi-User Support** - Separate interfaces for children, parents, and police

## User Types

The application supports three distinct user roles:

### 1. Child/User
- Primary users who may need emergency assistance
- Can send distress signals
- Can share location with parents
- Can chat with parents/guardians
- Can manage emergency contacts
- Access to safety tips and resources

### 2. Parent/Guardian
- Monitor children's safety
- Receive emergency notifications
- Chat with children
- View children's location during emergencies

### 3. Police
- Monitor distress signals in their area
- Respond to emergency situations
- Access to distress signal details including location and user information
- Dashboard for managing active cases

## Overall Architecture

The application follows the **MVVM (Model-View-ViewModel)** architectural pattern:

```
Views (UI) → ViewModels (Business Logic) → Models/Services (Data)
```

### Architecture Layers

#### 1. **View Layer** (`lib/views/`)
Contains the UI screens organized by user type:
- `child/` - Screens for child users (home, contacts, chat, etc.)
- `parents/` - Parent-specific screens
- `police/` - Police dashboard and related screens
- `selection/` - Authentication and user type selection
- `distress/` - Distress signal screens

#### 2. **ViewModel Layer** (`lib/view_model/`)
Manages state and business logic:
- `auth/` - Authentication logic (login, registration)
- `BottomSheetControllers` - Location and SMS services
- `ChatViewModel` - Chat functionality
- `distress/` - Distress signal handling
- Various screen-specific view models

#### 3. **Model Layer** (`lib/model/`)
Data structures:
- `ContactModel` - Emergency contact information
- `UserModel` - User profile data
- `DistressSignal` - Emergency alert data

#### 4. **Data/Services Layer** (`lib/data/`)
Handles data operations:
- `services/` - Firebase operations (chat, accounts, reviews)
- `shared_preferences/` - Local data persistence
- `hive db/` - Local database for contacts
- `background/` - Background services for continuous monitoring

#### 5. **Resources Layer** (`lib/res/`)
Shared resources:
- `components/` - Reusable UI components
- `colors/` - App color scheme
- `utils/` - Utility functions
- `const/` - Constants and Firebase configuration

## Technology Stack

### Core Technologies

1. **Flutter/Dart** - Cross-platform mobile framework
   - Supports Android, iOS, Web, Windows, Linux, macOS
   - Primary language: Dart (87.9% of codebase)

2. **Firebase**
   - **Firebase Authentication** - User authentication
   - **Cloud Firestore** - Real-time database for chat, user data, distress signals
   - **Firebase Storage** - Image and media storage

3. **GetX** - State management and navigation
   - Reactive programming
   - Simple dependency injection
   - Route management

4. **Hive** - Local NoSQL database
   - Stores emergency contacts locally
   - Fast and lightweight

### Key Dependencies

Based on the imports and code, the application uses:

- **google_fonts** - Custom typography
- **geolocator** - GPS location services
- **geocoding** - Convert coordinates to addresses
- **background_sms** - Send SMS in background
- **permission_handler** - Manage app permissions
- **image_picker** - Select images for chat/profile
- **cloud_firestore** - Firebase database
- **firebase_auth** - Firebase authentication

## Important Folders and Files

### Critical Files

1. **`lib/main.dart`**
   - Application entry point
   - Initializes Firebase, Hive, permissions
   - Determines initial route based on user type

2. **`lib/res/const/firebase_const.dart`**
   - Firebase configuration constants
   - Collection names
   - Global Firebase instances

3. **`lib/res/utils/utils.dart`**
   - Utility functions (loading indicators, error/success messages)
   - Imported by 28 files (highly used)

4. **`lib/res/colors/colors.dart`**
   - Application color scheme
   - Defines primary colors for different user types

### Key Directories

```
lib/
├── main.dart                    # Entry point
├── model/                       # Data models
├── view_model/                  # Business logic
│   ├── auth/                   # Authentication
│   ├── distress/               # Emergency handling
│   └── police_distress_actions/ # Police-specific
├── views/                       # UI screens
│   ├── child/                  # Child user screens
│   ├── parents/                # Parent screens
│   ├── police/                 # Police screens
│   └── distress/               # Emergency screens
├── data/                        # Data layer
│   ├── services/               # Firebase services
│   ├── hive db/                # Local database
│   └── background/             # Background services
└── res/                         # Resources
    ├── components/             # Reusable UI
    ├── colors/                 # Theme
    └── utils/                  # Helpers
```

## Data Flow Example: Sending a Distress Signal

1. **User Action**: Child presses emergency button on home screen
2. **View**: `DistressScreen` captures the action
3. **ViewModel**: `DistressSignalHandler` processes the request:
   - Gets current location via `BottomSheetControllers`
   - Creates distress signal object
   - Uploads to Firebase via `DistressSignalService`
   - Triggers SMS to emergency contacts
4. **Background Service**: Continues monitoring and updates
5. **Police Notification**: `PoliceDistressHandler` detects new signal
6. **Police View**: Updates `PoliceDashboard` with new emergency

## State Management with GetX

The application uses GetX for reactive state management:

```dart
class BottomSheetControllers extends GetxController {
  RxBool isLoading = false.obs;  // Observable boolean
  Position? currentPosition;      // Regular variable
  
  // Methods update observables, UI reacts automatically
  getCurrentLocation() async {
    isLoading.value = true;
    // ... location logic
    isLoading.value = false;
  }
}
```

Views observe these values:

```dart
Obx(() => controller.isLoading.value 
    ? loadingIndicator() 
    : actualContent())
```

## Security Considerations

The application handles sensitive data:

1. **Location Privacy** - User location shared only during emergencies
2. **Firebase Rules** - (Not visible in code but implied) should restrict data access
3. **Local Storage** - Hive database stores contacts locally
4. **Permissions** - Explicit permission requests for location and SMS

## Platform Support

The codebase includes platform-specific code for:

- **Android** - Native Kotlin integration
- **iOS** - Swift integration  
- **Windows** - C++ runner
- **Linux** - C++ runner
- **macOS** - Swift integration
- **Web** - HTML entry point

Most business logic is in Dart and shared across platforms.

## Next Steps

To understand how the application works in detail:

1. Read [Application Setup and Initialization](02_application_setup_and_initialization.md) to learn how the app starts
2. Explore [Core Features and View Models](03_core_features_and_view_models.md) to understand key functionality