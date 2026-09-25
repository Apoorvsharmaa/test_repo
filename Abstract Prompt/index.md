# Public Safety Application - Documentation

Welcome to the documentation for the Public Safety Application. This Flutter-based mobile application is designed to enhance personal safety through real-time communication, location tracking, and emergency alert systems.

## Overview

This application serves three types of users:
- **Children/Users** - Can send distress signals, share location, and communicate with parents
- **Parents/Guardians** - Can monitor their children and receive emergency alerts
- **Police** - Can respond to distress signals and monitor emergency situations in their area

## Documentation Chapters

### [1. Application Overview and Architecture](01_application_overview_and_architecture.md)
Learn about the application's purpose, overall architecture, user types, and how different components work together to provide safety features.

### [2. Application Setup and Initialization](02_application_setup_and_initialization.md)
Understand how the application starts, including Firebase setup, database initialization, permission handling, and the initial routing logic for different user types.

### [3. Core Features and View Models](03_core_features_and_view_models.md)
Explore the main features of the application including location sharing, distress signals, emergency messaging, chat functionality, and authentication through detailed examination of key view models and services.

## Quick Start

The application is built with:
- **Flutter/Dart** for cross-platform mobile development
- **Firebase** for authentication and cloud storage
- **Hive** for local data storage
- **GetX** for state management
- **Google Maps** for location services

## Key Technologies

- Firebase Authentication & Firestore
- Background SMS services
- Geolocation and geocoding
- Local push notifications
- Real-time database synchronization

---

**Note**: This documentation is based on the codebase analysis and provides a beginner-friendly guide to understanding the application structure and functionality.