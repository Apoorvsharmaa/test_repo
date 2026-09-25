# Chapter 7: UI Components and Styling

## Overview

The app uses a consistent design system with reusable components. This chapter explores the UI building blocks and styling utilities that maintain visual consistency throughout the application.

## Color System

**Location:** `lib/res/colors/colors.dart`

The app defines a centralized color palette:

```dart
import 'package:flutter/material.dart';

Color white = Colors.white;
Color black = Colors.black;

List<Color> gradien = const [
  Color(0xFFFD8080),
  Color(0xFFFB8580),
  Color(0xFFFBD079),
];

Color primaryColor = const Color(0xfffc3b77);
Color policePrimary = const Color.fromARGB(255, 15, 0, 125);
```

### Color Breakdown

**Primary Color:**
```dart
Color primaryColor = const Color(0xfffc3b77);  // Pink/Magenta
```
- Used for main buttons, accents, and branding
- Hex: `#fc3b77`
- RGB: `(252, 59, 119)`

**Police Primary:**
```dart
Color policePrimary = const Color.fromARGB(255, 15, 0, 125);  // Dark Blue
```
- Used specifically for police user interfaces
- RGB: `(15, 0, 125)`

**Gradient Colors:**
```dart
List<Color> gradien = const [
  Color(0xFFFD8080),  // Light red/pink
  Color(0xFFFB8580),  // Coral
  Color(0xFFFBD079),  // Light orange
];
```
- Used for gradient backgrounds in UI elements
- Creates smooth color transitions

### Using Colors in Widgets

```dart
import 'package:women_safety_app/res/colors/colors.dart';

Container(
  color: primaryColor,
  child: Text(
    'Hello',
    style: TextStyle(color: white),
  ),
)
```

## Reusable UI Components

### PrimaryButton Component

**Location:** `lib/res/components/common/primary_button.dart`

A standardized button used throughout the app:

```dart
class PrimaryButton extends StatelessWidget {
  final String title;
  final Function onPressed;

  const PrimaryButton({
    Key? key,
    required this.title,
    required this.onPressed,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: 60,
      width: double.infinity,
      child: ElevatedButton(
        onPressed: () {
          onPressed();
        },
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

**Features:**
- Fixed height: 60 pixels
- Full width: `double.infinity`
- Rounded corners: 30px border radius
- Primary color background
- White text at 18px font size

**Usage:**
```dart
PrimaryButton(
  title: 'LOGIN',
  onPressed: () {
    // Handle button press
    print('Login pressed');
  },
)
```

**Example from Login Screen:**
```dart
PrimaryButton(
  title: 'LOGIN',
  onPressed: () {
    loginViewModel.onSaveValue();
  },
)
```

### CustomTextField Component

**Location:** `lib/res/components/common/custom_textfield.dart`

A flexible, reusable text input field:

```dart
class CustomTextField extends StatelessWidget {
  final String? hintText;
  final TextEditingController? controller;
  final String? Function(String?)? validate;
  final Function(String?)? onsave;
  final int? maxLines;
  final bool isPassword;
  final bool enable;
  final bool? check;
  final TextInputType? keyboardtype;
  final TextInputAction? textInputAction;
  final FocusNode? focusNode;
  final Widget? prefix;
  final Widget? suffix;

  CustomTextField({
    this.controller,
    this.check,
    this.enable = true,
    this.focusNode,
    this.hintText,
    this.isPassword = false,
    this.keyboardtype,
    this.maxLines,
    this.onsave,
    this.prefix,
    this.suffix,
    this.textInputAction,
    this.validate
  });

  @override
  Widget build(BuildContext context) {
    return TextFormField(
      enabled: enable,
      maxLines: maxLines ?? 1,
      onSaved: onsave,
      focusNode: focusNode,
      textInputAction: textInputAction,
      keyboardType: keyboardtype ?? TextInputType.name,
      controller: controller,
      validator: validate,
      obscureText: isPassword,
      decoration: InputDecoration(
        prefixIcon: prefix,
        suffixIcon: suffix,
        labelText: hintText ?? "hint text..",
        focusedBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(30),
          borderSide: BorderSide(
            style: BorderStyle.solid,
            color: Theme.of(context).primaryColor,
          ),
        ),
        enabledBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(30),
          borderSide: const BorderSide(
            style: BorderStyle.solid,
            color: Color(0xFF909A9E),
          ),
        ),
        focusedErrorBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(30),
          borderSide: BorderSide(
            style: BorderStyle.solid,
            color: Theme.of(context).primaryColor,
          ),
        ),
        errorBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(30),
          borderSide: const BorderSide(
            style: BorderStyle.solid,
            color: Colors.red,
          ),
        ),
      ),
    );
  }
}
```

**Features:**
- Rounded borders (30px radius)
- Optional password masking
- Validation support
- Prefix/suffix icons
- Multiple keyboard types
- Customizable max lines
- Enable/disable state

**Border States:**
1. **Enabled Border** - Gray (`#909A9E`)
2. **Focused Border** - Primary color
3. **Error Border** - Red
4. **Focused Error Border** - Primary color

