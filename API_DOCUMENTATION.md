# MeshNotes API Documentation

## Table of Contents

1. [Overview](#overview)
2. [Application Entry Points](#application-entry-points)
3. [Core Controller System](#core-controller-system)
4. [Mind Editor Components](#mind-editor-components)
5. [Document Management](#document-management)
6. [Networking & P2P Communication](#networking--p2p-communication)
7. [Plugin System](#plugin-system)
8. [UI Components & Pages](#ui-components--pages)
9. [Utility Functions](#utility-functions)
10. [Settings Management](#settings-management)
11. [Examples](#examples)

## Overview

MeshNotes is a collaborative note-taking application built with Flutter that features:
- Real-time collaborative editing
- P2P networking for synchronization
- Plugin architecture for extensibility
- Cross-platform support (Android, iOS, Windows, macOS, Linux)
- Mind-mapping style document editing

## Application Entry Points

### main.dart

The main entry point for the application.

```dart
void main(List<String> args) async
```

**Parameters:**
- `args`: Command line arguments

**Usage:**
```dart
// Run as client application
void main() async {
  _runApp();
}

// Run as server
void main(['server', '8080']) async {
  serverInit();
  runServer();
}
```

### appInit()

Initializes the application components.

```dart
Future<void> appInit() async
```

**Features:**
- Initializes Flutter bindings
- Sets up logging system
- Initializes the main controller
- Configures system UI overlay for Android

**Example:**
```dart
await appInit();
runApp(const MeshApp());
```

## Core Controller System

### Controller

The main controller class that manages the entire application state.

```dart
class Controller
```

**Key Properties:**
- `DocumentManager docManager`: Manages documents
- `Document? document`: Current active document
- `PluginManager pluginManager`: Manages plugins
- `NetworkController network`: Handles networking
- `Environment environment`: Platform detection
- `Device device`: Device information
- `Setting setting`: Application settings

**Main Methods:**

#### Initialization
```dart
Future<bool> initAll() async
```
Initializes all application components including database, network, plugins, and settings.

#### Document Management
```dart
void openDocument(String docId)
void newDocument()
void closeDocument()
void deleteDocument()
```

#### User Management
```dart
void setUserPrivateInfo(EncryptedUserPrivateInfo userInfo, String password)
UserPrivateInfo? getUserPrivateInfo()
bool changeUserInfo(UserPrivateInfo newUserInfo, String? newPassword)
```

#### Network Operations
```dart
bool tryStartingNetwork()
void sendVersionBroadcast()
bool tryToSaveAndSendVersionTree()
```

**Example:**
```dart
final controller = Controller();
await controller.initAll();
controller.newDocument();
controller.openDocument('doc123');
```

## Mind Editor Components

### MindEditor

The main editor widget for document editing.

```dart
class MindEditor extends StatefulWidget
```

**Properties:**
- `Key? key`: Widget key

**Methods:**
```dart
void open(Document doc)
void close()
```

**Example:**
```dart
MindEditor(
  key: Key('editor'),
)
```

### MindEditField

The core editing field that handles text input and document rendering.

```dart
class MindEditField extends StatefulWidget
```

**Properties:**
- `Controller controller`: Main controller
- `bool isReadOnly`: Read-only mode flag
- `FocusNode focusNode`: Focus management
- `Document document`: Document to edit

**Key Methods:**
```dart
void refreshDoc({String? activeBlockId, int position = 0})
void pasteText(String text)
void requestKeyboard()
void hideKeyboard()
List<Widget> getReadOnlyBlocks()
```

**Example:**
```dart
MindEditField(
  controller: controller,
  focusNode: focusNode,
  document: document,
  isReadOnly: false,
)
```

### MindEditBlock

Individual block component for text editing.

```dart
class MindEditBlock extends StatefulWidget
```

**Properties:**
- `ParagraphDesc texts`: Block content
- `Controller controller`: Main controller
- `bool readOnly`: Read-only mode

## Document Management

### DocumentManager

Manages all documents in the application.

```dart
class DocumentManager
```

**Key Methods:**

#### Document Operations
```dart
Document? getCurrentDoc()
String? getCurrentDocId()
List<DocDataModel> getAllDocuments()
void openDocument(String docId)
void closeDocument()
String newDocument()
void deleteDocument(String docId)
bool createDocument(String title, String content)
```

#### Version Management
```dart
void genNewVersionTree({String? parent})
void mergeVersionTree()
String getLatestVersion()
List<VersionDataModel> getCurrentValidVersionTree()
```

#### Synchronization
```dart
List<SendVersions> assembleRequireVersions(List<String> requiredVersions)
void assembleResources(List<UnsignedResource> resources)
```

**Example:**
```dart
final docManager = controller.docManager;
final docId = docManager.newDocument();
docManager.createDocument('My Note', 'Initial content');
docManager.openDocument(docId);
```

### Document

Represents a single document with its content and metadata.

```dart
class Document
```

**Properties:**
- `String id`: Document identifier
- `List<ParagraphDesc> paragraphs`: Document content
- `bool _hasModified`: Modification flag

**Key Methods:**

#### Content Management
```dart
ParagraphDesc getTitle()
List<String> getTitlePath()
bool hasContent()
ParagraphDesc? getParagraph(String id)
void insertNewParagraphAfterId(String id, ParagraphDesc newItem)
void removeParagraph(String id)
void updateTitle(String title)
```

#### State Management
```dart
void setEditingBlockId(String id)
String? getEditingBlockId()
MindEditBlockState? getBlockState(String id)
void clearTextSelection()
void setModified()
bool getModified()
```

#### Persistence
```dart
String genAndSaveObject()
Map<String, String> getRequiredBlocks()
```

**Example:**
```dart
final document = Document.createDocument(db, 'Title', 'Content', parent, timestamp);
document.insertNewParagraphAfterId('title_id', newParagraph);
document.updateTitle('New Title');
```

## Networking & P2P Communication

### NetworkController

Handles all network operations and P2P communication.

```dart
class NetworkController
```

**Key Methods:**

#### Network Management
```dart
void start(Setting settings, String deviceId, UserPrivateInfo userPrivateInfo, String? logPath)
bool isStarted()
NetworkStatus getNetworkStatus()
bool isAlone()
List<NodeInfo> getNetworkDetails()
```

#### Message Sending
```dart
void sendVersionBroadcast(String latestVersion)
void sendNewVersionTree(List<VersionDataModel> versionData, int timestamp)
void sendRequireVersions(List<String> versions)
void sendVersions(List<SendVersions> versions)
```

#### Lifecycle
```dart
Completer<bool>? gracefulTerminate()
```

**Example:**
```dart
final network = NetworkController(isolate, receivePort);
network.start(settings, deviceId, userInfo, logPath);
network.sendVersionBroadcast('latest_version_hash');
```

### P2P Network Functions

#### startVillage()
```dart
Future<Village> startVillage({
  required String localPort,
  required String serverList,
  required String deviceId,
  required UserPublicInfo userInfo,
  required Function(VillagerNode) connectedCallback,
  required VillageMessageHandler messageHandler,
  bool useMulticast = false,
})
```

**Parameters:**
- `localPort`: Local port for P2P communication
- `serverList`: Comma-separated list of server addresses
- `deviceId`: Unique device identifier
- `userInfo`: User public information
- `connectedCallback`: Callback for node connections
- `messageHandler`: Message handling function
- `useMulticast`: Whether to use multicast discovery

**Example:**
```dart
final village = await startVillage(
  localPort: '8080',
  serverList: '192.168.1.100:8080,192.168.1.101:8080',
  deviceId: 'device123',
  userInfo: userInfo,
  connectedCallback: (node) => print('Connected to ${node.peer}'),
  messageHandler: handleMessage,
  useMulticast: false,
);
```

## Plugin System

### PluginManager

Manages all plugins in the application.

```dart
class PluginManager
```

**Key Methods:**

#### Plugin Registration
```dart
void registerEditorPlugin(PluginProxyImpl proxy, EditorPluginRegisterInformation pluginInfo)
void registerGlobalPlugin(PluginProxyImpl proxy, GlobalPluginRegisterInformation pluginInfo)
```

#### UI Building
```dart
List<Widget> buildButtons({
  required AppearanceSetting appearance,
  required Controller controller,
})
Widget buildGlobalButtons({required Controller controller})
```

#### Content Operations
```dart
String getSelectedOrFocusedContent()
String? appendTextToNextBlock(String blockId, String text)
void addExtra(PluginProxyImpl proxy, String blockId, String content)
void clearExtra(PluginProxyImpl proxy, String blockId)
```

#### Dialog Management
```dart
void showDialog(String title, Widget subChild)
void closeDialog()
void showToast(String message)
```

#### Document Operations
```dart
UserNotes? getUserNotes()
List<DocumentMeta> getAllDocumentList()
String getDocumentContent(String documentId)
void appendToDocument(String documentId, String content)
bool createNote(String title, String content)
void openDocument(String documentId)
```

**Example:**
```dart
final pluginManager = controller.pluginManager;
final content = pluginManager.getSelectedOrFocusedContent();
pluginManager.appendTextToNextBlock('block123', 'New content');
pluginManager.showDialog('Plugin Dialog', MyWidget());
```

### PluginProxy

Interface for plugins to interact with the application.

```dart
abstract class PluginProxy
```

**Key Methods:**
```dart
void registerEditorPlugin(EditorPluginRegisterInformation registerInformation)
void registerGlobalPlugin(GlobalPluginRegisterInformation registerInformation)
void showDialog(String title, Widget child)
void closeDialog()
void showToast(String message)
String getSelectedOrFocusedContent()
String? getEditingBlockId()
void sendTextToClipboard(String text)
UserNotes? getUserNotes()
List<DocumentMeta> getAllDocumentList()
String getDocumentContent(String documentId)
bool createNote(String title, String content)
String? appendTextToNextBlock(String blockId, String text)
void appendToDocument(String documentId, String content)
void addExtra(String blockId, String content)
void clearExtra(String blockId)
String? getSettingValue(String key)
Platform getPlatform()
bool openDocument(String documentId)
```

### PluginInstance

Base class for all plugins.

```dart
abstract class PluginInstance
```

**Methods:**
```dart
void initPlugin(PluginProxy proxy)
void start()
```

**Example Plugin Implementation:**
```dart
class MyPlugin extends PluginInstance {
  @override
  void initPlugin(PluginProxy proxy) {
    proxy.registerEditorPlugin(EditorPluginRegisterInformation(
      pluginName: 'MyPlugin',
      toolbarInformation: ToolbarInformation(
        buttonIcon: Icons.star,
        action: () => proxy.showDialog('My Plugin', MyWidget()),
        tip: 'My Plugin Tooltip',
      ),
      settingsInformation: [
        PluginSetting(
          settingKey: 'my_setting',
          settingName: 'My Setting',
          settingComment: 'Description of my setting',
          settingDefaultValue: 'default_value',
          type: PluginSettingType.string,
        ),
      ],
    ));
  }

  @override
  void start() {
    // Plugin startup logic
  }
}
```

## UI Components & Pages

### MeshApp

Main application widget.

```dart
class MeshApp extends StatefulWidget
```

**Properties:**
- `Key? key`: Widget key

**Example:**
```dart
runApp(const MeshApp());
```

### DocumentNavigator

Navigation component for document management.

```dart
class DocumentNavigator extends StatefulWidget
```

**Properties:**
- `bool smallView`: Whether to use small view mode

### DocumentView

Main document viewing component.

```dart
class DocumentView extends StatefulWidget
```

**Properties:**
- `bool smallView`: Whether to use small view mode

### WelcomeView

Welcome screen for new users.

```dart
class WelcomeView extends StatefulWidget
```

### LargeScreenView

Layout for large screens with multiple panels.

```dart
class LargeScreenView extends StatefulWidget
```

### Setting Pages

#### SettingPageLargeScreen
```dart
class SettingPageLargeScreen extends StatefulWidget
```

#### SettingPageSmallScreen
```dart
class SettingPageSmallScreen extends StatefulWidget
```

### User Management Pages

#### SignInPage
```dart
class SignInPage extends StatefulWidget
```

#### InputPasswordPage
```dart
class InputPasswordPage extends StatefulWidget
```

#### ChangeUserInfoDialog
```dart
class ChangeUserInfoDialog extends StatefulWidget
```

## Utility Functions

### Util

General utility functions.

```dart
class Util
```

**Static Methods:**
```dart
static int getTimeStamp()
static int getRandom(int max)
static void runInPostFrame(Function() callback)
```

**Example:**
```dart
final timestamp = Util.getTimeStamp();
final randomNum = Util.getRandom(100);
Util.runInPostFrame(() => print('Post frame callback'));
```

### ClipboardUtil

Clipboard operations.

```dart
class ClipboardUtil
```

**Static Methods:**
```dart
static Future<void> writeToClipboard(String content)
```

**Example:**
```dart
await ClipboardUtil.writeToClipboard('Text to copy');
```

## Settings Management

### Setting

Manages application settings.

```dart
class Setting
```

**Properties:**
- `String settingFileName`: Settings file path
- Various UI configuration properties (colors, sizes, etc.)

**Key Methods:**

#### Settings Management
```dart
void load()
List<SettingData> getSettings()
List<SettingGroup> getSettingsByGroup()
String? getSetting(String key)
bool saveSettings(List<SettingData> settings)
void addAdditionalSettings(List<SettingData> settings)
```

**Example:**
```dart
final setting = Setting('settings.conf');
setting.load();
final serverList = setting.getSetting('server_list');
setting.saveSettings([newSetting]);
```

### SettingData

Represents a single setting.

```dart
class SettingData
```

**Properties:**
- `String name`: Setting key
- `String? displayName`: Human-readable name
- `String? value`: Setting value
- `String? comment`: Description
- `SettingType type`: Data type
- `String defaultValue`: Default value
- `String settingGroupName`: Group name
- `bool needDisplay`: Whether to show in UI

### SettingType

Enum for setting data types.

```dart
enum SettingType {
  string,
  number,
  bool,
}
```

## Examples

### Basic Application Setup

```dart
import 'package:mesh_note/init.dart';
import 'package:mesh_note/page/mesh_app.dart';

void main() async {
  await appInit();
  runApp(const MeshApp());
}
```

### Creating and Editing Documents

```dart
final controller = Controller();
await controller.initAll();

// Create new document
controller.newDocument();

// Get current document
final document = controller.document;
if (document != null) {
  // Update title
  document.updateTitle('My New Title');
  
  // Insert new paragraph
  final newPara = ParagraphDesc.fromRawString('New content');
  document.insertNewParagraphAfterId('title_id', newPara);
}
```

### Plugin Development

```dart
class MyCustomPlugin extends PluginInstance {
  @override
  void initPlugin(PluginProxy proxy) {
    // Register editor plugin
    proxy.registerEditorPlugin(EditorPluginRegisterInformation(
      pluginName: 'CustomPlugin',
      toolbarInformation: ToolbarInformation(
        buttonIcon: Icons.extension,
        action: () => _handlePluginAction(proxy),
        tip: 'Custom Plugin Action',
      ),
      settingsInformation: [
        PluginSetting(
          settingKey: 'api_key',
          settingName: 'API Key',
          settingComment: 'Enter your API key',
          settingDefaultValue: '',
          type: PluginSettingType.string,
        ),
      ],
    ));
  }

  void _handlePluginAction(PluginProxy proxy) {
    final content = proxy.getSelectedOrFocusedContent();
    if (content.isNotEmpty) {
      // Process content
      final processed = processContent(content);
      proxy.appendTextToNextBlock(proxy.getEditingBlockId()!, processed);
    }
  }

  String processContent(String content) {
    // Custom processing logic
    return content.toUpperCase();
  }

  @override
  void start() {
    print('Custom plugin started');
  }
}
```

### Network Configuration

```dart
final controller = Controller();
await controller.initAll();

// Configure network settings
final setting = controller.setting;
setting.saveSettings([
  SettingData(
    name: 'server_list',
    value: '192.168.1.100:8080,192.168.1.101:8080',
  ),
  SettingData(
    name: 'local_port',
    value: '8080',
  ),
]);

// Start network
if (controller.tryStartingNetwork()) {
  print('Network started successfully');
}
```

### Document Synchronization

```dart
final controller = Controller();
final docManager = controller.docManager;

// Generate new version tree
docManager.genNewVersionTree();

// Send version broadcast
controller.sendVersionBroadcast();

// Save and send version tree
if (controller.tryToSaveAndSendVersionTree()) {
  print('Version tree saved and sent');
}
```

This documentation covers all the major public APIs, functions, and components in the MeshNotes application. Each section includes detailed method signatures, parameters, return values, and practical examples for implementation.