# Plugin Development Guide

## Overview

The MeshNotes plugin system allows developers to extend the application's functionality by creating custom plugins. Plugins can add toolbar buttons, settings, and interact with documents and the user interface.

## Plugin Architecture

### Core Components

1. **PluginInstance**: Base class that all plugins must extend
2. **PluginProxy**: Interface for plugins to interact with the application
3. **PluginManager**: Manages plugin registration and lifecycle
4. **PluginProxyImpl**: Implementation of the PluginProxy interface

### Plugin Types

#### Editor Plugins
Editor plugins add functionality to the document editing interface:
- Toolbar buttons
- Settings
- Block content change listeners

#### Global Plugins
Global plugins provide application-wide functionality:
- Global toolbar buttons
- Settings
- Application-level features

## Creating a Plugin

### Step 1: Define Your Plugin Class

```dart
import 'package:mesh_note/plugin/plugin_api.dart';
import 'package:flutter/material.dart';

class MyPlugin extends PluginInstance {
  @override
  void initPlugin(PluginProxy proxy) {
    // Plugin initialization logic
  }

  @override
  void start() {
    // Plugin startup logic
  }
}
```

### Step 2: Register Your Plugin

#### Editor Plugin Registration

```dart
@override
void initPlugin(PluginProxy proxy) {
  proxy.registerEditorPlugin(EditorPluginRegisterInformation(
    pluginName: 'MyPlugin',
    toolbarInformation: ToolbarInformation(
      buttonIcon: Icons.star,
      action: () => _handleButtonClick(proxy),
      tip: 'My Plugin Action',
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
    onBlockChanged: (data) => _handleBlockChanged(data),
  ));
}
```

#### Global Plugin Registration

```dart
@override
void initPlugin(PluginProxy proxy) {
  proxy.registerGlobalPlugin(GlobalPluginRegisterInformation(
    pluginName: 'MyGlobalPlugin',
    toolbarInformation: GlobalToolbarInformation(
      buttonIcon: Icons.settings,
      buildWidget: ({required void Function() onClose}) => MyGlobalWidget(onClose: onClose),
      tip: 'Global Plugin',
      isAvailable: () => true,
    ),
    settingsInformation: [
      PluginSetting(
        settingKey: 'global_setting',
        settingName: 'Global Setting',
        settingComment: 'Global setting description',
        settingDefaultValue: 'true',
        type: PluginSettingType.bool,
      ),
    ],
  ));
}
```

### Step 3: Add Plugin to PluginManager

In `lib/plugin/plugin_manager.dart`, add your plugin to the `_plugins` list:

```dart
List<PluginInstance> _plugins = [
  PluginAI(),
  MyPlugin(), // Add your plugin here
];
```

## PluginProxy API Reference

### UI Operations

#### Dialog Management
```dart
void showDialog(String title, Widget child)
void closeDialog()
void showToast(String message)
```

#### Example:
```dart
void _showMyDialog(PluginProxy proxy) {
  proxy.showDialog('My Dialog', Container(
    child: Text('Hello from my plugin!'),
  ));
}

void _showNotification(PluginProxy proxy) {
  proxy.showToast('Operation completed successfully!');
}
```

### Content Operations

#### Getting Content
```dart
String getSelectedOrFocusedContent()
String? getEditingBlockId()
UserNotes? getUserNotes()
List<DocumentMeta> getAllDocumentList()
String getDocumentContent(String documentId)
```

#### Modifying Content
```dart
String? appendTextToNextBlock(String blockId, String text)
void appendToDocument(String documentId, String content)
void addExtra(String blockId, String content)
void clearExtra(String blockId)
void sendTextToClipboard(String text)
```

#### Example:
```dart
void _processSelectedText(PluginProxy proxy) {
  final content = proxy.getSelectedOrFocusedContent();
  if (content.isNotEmpty) {
    // Process the content
    final processed = content.toUpperCase();
    
    // Append to next block
    final blockId = proxy.getEditingBlockId();
    if (blockId != null) {
      proxy.appendTextToNextBlock(blockId, processed);
    }
  }
}
```