**Usage Examples:**

**Email Field:**
```dart
CustomTextField(
  hintText: 'Enter email',
  textInputAction: TextInputAction.next,
  keyboardtype: TextInputType.emailAddress,
  prefix: const Icon(Icons.email),
  validate: (email) {
    if (email!.isEmpty || !email.contains('@')) {
      return 'Enter correct email';
    }
    return null;
  },
  onsave: (email) {
    formdata['email'] = email ?? "";
  },
)
```

**Password Field:**
```dart
CustomTextField(
  isPassword: true,
  hintText: 'Enter password',
  prefix: const Icon(Icons.lock),
  suffix: IconButton(
    icon: Icon(Icons.visibility_off),
    onPressed: () {
      // Toggle password visibility
    },
  ),
  validate: (password) {
    if (password!.isEmpty || password.length < 7) {
      return 'Enter correct password';
    }
    return null;
  },
)
```

**Multi-line Field:**
```dart
CustomTextField(
  hintText: 'Enter message',
  maxLines: 5,
  keyboardtype: TextInputType.multiline,
  controller: messageController,
)
```

### SecondaryButton Component

**Location:** `lib/res/components/common/secondary_button.dart`

A text-button variant for secondary actions:

```dart
class SecondaryButton extends StatelessWidget {
  final String title;
  final Function onPress;
  // ...
  
  @override
  Widget build(BuildContext context) {
    return TextButton(
      onPressed: () => onPress(),
      child: Text(title),
    );
  }
}
```

**Usage:**
```dart
SecondaryButton(
  title: 'Click here',
  onPress: () {
    Get.to(() => const ForgetScreen());
  },
)
```

## Utility Functions

**Location:** `lib/res/utils/utils.dart`

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

**Usage:**
```dart
Obx(
  () => isLoading.value
      ? loadingIndicator()
      : ActualContent(),
)
```

### Error Messages

```dart
showError(message) {
  Get.snackbar(
    "Error",
    message,
    backgroundColor: Colors.red,
    colorText: Colors.white,
    snackPosition: SnackPosition.BOTTOM
  );
}
```

**Usage:**
```dart
if (email.isEmpty) {
  showError('Email is required');
}
```

**Visual appearance:**
- Red background
- White text
- Appears at bottom of screen
- Auto-dismisses after a few seconds

### Success Messages

```dart
showSuccess(title, message) {
  Get.snackbar(
    title,
    message,
    backgroundColor: Colors.white,
    colorText: Colors.black,
    snackPosition: SnackPosition.BOTTOM
  );
}
```

**Usage:**
```dart
showSuccess('Success', 'Message sent successfully');
```

### Alert Dialog

```dart
void showAlertDialog(title, msg) {
  Get.defaultDialog(
    title: title,
    content: Text(msg),
    confirm: ElevatedButton(
      onPressed: () {
        Get.back(); // Close the dialog
      },
      child: Text("OK"),
    ),
  );
}
```

**Usage:**
```dart
showAlertDialog(
  'Confirm Action',
  'Are you sure you want to proceed?'
);
```

### Location Dialog

```dart
void showLocationDialog(currentAddress, currentPosition) {
  Get.defaultDialog(
    contentPadding: EdgeInsets.zero,
    title: "Location Information",
    content: Container(
      padding: const EdgeInsets.all(20),
      child: Column(
        children: [
          RichText(
            text: TextSpan(
              style: const TextStyle(fontSize: 18, color: Colors.black),
              children: [
                const TextSpan(
                  text: "Address: ",
                  style: TextStyle(
                      fontWeight: FontWeight.normal, color: Colors.blue),
                ),
                TextSpan(text: "$currentAddress"),
              ],
            ),
          ),
          const SizedBox(height: 10),
          RichText(
            text: TextSpan(
              style: const TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                  color: Colors.black),
              children: [
                const TextSpan(
                  text: "Latitude: ",
                  style: TextStyle(
                      fontWeight: FontWeight.normal, color: Colors.blue),
                ),
                TextSpan(text: "${currentPosition?.latitude}"),
              ],
            ),
          ),
          RichText(
            text: TextSpan(
              children: [
                const TextSpan(
                  text: "Longitude: ",
                  style: TextStyle(color: Colors.blue),
                ),
                TextSpan(text: "${currentPosition?.longitude}"),
              ],
            ),
          ),
          const Divider(),
        ],
      ),
    ),
    confirm: ElevatedButton(
      style: ElevatedButton.styleFrom(backgroundColor: primaryColor),
      onPressed: () {
        Get.back();
      },
      child: const Text(
        "OK",
        style: TextStyle(fontSize: 16, color: Colors.white),
      ),
    ),
  );
}
```

