# Chapter 6: Chat System and Communication

## Overview

Beyond emergency features, the app provides a chat system enabling communication between parents and children. This chapter explores the **ChatViewModel** class and the Firestore-based messaging infrastructure.

## ChatViewModel Class

**Location:** `lib/view_model/chat_view_model.dart`

This ViewModel manages all chat-related functionality:
- Fetching messages from Firestore
- Sending text messages
- Sharing images and audio
- Identifying user type (parent vs. child)

### Class Structure

```dart
class ChatViewModel extends GetxController {
  final Rx<Timestamp?> date = Rx<Timestamp?>(null);
  var status = ''.obs;
  var name = ''.obs;
  var controller = TextEditingController();
  var imagePath = ''.obs;
  var imageUrls = ''.obs;

  final ChatFirestoreService firestoreService = ChatFirestoreService();
  final ChatMessageService messageService = ChatMessageService();
  final ChatImageService imageService = ChatImageService();
  final ChatAudioService audioService = ChatAudioService();

  @override
  onInit() {
    getstatus();
    super.onInit();
  }
  
  // Methods explained below
}
```

**State Variables:**
- `date` - Message timestamp
- `status` - User type ('child', 'parent', or 'police')
- `name` - Current user's name
- `controller` - TextEditingController for message input
- `imagePath` - Local path of selected image
- `imageUrls` - Firebase Storage URL of uploaded image

**Service Instances:**
- `firestoreService` - Handles Firestore queries
- `messageService` - Sends and deletes messages
- `imageService` - Image upload/selection
- `audioService` - Audio message handling

## Getting User Status

When ChatViewModel initializes, it fetches the current user's type:

```dart
getstatus() async {
  return await firestore
      .collection(usercollection)  // 'users'
      .doc(auth.currentUser!.uid)
      .get()
      .then((value) {
        status.value = value.data()!['type'];
        name.value = value.data()!['name'];
      }).catchError((e) {
        return showError(e.toString());
      });
}
```

**What it does:**
- Queries Firestore for current user's document
- Extracts `type` field ('child', 'parent', or 'police')
- Extracts user's `name`
- Stores in observable variables for UI access

## Firestore Chat Structure

The chat system uses a nested Firestore structure:

```
chat/
  └── {chatId}/
        ├── createdAt: Timestamp
        ├── fromId: "user1_uid"
        ├── toId: "user2_uid"
        └── message/  (subcollection)
              └── {messageId}/
                    ├── message: "Hello!"
                    ├── senderId: "user1_uid"
                    ├── receiverId: "user2_uid"
                    ├── createdAt: Timestamp
                    ├── type: "text" (or "image", "audio")
                    └── imageUrl: "..." (if type is "image")
```

### Chat ID Format

Chat IDs are constructed from both user IDs to ensure consistency:

```dart
// In ChatFirestoreService and ChatMessageService
String chatId = "${currentId}_${friendid}";  // Example: "abc123_def456"
```

This ensures both users reference the same conversation.

## Fetching Messages

### ChatFirestoreService

**Location:** `lib/data/services/chat_firestore_services.dart`

```dart
class ChatFirestoreService {
  Stream<QuerySnapshot> getChat(friendid) {
    return firestore
        .collection(chatcollection)  // 'chat'
        .doc("${currentId}_$friendid")
        .collection(msgcollection)   // 'message'
        .orderBy('createdAt', descending: true)
        .snapshots();
  }

  Stream<QuerySnapshot> getChildData() {
    return firestore
        .collection(usercollection)
        .where('type', isEqualTo: 'child')
        .snapshots();
  }

  Stream<QuerySnapshot> getParentData() {
    return firestore
        .collection(usercollection)
        .where('type', isEqualTo: 'parent')
        .snapshots();
  }
}
```

### Getting Chat Messages

```dart
Stream<QuerySnapshot> getChat(friendid) {
  return ChatViewModel().getChat(friendid);
}
```

**How it works:**
1. Constructs chat document path: `chat/{currentId}_{friendId}`
2. Accesses `message` subcollection
3. Orders by `createdAt` timestamp (newest first with `descending: true`)
4. Returns a **Stream** that emits updates in real-time

**Usage in UI:**
```dart
StreamBuilder<QuerySnapshot>(
  stream: chatViewModel.getChat(friendUserId),
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      var messages = snapshot.data!.docs;
      return ListView.builder(
        itemCount: messages.length,
        itemBuilder: (context, index) {
          var message = messages[index];
          return MessageBubble(
            text: message['message'],
            isSender: message['senderId'] == currentId,
          );
        },
      );
    }
    return loadingIndicator();
  },
)
```

### Getting Child Users (for Parents)

