# Chapter 1: Project Overview and Features

## Introduction

The **Public Safety Application** (also referred to as "Women Safety App" in the codebase) is a Flutter-based mobile application designed to enhance personal safety through technology. The app provides emergency response features, location tracking, and communication tools.

## What Does This Application Do?

This application serves as a comprehensive safety platform that connects three types of users:

1. **Children/Protected Persons** - Primary users who may need emergency assistance
2. **Parents/Guardians** - People who monitor and communicate with protected persons
3. **Police Officers** - Emergency responders who receive and act on distress signals

## Core Features

### 1. Emergency Distress System

The app allows users to send emergency alerts that include:
- Real-time location coordinates
- Address information
- Automatic SMS notifications to saved contacts

### 2. Location Sharing

Users can:
- Track their current GPS location
- Share location via Google Maps links
- Send location information to emergency contacts via SMS
- View real-time address details

As seen in `lib/view_model/bottom_sheat_view_model.dart`:

```dart
String msgBody =
    "https://maps.google.com/?daddr=${currentPosition!.latitude},${currentPosition!.longitude}  ,  $currentAddress";
```

### 3. Contact Management

The app stores emergency contacts locally using Hive database:
- Add and manage emergency contacts
- Store contact names and phone numbers
- Automatically send alerts to all saved contacts during emergencies

From `lib/model/contact_model.dart`:

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

### 4. In-App Chat System

The application provides a messaging system that enables:
- Communication between parents and children
- Text messages stored in Firebase Firestore
- Image sharing capabilities
- Audio message support
- Real-time message synchronization

### 5. Multi-User Authentication

The app supports different user types with Firebase Authentication:
- Email/password authentication
- User type identification (child, parent, police)
- Persistent login sessions
- Role-based interface customization

### 6. Background Services

The app can run background services for:
- Continuous location monitoring
- Emergency notifications
- SMS delivery

## Supported Platforms

Based on the repository structure, the app is configured for:
- **Android** (primary target)
- **iOS**
- **Windows** (desktop support)
- **Linux** (desktop support)
- **macOS**
- **Web** (partial support)

## Technology Stack Overview

### Frontend Framework
- **Flutter** - Cross-platform UI framework using Dart

### State Management
- **GetX** - For reactive state management and navigation

From `lib/main.dart`:

```dart
return GetMaterialApp(
  title: 'Woman Safety App',
  theme: ThemeData(
    textTheme: GoogleFonts.figtreeTextTheme(ThemeData.light().textTheme),
  ),
  // ...
);
```

### Backend Services
- **Firebase Authentication** - User authentication
- **Cloud Firestore** - Cloud database for chat and user data
- **Firebase Cloud Messaging** - Push notifications (configured)

### Local Storage
- **Hive** - Local NoSQL database for contacts
- **Shared Preferences** - For storing user session data

### Key Packages
- **geolocator** - GPS location services
- **geocoding** - Convert coordinates to addresses
- **background_sms** - Send SMS messages
- **permission_handler** - Manage app permissions
- **google_fonts** - Custom typography

## Use Cases

### For Children/Protected Persons
1. Save emergency contacts in the app
2. When in danger, activate the emergency button
3. App automatically sends location and alert SMS to all contacts
4. Chat with parents/guardians for regular communication

### For Parents/Guardians
1. Register as a parent user type
2. Receive emergency alerts from their children
3. Chat with children through the app
4. Monitor safety through communication

### For Police Officers
1. Register as police personnel
2. Receive distress signals from users
3. Access location information of distress calls
4. Respond to emergencies efficiently

## Key Differentiators

1. **Multi-Role System**: Unlike simple panic button apps, this supports three distinct user types
2. **Offline-First Emergency Contacts**: Uses local Hive storage for contacts, ensuring SMS can be sent even with poor internet
3. **Integrated Communication**: Combines emergency features with everyday parent-child chat
4. **Real-Time Location**: Provides both GPS coordinates and human-readable addresses

## Application Flow Summary

```
App Launch
    ↓
Check User Type (from Shared Preferences)
    ↓
┌───────────────┬──────────────┬──────────────┐
│               │              │              │
Child Screen   Parent Screen  Police Screen  Login Screen
(if logged in) (if logged in) (if logged in) (if not logged in)
    ↓               ↓              ↓              ↓
Home with      View Children   Distress      Email/Password Auth
Emergency      Chat with       Dashboard         ↓
Button         Children        View Alerts    Navigate to Role-Based Screen
```

## Summary

The Public Safety Application is a comprehensive safety solution that leverages modern mobile technologies to provide emergency response, location sharing, and communication features. It's built with Flutter for cross-platform compatibility and uses Firebase for backend services, making it scalable and reliable. The app's architecture supports multiple user roles, making it suitable for families and emergency response systems.

In the next chapter, we'll explore the application's architecture and how the code is organized.