### Document Operations

#### Creating Documents
```dart
bool createNote(String title, String content)
bool openDocument(String documentId)
```

#### Example:
```dart
void _createNewNote(PluginProxy proxy) {
  final success = proxy.createNote('Plugin Generated Note', 'This note was created by my plugin!');
  if (success) {
    proxy.showToast('Note created successfully!');
  }
}
```

### Settings Management

```dart
String? getSettingValue(String key)
```

#### Example:
```dart
void _usePluginSetting(PluginProxy proxy) {
  final apiKey = proxy.getSettingValue('api_key');
  if (apiKey != null && apiKey.isNotEmpty) {
    // Use the API key
    _makeApiCall(apiKey);
  } else {
    proxy.showToast('Please configure your API key in settings');
  }
}
```

### Platform Detection

```dart
Platform getPlatform()
```

The Platform interface provides:
```dart
bool isWindows()
bool isMacOS()
bool isAndroid()
bool isIOS()
bool isMobile()
bool isLinux()
```

#### Example:
```dart
void _handlePlatformSpecificLogic(PluginProxy proxy) {
  final platform = proxy.getPlatform();
  
  if (platform.isMobile()) {
    // Mobile-specific logic
    proxy.showToast('Mobile platform detected');
  } else if (platform.isWindows()) {
    // Windows-specific logic
    proxy.showToast('Windows platform detected');
  }
}
```

## Plugin Settings

### Setting Types

```dart
enum PluginSettingType {
  string,
  number,
  bool,
}
```

### Setting Definition

```dart
PluginSetting(
  settingKey: 'unique_key',
  settingName: 'Display Name',
  settingComment: 'Description shown to user',
  settingDefaultValue: 'default_value',
  type: PluginSettingType.string,
)
```

### Accessing Settings

```dart
final value = proxy.getSettingValue('unique_key');
```

## Event Handling

### Block Content Changes

```dart
void onBlockChanged(BlockChangedEventData data) {
  print('Block ${data.blockId} changed: ${data.content}');
}
```

### Registration

```dart
EditorPluginRegisterInformation(
  // ... other properties
  onBlockChanged: onBlockChanged,
)
```

## Complete Plugin Example

Here's a complete example of a text processing plugin:

```dart
import 'package:mesh_note/plugin/plugin_api.dart';
import 'package:flutter/material.dart';

class TextProcessorPlugin extends PluginInstance {
  @override
  void initPlugin(PluginProxy proxy) {
    proxy.registerEditorPlugin(EditorPluginRegisterInformation(
      pluginName: 'TextProcessor',
      toolbarInformation: ToolbarInformation(
        buttonIcon: Icons.text_fields,
        action: () => _processText(proxy),
        tip: 'Process Selected Text',
      ),
      settingsInformation: [
        PluginSetting(
          settingKey: 'uppercase_enabled',
          settingName: 'Enable Uppercase',
          settingComment: 'Convert text to uppercase',
          settingDefaultValue: 'true',
          type: PluginSettingType.bool,
        ),
        PluginSetting(
          settingKey: 'remove_spaces',
          settingName: 'Remove Extra Spaces',
          settingComment: 'Remove multiple consecutive spaces',
          settingDefaultValue: 'false',
          type: PluginSettingType.bool,
        ),
      ],
      onBlockChanged: (data) => _onBlockChanged(data),
    ));
  }

  void _processText(PluginProxy proxy) {
    final content = proxy.getSelectedOrFocusedContent();
    if (content.isEmpty) {
      proxy.showToast('No text selected');
      return;
    }

    String processed = content;
    
    // Apply settings
    final uppercaseEnabled = proxy.getSettingValue('uppercase_enabled') == 'true';
    final removeSpaces = proxy.getSettingValue('remove_spaces') == 'true';
    
    if (uppercaseEnabled) {
      processed = processed.toUpperCase();
    }
    
    if (removeSpaces) {
      processed = processed.replaceAll(RegExp(r'\s+'), ' ');
    }
    
    // Insert processed text
    final blockId = proxy.getEditingBlockId();
    if (blockId != null) {
      proxy.appendTextToNextBlock(blockId, processed);
      proxy.showToast('Text processed successfully');
    }
  }

  void _onBlockChanged(BlockChangedEventData data) {
    // Handle block content changes
    print('Block ${data.blockId} content changed');
  }

  @override
  void start() {
    print('TextProcessor plugin started');
  }
}
```

