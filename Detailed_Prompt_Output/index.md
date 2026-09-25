# Public Safety Application - Documentation

Welcome to the Public Safety Application documentation! This guide will help you understand how this Flutter-based women's safety application works.

## About This Project

The Public Safety Application is a comprehensive safety app built with Flutter that enables users to:
- Send emergency distress signals with location information
- Share real-time location via SMS to emergency contacts
- Communicate through an in-app chat system
- Support three user roles: Child (protected person), Parent (guardian), and Police (emergency responder)

## Documentation Chapters

This documentation is organized into the following chapters:

1. [**Project Overview and Features**](01_project_overview_and_features.md)
   - What the application does
   - Key features and capabilities
   - Target users and use cases

2. [**Architecture and Project Structure**](02_architecture_and_project_structure.md)
   - Overall architecture (MVVM pattern)
   - Folder organization
   - Key dependencies and technologies

3. [**Application Entry Point and Initialization**](03_application_entry_point_and_initialization.md)
   - How the app starts
   - Initialization process
   - Permission handling
   - User routing logic

4. [**Authentication and User Management**](04_authentication_and_user_management.md)
   - User types (child, parent, police)
   - Login system and Firebase authentication
   - Session management

5. [**Location Services and Emergency Features**](05_location_services_and_emergency_features.md)
   - BottomSheetControllers class
   - Location tracking functionality
   - SMS emergency messaging
   - Permission handling

6. [**Chat System and Communication**](06_chat_system_and_communication.md)
   - ChatViewModel architecture
   - Firestore integration
   - Parent-child messaging
   - Image and audio sharing

7. [**UI Components and Styling**](07_ui_components_and_styling.md)
   - Reusable components (PrimaryButton, CustomTextField)
   - Color scheme and theming
   - Utility functions

## Getting Started

If you're new to this codebase:
1. Start with [Chapter 1](01_project_overview_and_features.md) to understand what the app does
2. Read [Chapter 2](02_architecture_and_project_structure.md) to grasp the overall structure
3. Follow [Chapter 3](03_application_entry_point_and_initialization.md) to see how the app initializes
4. Explore specific features in Chapters 4-7 based on your interests

## Prerequisites

To work with this codebase, you should have:
- Basic understanding of Dart programming language
- Familiarity with Flutter framework
- Knowledge of Firebase services (optional but helpful)
- Understanding of state management concepts (GetX is used here)

## Quick Navigation

- **For understanding the app flow**: Chapters 3 → 4
- **For location/emergency features**: Chapter 5
- **For chat functionality**: Chapter 6
- **For UI development**: Chapter 7

---

*This documentation is generated from the source code and reflects the current state of the repository.*