**Features:**
- Shows formatted address
- Displays latitude and longitude
- Blue labels for field names
- Primary color confirm button

**Usage:**
```dart
showLocationDialog(
  controller.currentAddress,
  controller.currentPosition,
);
```

## Spacing Components

### Vertical Space

**Location:** `lib/res/components/spacing/vspace.dart`

```dart
class Vspace extends StatelessWidget {
  final double height;

  const Vspace({Key? key, required this.height}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SizedBox(height: height);
  }
}
```

**Usage:**
```dart
Column(
  children: [
    Text('First item'),
    Vspace(height: 20),
    Text('Second item'),
  ],
)
```

### Horizontal Space

**Location:** `lib/res/components/spacing/hspace.dart`

```dart
class HSpace extends StatelessWidget {
  final double width;

  const HSpace({Key? key, required this.width}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SizedBox(width: width);
  }
}
```

**Usage:**
```dart
Row(
  children: [
    Text('Left'),
    HSpace(width: 15),
    Text('Right'),
  ],
)
```

## Typography

### Google Fonts Integration

From `lib/main.dart`:

```dart
GetMaterialApp(
  theme: ThemeData(
    textTheme: GoogleFonts.figtreeTextTheme(ThemeData.light().textTheme),
  ),
  // ...
)
```

**What it does:**
- Sets "Figtree" as the default font family for all text
- Applies to all Text widgets automatically
- No need to specify font in individual widgets

### Custom Text Styles

While the app uses default styles, you can create custom ones:

```dart
Text(
  'Welcome',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: primaryColor,
  ),
)
```

## Component Organization

The `lib/res/components/` directory is organized by feature:

```
components/
├── common/              # Generic, reusable components
│   ├── primary_button.dart
│   ├── custom_textfield.dart
│   └── secondary_button.dart
│
├── chat/                # Chat-specific components
│   └── bottom_textfield.dart
│
├── emergency/           # Emergency feature components
│   ├── custom_emergency_componenet.dart
│   └── emergency.dart
│
├── home/                # Home screen components
│   ├── appbar.dart
│   └── custom_slider.dart
│
├── share_location/      # Location sharing components
│   ├── bottomsheat.dart
│   └── share_location.dart
│
├── spacing/             # Layout helpers
│   ├── hspace.dart
│   └── vspace.dart
│
└── dialogues/           # Dialog components
    └── video_dialog.dart
```

## Design Patterns in Components

### Composition Over Inheritance

Components accept child widgets:

```dart
PrimaryButton(
  title: 'Click Me',
  onPressed: () {},
)
```

Instead of extending:
```dart
// Not used in this app
class LoginButton extends PrimaryButton { ... }
```

### Configuration via Constructor

All customization through parameters:

```dart
CustomTextField(
  hintText: 'Email',
  keyboardtype: TextInputType.emailAddress,
  prefix: Icon(Icons.email),
  validate: (value) => ...,
)
```

### Stateless Preference

Most components are StatelessWidgets:
- More performant
- Easier to reason about
- State managed by ViewModels

## Theming Best Practices

### Centralized Colors

✅ **Good:**
```dart
import 'package:women_safety_app/res/colors/colors.dart';

Container(color: primaryColor);
```

❌ **Avoid:**
```dart
Container(color: Color(0xfffc3b77));  // Hardcoded
```

### Reusable Components

✅ **Good:**
```dart
PrimaryButton(
  title: 'Submit',
  onPressed: handleSubmit,
)
```

❌ **Avoid:**
```dart
ElevatedButton(
  style: ElevatedButton.styleFrom(
    backgroundColor: Color(0xfffc3b77),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(30)
    ),
  ),
  onPressed: handleSubmit,
  child: Text('Submit'),
)  // Duplicates styling
```

### Consistent Spacing

✅ **Good:**
```dart
Vspace(height: 20);
```

❌ **Avoid:**
```dart
SizedBox(height: 20);  // Direct usage everywhere
```

## Summary

The UI system provides:

1. **Centralized color palette** for brand consistency
2. **Reusable components** (PrimaryButton, CustomTextField)
3. **Utility functions** for common UI patterns (loading, errors, dialogs)
4. **Spacing helpers** for consistent layouts
5. **Google Fonts integration** for typography
6. **Organized component structure** by feature

**Benefits:**
- **Consistency**: Same look and feel throughout the app
- **Maintainability**: Change styles in one place
- **Productivity**: Faster UI development with pre-built components
- **Flexibility**: Components accept customization parameters

**Component Philosophy:**
- Small, focused components
- Configuration over code duplication
- Stateless when possible
- Composition for complex UIs

This concludes the Public Safety Application documentation. You now understand:
1. What the app does and its features
2. The MVVM architecture and project structure
3. Application initialization and routing
4. Authentication and user management
5. Location services and emergency features
6. Chat system and real-time communication
7. UI components and styling system

The codebase follows Flutter best practices with clear separation of concerns, making it maintainable and scalable for future enhancements.