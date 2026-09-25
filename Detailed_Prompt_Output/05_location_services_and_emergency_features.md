# Chapter 5: Location Services and Emergency Features

## Overview

The core purpose of this safety application is to share location information during emergencies. This chapter explains the **BottomSheetControllers** class, which manages GPS tracking, address lookup, and emergency SMS messaging.

## BottomSheetControllers Class

**Location:** `lib/view_model/bottom_sheat_view_model.dart`

This is one of the most important classes in the application. It handles:
- GPS location tracking
- Converting coordinates to human-readable addresses (reverse geocoding)
- Sending emergency SMS messages
- Managing location and SMS permissions

### Class Structure

```dart
class BottomSheetControllers extends GetxController {
  RxBool isLoading = false.obs;
  Position? currentPosition;
  String? currentAddress = 'Your Address/Location';
  RxString realtimeAddress = "Your Address/Location".obs;
  LocationPermission? permission;

  @override
  void onInit() {
    super.onInit();
    getCurrentLocation();
  }
  
  // Methods explained below...
}
```

**State Variables:**
- `isLoading` - Observable boolean for loading indicator
- `currentPosition` - Stores GPS coordinates (latitude/longitude)
- `currentAddress` - Human-readable address string
- `realtimeAddress` - Observable address for UI updates
- `permission` - Current location permission status

## Permission Management

### Requesting Permissions

```dart
requestPermissions() async {
  await Permission.location.request();
  await Permission.sms.request();
}
```

Requests two critical permissions:
1. **Location** - For GPS tracking
2. **SMS** - For sending emergency messages

### Checking SMS Permission

```dart
grantedPermission() async {
  return await Permission.sms.status.isGranted;
}
```

Returns `true` if SMS permission is granted, `false` otherwise.

## Location Tracking

### Getting Current Location

The `getCurrentLocation()` method is the heart of location tracking:

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

  if (permission == LocationPermission.deniedForever) {
    return Future.error(
        'Location permissions are permanently denied, we cannot request permissions.');
  }

  try {
    currentPosition = await Geolocator.getCurrentPosition(
        forceAndroidLocationManager: true,
        desiredAccuracy: LocationAccuracy.high);

    if (currentPosition != null) {
      getCurrentAddress();
      update();
    } else {
      showError('Failed to obtain position, currentPosition is null.');
    }
  } catch (e) {
    openLocationSettings();
    showError('Failed to get current position: $e');
  }
  isLoading.value = false;
}
```

**Step-by-step breakdown:**

#### Step 1: Set Loading State
```dart
isLoading.value = true;
```
Triggers loading indicator in UI

#### Step 2: Request Permissions
```dart
requestPermissions();
```
Ensures location and SMS permissions are requested

#### Step 3: Check Permission Status
```dart
permission = await Geolocator.checkPermission();
if (permission == LocationPermission.denied) {
  permission = await Geolocator.requestPermission();
  // ...
}
```

**Permission states:**
- `denied` - Not yet granted, can request
- `deniedForever` - User permanently denied, must open settings
- `granted` - Permission granted

#### Step 4: Get GPS Coordinates
```dart
currentPosition = await Geolocator.getCurrentPosition(
    forceAndroidLocationManager: true,
    desiredAccuracy: LocationAccuracy.high);