```dart
Stream<QuerySnapshot> getChildData() {
  return firestore
      .collection(usercollection)
      .where('type', isEqualTo: 'child')
      .snapshots();
}
```

**Purpose:** Parents see a list of children they can chat with.

### Getting Parent Users (for Children)

```dart
Stream<QuerySnapshot> getParentData() {
  return firestore
      .collection(usercollection)
      .where('type', isEqualTo: 'parent')
      .snapshots();
}
```

**Purpose:** Children see a list of parents/guardians to contact.

## Sending Messages

### ChatMessageService

**Location:** `lib/data/services/chat_message_services.dart`

```dart
class ChatMessageService {
  sendMessage({friendId, message, type}) {
    String chatid = "${currentId}_$friendId";
    firestore.collection(chatcollection).doc(chatid).set({
      'createdAt': FieldValue.serverTimestamp(),
      'fromId': currentId,
      'toId': friendId
    }, SetOptions(merge: true));

    var newdoc = firestore
        .collection(chatcollection)
        .doc(chatid)
        .collection(msgcollection)
        .doc();

    firestore.runTransaction((transaction) async {
      transaction.set(newdoc, {
        'message': message,
        'senderId': currentId,
        'receiverId': friendId,
        'createdAt': FieldValue.serverTimestamp(),
        'type': type,
      });
    }).then((value) {
      // Success
    }).catchError((e) {
      showError(e.toString());
    });
  }
}
```

**Step-by-step:**

#### Step 1: Create/Update Chat Document
```dart
String chatid = "${currentId}_$friendId";
firestore.collection(chatcollection).doc(chatid).set({
  'createdAt': FieldValue.serverTimestamp(),
  'fromId': currentId,
  'toId': friendId
}, SetOptions(merge: true));
```

- Creates chat document if it doesn't exist
- `SetOptions(merge: true)` prevents overwriting existing data
- Stores chat metadata (participants, creation time)

#### Step 2: Create New Message Document
```dart
var newdoc = firestore
    .collection(chatcollection)
    .doc(chatid)
    .collection(msgcollection)
    .doc();  // Auto-generates message ID
```

#### Step 3: Write Message in Transaction
```dart
firestore.runTransaction((transaction) async {
  transaction.set(newdoc, {
    'message': message,
    'senderId': currentId,
    'receiverId': friendId,
    'createdAt': FieldValue.serverTimestamp(),
    'type': type,  // 'text', 'image', or 'audio'
  });
})
```

**Why use a transaction?**
- Ensures atomic operation (all-or-nothing)
- Prevents partial writes if network fails

### Sending from ChatViewModel

```dart
sendMessage({friendId, message, type}) {
  return messageService.sendMessage(
      friendId: friendId, message: message, type: type);
}
```

**Usage in UI:**
```dart
ElevatedButton(
  onPressed: () {
    chatViewModel.sendMessage(
      friendId: selectedFriendId,
      message: messageController.text,
      type: 'text',
    );
    messageController.clear();
  },
  child: Icon(Icons.send),
)
```

## Deleting Messages

```dart
// In ChatMessageService
deleteMessage({friendId, docId}) {
  String chatid = "${currentId}_$friendId";
  return firestore
      .collection(chatcollection)
      .doc(chatid)
      .collection(msgcollection)
      .doc(docId)
      .delete()
      .then((value) {
        // Success
      }).catchError((e) {
        showError(e.toString());
      });
}
```

**Usage:**
```dart
chatViewModel.deleteMsg(
  friendid: friendUserId,
  docId: messageDocumentId,
);
```

## Image Sharing

### ChatImageService

**Location:** `lib/data/services/chat_image_services.dart`

```dart
class ChatImageService {
  selectImage({required ImageSource source}) async {
    try {
      final pickedFile = await ImagePicker().pickImage(source: source);
      if (pickedFile != null) {
        return pickedFile.path;
      }
    } catch (e) {
      showError("Error selecting image: $e");
    }
  }

  uploadImage({required String imagePath}) async {
    try {
      // Upload to Firebase Storage
      // Return download URL
    } catch (e) {
      showError("Error uploading image: $e");
    }
  }
}
```

### Selecting an Image

In ChatViewModel:

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

**Image sources:**
```dart
ImageSource.camera   // Take photo
ImageSource.gallery  // Choose from gallery
```

**Usage:**
```dart
// Camera
chatViewModel.selectImage(ImageSource.camera);

// Gallery
chatViewModel.selectImage(ImageSource.gallery);
```

### Uploading an Image

```dart
Future<void> uploadImage() async {
  try {
    var value = await imageService.uploadImage(imagePath: imagePath.value);
    if (value != null) {
      imageUrls.value = value;
    }
    print("value :${imageUrls.value} ");
  } catch (e) {
    showError("Error uploading image: $e");
  }
}
```