## Best Practices

### 1. Error Handling

Always handle potential errors gracefully:

```dart
void _safeOperation(PluginProxy proxy) {
  try {
    final content = proxy.getSelectedOrFocusedContent();
    // Process content
  } catch (e) {
    proxy.showToast('Error: ${e.toString()}');
  }
}
```

### 2. User Feedback

Provide clear feedback to users:

```dart
void _operationWithFeedback(PluginProxy proxy) {
  proxy.showToast('Processing...');
  
  // Perform operation
  
  proxy.showToast('Operation completed!');
}
```

### 3. Settings Validation

Validate settings before use:

```dart
void _validateSettings(PluginProxy proxy) {
  final apiKey = proxy.getSettingValue('api_key');
  if (apiKey == null || apiKey.isEmpty) {
    proxy.showToast('Please configure API key in settings');
    return;
  }
  
  // Use validated setting
}
```

### 4. Platform Considerations

Consider platform differences:

```dart
void _platformAwareOperation(PluginProxy proxy) {
  final platform = proxy.getPlatform();
  
  if (platform.isMobile()) {
    // Use mobile-optimized UI
    proxy.showDialog('Mobile Dialog', MobileWidget());
  } else {
    // Use desktop-optimized UI
    proxy.showDialog('Desktop Dialog', DesktopWidget());
  }
}
```

### 5. Resource Management

Clean up resources when needed:

```dart
class MyPlugin extends PluginInstance {
  StreamSubscription? _subscription;
  
  @override
  void start() {
    _subscription = someStream.listen((data) {
      // Handle stream data
    });
  }
  
  void dispose() {
    _subscription?.cancel();
  }
}
```

## Debugging Plugins

### 1. Use Logging

```dart
import 'package:my_log/my_log.dart';

void _debugOperation() {
  MyLogger.info('Plugin operation started');
  // ... operation logic
  MyLogger.info('Plugin operation completed');
}
```

### 2. Test Settings

```dart
void _testSettings(PluginProxy proxy) {
  final allSettings = [
    'my_setting1',
    'my_setting2',
    'my_setting3',
  ];
  
  for (final setting in allSettings) {
    final value = proxy.getSettingValue(setting);
    MyLogger.info('Setting $setting: $value');
  }
}
```

### 3. Validate Content

```dart
void _validateContent(PluginProxy proxy) {
  final content = proxy.getSelectedOrFocusedContent();
  MyLogger.info('Content length: ${content.length}');
  MyLogger.info('Content preview: ${content.substring(0, content.length > 100 ? 100 : content.length)}');
}
```

## Distribution

### 1. Add to Plugin List

Add your plugin to the `_plugins` list in `plugin_manager.dart`:

```dart
List<PluginInstance> _plugins = [
  PluginAI(),
  MyPlugin(),
  // Add your plugin here
];
```

### 2. Import Statement

Ensure your plugin is imported:

```dart
import 'package:mesh_note/plugin/my_plugin.dart';
```

### 3. Testing

Test your plugin thoroughly:
- Test all toolbar buttons
- Verify settings work correctly
- Test error conditions
- Test on different platforms

This guide provides comprehensive information for developing plugins for MeshNotes. Follow these patterns and best practices to create robust, user-friendly plugins that integrate seamlessly with the application.