```

**Parameters:**
- `forceAndroidLocationManager: true` - Use Android's native location manager
- `desiredAccuracy: LocationAccuracy.high` - Request high accuracy GPS

**Result:** A `Position` object containing:
```dart
currentPosition.latitude   // e.g., 37.7749
currentPosition.longitude  // e.g., -122.4194
```

#### Step 5: Convert to Address
```dart
if (currentPosition != null) {
  getCurrentAddress();
  update();
}
```

Calls the address lookup method (explained next)

### Reverse Geocoding (Address Lookup)

The `getCurrentAddress()` method converts coordinates to a readable address:

```dart
getCurrentAddress() async {
  isLoading.value = true;
  try {
    List<Placemark> placemarks = await placemarkFromCoordinates(
        currentPosition!.latitude, currentPosition!.longitude);

    if (placemarks.isNotEmpty) {
      Placemark place = placemarks[0];
      currentAddress =
          "${place.locality}, ${place.street}, ${place.postalCode}, ${place.name}, ${place.subAdministrativeArea}";
      realtimeAddress.value = currentAddress!;
    } else {
      showError('No address found for the given coordinates.');
    }

    update();
  } catch (e) {
    showError('Failed to get current address: $e');
  }
  isLoading.value = false;
}
```

**How it works:**

1. **placemarkFromCoordinates()** - Geocoding package function that queries a geocoding service
2. **Placemark object** - Contains address components:
   - `locality` - City (e.g., "San Francisco")
   - `street` - Street name (e.g., "Market Street")
   - `postalCode` - ZIP code (e.g., "94103")
   - `name` - Specific location name
   - `subAdministrativeArea` - County/region

3. **Build address string:**
```dart
currentAddress = "${place.locality}, ${place.street}, ${place.postalCode}, ${place.name}, ${place.subAdministrativeArea}";
```

Example result:
```
"San Francisco, Market Street, 94103, Civic Center, San Francisco County"
```

### Updating Location

```dart
updateLocation() async {
  isLoading.value = true;
  await getCurrentAddress();
  await getCurrentLocation();
  isLoading.value = false;
}
```

Refreshes both GPS coordinates and address. Used when user manually requests location update.

### Opening Location Settings

```dart
void openLocationSettings() {
  Geolocator.openLocationSettings();
}
```

Opens device's location settings if permission is denied forever.

## Emergency SMS System

### Sending Individual SMS

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

**Parameters:**
- `phoneNumber` - Recipient's phone number
- `msg` - Message content
- `sim` - Optional SIM card slot (for dual-SIM phones)

**Uses:** `background_sms` package to send SMS even when app is in background

### Sending Location to All Contacts

The `sendLocation()` method is called during emergencies:

```dart
sendLocation() async {
  final contactBox = Boxes.getContacts();
  if (contactBox == null || contactBox.isEmpty) {
    showError('No contacts available to send location');
    return;
  }

  String msgBody =
      "https://maps.google.com/?daddr=${currentPosition!.latitude},${currentPosition!.longitude}  ,  $currentAddress";

  final contactList = contactBox.values.toList();
  List<Future> smsSendingTasks = [];

  if (await grantedPermission()) {
    for (var element in contactList) {
      smsSendingTasks
          .add(sendMsg(element.phoneNumber, "I am in trouble $msgBody"));
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

**Step-by-step:**

#### Step 1: Get Emergency Contacts
```dart
final contactBox = Boxes.getContacts();
if (contactBox == null || contactBox.isEmpty) {
  showError('No contacts available to send location');
  return;
}
```

Retrieves contacts from Hive database (local storage).

**Boxes helper** (from `lib/data/hive db/boxes.dart`):
```dart
class Boxes {
  static Box<ContactModel> getContacts() =>
      Hive.box<ContactModel>('contactsData');
}
```

#### Step 2: Build Message
```dart
String msgBody =
    "https://maps.google.com/?daddr=${currentPosition!.latitude},${currentPosition!.longitude}  ,  $currentAddress";
```

**Message format:**
```
https://maps.google.com/?daddr=37.7749,-122.4194  ,  San Francisco, Market Street, 94103...
```

The Google Maps URL opens directions to the location when clicked.

#### Step 3: Send to All Contacts
```dart
final contactList = contactBox.values.toList();
List<Future> smsSendingTasks = [];

for (var element in contactList) {
  smsSendingTasks
      .add(sendMsg(element.phoneNumber, "I am in trouble $msgBody"));
}
```

**How it works:**
- Iterates through all saved contacts
- Creates a list of Future tasks (asynchronous operations)
- Each task sends an SMS to one contact

#### Step 4: Wait for All Messages
```dart
await Future.wait(smsSendingTasks);
showSuccess('All messages sent successfully', '');
```

`Future.wait()` waits for all SMS operations to complete before showing success.

## ContactModel Structure

Emergency contacts are stored using this model (`lib/model/contact_model.dart`):

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

**Hive annotations:**
- `@HiveType(typeId: 0)` - Identifies this type to Hive
- `@HiveField(0)` - Marks fields for storage

## Usage in UI

### Initializing the Controller

```dart
var controller = Get.put(BottomSheetControllers());
```

Creates and registers the controller with GetX.

### Displaying Current Address

```dart
Obx(
  () => Text(
    controller.realtimeAddress.value,
    style: TextStyle(fontSize: 16),
  ),
)
```

The `Obx` widget automatically rebuilds when `realtimeAddress` changes.

### Emergency Button

```dart
ElevatedButton(
  onPressed: () async {
    await controller.updateLocation();  // Refresh location
    await controller.sendLocation();    // Send to contacts
  },
  child: Text('SEND EMERGENCY ALERT'),
)
```

### Manual Location Update

```dart
IconButton(
  icon: Icon(Icons.refresh),
  onPressed: () {
    controller.updateLocation();
  },
)
```

### Show Location Dialog

From `lib/res/utils/utils.dart`, there's a utility to display location info:

```dart
void showLocationDialog(currentAddress, currentPosition) {
  Get.defaultDialog(
    title: "Location Information",
    content: Container(
      padding: const EdgeInsets.all(20),
      child: Column(
        children: [
          RichText(
            text: TextSpan(
              children: [
                TextSpan(text: "Address: "),
                TextSpan(text: "$currentAddress"),
              ],
            ),
          ),
          RichText(
            text: TextSpan(
              children: [
                TextSpan(text: "Latitude: "),
                TextSpan(text: "${currentPosition?.latitude}"),
              ],
            ),
          ),
          RichText(
            text: TextSpan(
              children: [
                TextSpan(text: "Longitude: "),
                TextSpan(text: "${currentPosition?.longitude}"),
              ],
            ),
          ),
        ],
      ),
    ),
    // ...
  );
}
```

Usage:
```dart
showLocationDialog(
  controller.currentAddress,
  controller.currentPosition,
);
```

## Location Flow Diagram

```
User opens app / Taps emergency button
          |
          ↓
    getCurrentLocation()
          |
          ↓
  Check Location Permission
          |
    ┌─────┴─────┐
    │           │
Granted      Denied
    │           │
    │           ↓
    │    Request Permission
    │           │
    │      ┌────┴────┐
    │      │         │
    │   Granted   Denied Forever
    │      │         │
    │      │         ↓
    │      │    Open Settings
    └──────┘
          │
          ↓
Geolocator.getCurrentPosition()
          |
          ↓
  Receive Coordinates
  (latitude, longitude)
          |
          ↓
  getCurrentAddress()
          |
          ↓
  placemarkFromCoordinates()
          |
          ↓
  Build readable address
          |
          ↓
  Update UI (realtimeAddress)
          |
  [Emergency Triggered?]
          |
          ↓
    sendLocation()
          |
          ↓
  Get contacts from Hive
          |
          ↓
  Build message:
  "I am in trouble
   https://maps.google.com/?daddr=lat,lng
   Address..."
          |
          ↓
  For each contact:
    sendMsg(phone, message)
          |
          ↓
  BackgroundSms.sendMessage()
          |
          ↓
  Show success/error message
```

## Error Handling

The class includes comprehensive error handling:

1. **Permission Denied:**
```dart
if (permission == LocationPermission.denied) {
  return Future.error('Location permissions are denied');
}
```

2. **No Location Obtained:**
```dart
if (currentPosition == null) {
  showError('Failed to obtain position, currentPosition is null.');
}
```

3. **Geocoding Failure:**
```dart
catch (e) {
  showError('Failed to get current address: $e');
}
```

4. **No Emergency Contacts:**
```dart
if (contactBox == null || contactBox.isEmpty) {
  showError('No contacts available to send location');
  return;
}
```

## Summary

The **BottomSheetControllers** class is the core of the app's safety features:

1. **Tracks GPS location** with high accuracy using Geolocator
2. **Converts coordinates to addresses** using reverse geocoding
3. **Manages permissions** for location and SMS
4. **Sends emergency SMS** to all saved contacts
5. **Includes Google Maps links** for easy navigation
6. **Handles errors gracefully** with user-friendly messages

The emergency SMS feature works offline (after initial location fetch) because:
- Contacts stored locally in Hive
- SMS doesn't require internet
- Location can be cached from last successful fetch

This makes it reliable even in areas with poor connectivity.

In the next chapter, we'll explore the chat system for parent-child communication.