### Sending Image Message

```dart
// After upload completes
await chatViewModel.uploadImage();

if (chatViewModel.imageUrls.value.isNotEmpty) {
  chatViewModel.sendMessage(
    friendId: friendUserId,
    message: chatViewModel.imageUrls.value,  // Firebase Storage URL
    type: 'image',
  );
}
```

## Audio Messages

### ChatAudioService

**Location:** `lib/data/services/chat_audio_services.dart`

```dart
class ChatAudioService {
  // Audio recording and upload logic
}
```

Similar pattern to images:
1. Record audio
2. Upload to Firebase Storage
3. Send message with `type: 'audio'` and audio URL

## Chat UI Components

### Bottom Text Field

**Location:** `lib/res/components/chat/bottom_textfield.dart`

This component provides the message input interface:

```dart
class BottomTextField extends StatelessWidget {
  // TextField + Send button + Image/Audio buttons
}
```

Features:
- Text input field
- Send button
- Image attachment button
- Audio recording button
- Integration with ChatViewModel

### Message Display

**Location:** `lib/views/child/chat/message_screen.dart`

Displays messages in chat bubbles:

```dart
class MessageScreen extends StatelessWidget {
  // StreamBuilder with ListView of messages
  // Different bubbles for sender/receiver
  // Image/audio rendering based on type
}
```

## Chat Flow Diagram

```
User opens chat with friend
        |
        ↓
  ChatViewModel.getChat(friendId)
        |
        ↓
  Stream from Firestore
  (chat/{userId}_{friendId}/message)
        |
        ↓
  Real-time message updates
        |
        ↓
  Display in ListView
        |
[User types message]
        |
        ↓
  User taps Send
        |
        ↓
ChatViewModel.sendMessage(
  friendId,
  message,
  type: 'text'
)
        |
        ↓
ChatMessageService.sendMessage()
        |
        ↓
  1. Create/update chat document
  2. Add message to subcollection
  3. Set timestamp
        |
        ↓
  Firestore write complete
        |
        ↓
  Stream emits new message
        |
        ↓
  UI automatically updates
  (StreamBuilder rebuilds)
```

### Image Message Flow

```
User taps image button
        |
        ↓
  Choose: Camera or Gallery
        |
        ↓
ChatViewModel.selectImage(source)
        |
        ↓
  ImagePicker selects image
        |
        ↓
  imagePath.value = "/local/path"
        |
        ↓
  Show preview
        |
        ↓
  User confirms send
        |
        ↓
ChatViewModel.uploadImage()
        |
        ↓
  Upload to Firebase Storage
        |
        ↓
  imageUrls.value = "https://..."
        |
        ↓
ChatViewModel.sendMessage(
  friendId,
  message: imageUrls.value,
  type: 'image'
)
        |
        ↓
  Message saved to Firestore
        |
        ↓
  Receiver's UI shows image
```

## Real-time Updates

Firestore Streams provide real-time synchronization:

```dart
Stream<QuerySnapshot> getChat(friendid) {
  return firestore
      .collection(chatcollection)
      .doc("${currentId}_$friendid")
      .collection(msgcollection)
      .orderBy('createdAt', descending: true)
      .snapshots();  // Real-time stream
}
```

**How it works:**
1. `.snapshots()` creates a listener on Firestore
2. Whenever a message is added/updated/deleted, Firestore pushes an update
3. StreamBuilder rebuilds automatically
4. Users see new messages instantly without refresh

## Firebase Constants

From `lib/res/const/firebase_const.dart`:

```dart
var usercollection = 'users';
var chatcollection = 'chat';
var msgcollection = 'message';
```

These constants ensure consistent collection names throughout the app.

## Error Handling

All chat operations include error handling:

```dart
.catchError((e) {
  return showError(e.toString());
});
```

Common errors:
- Network connectivity issues
- Permission denied (Firebase rules)
- Invalid data formats
- Storage upload failures

## Summary

The chat system:

1. **Uses Firestore** for real-time message synchronization
2. **Supports multiple message types** (text, image, audio)
3. **Organizes chats** with nested collections (chat → messages)
4. **Provides real-time updates** via Firestore Streams
5. **Separates concerns** with service classes:
   - `ChatFirestoreService` - Queries
   - `ChatMessageService` - Message operations
   - `ChatImageService` - Image handling
   - `ChatAudioService` - Audio handling
6. **Manages state** with ChatViewModel (GetX)

The MVVM architecture makes the chat system:
- **Testable** - Business logic separated from UI
- **Reusable** - Services can be used in different contexts
- **Maintainable** - Clear separation of responsibilities

In the final chapter, we'll explore the reusable UI components and styling system.