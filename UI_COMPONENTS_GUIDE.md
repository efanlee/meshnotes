# UI Components Guide

## Overview

MeshNotes provides a comprehensive set of UI components for building the user interface. These components handle everything from the main application layout to individual editing blocks and user interactions.

## Main Application Components

### MeshApp

The root application widget that sets up the main theme and routing.

```dart
class MeshApp extends StatefulWidget {
  const MeshApp({Key? key}) : super(key: key);
}
```

**Features:**
- Material Design 3 theme
- Lifecycle management for mobile and desktop
- Window close handling for desktop platforms

**Example:**
```dart
runApp(const MeshApp());
```

### StackPageView

Main application layout using a stack-based approach.

```dart
class StackPageView extends StatefulWidget {
  const StackPageView({Key? key}) : super(key: key);
}
```

**Features:**
- Stack-based navigation
- Multiple overlay support
- Responsive layout

## Document Navigation Components

### DocumentNavigator

Navigation component for document management.

```dart
class DocumentNavigator extends StatefulWidget {
  final bool smallView;
  
  const DocumentNavigator({
    Key? key,
    required this.smallView,
  }) : super(key: key);
}
```

**Properties:**
- `bool smallView`: Whether to use compact view mode

**Features:**
- Document list display
- Document creation and deletion
- Search and filtering
- Responsive design

**Example:**
```dart
DocumentNavigator(
  smallView: MediaQuery.of(context).size.width < 600,
)
```

### DocumentView

Main document viewing and editing component.

```dart
class DocumentView extends StatefulWidget {
  final bool smallView;
  
  const DocumentView({
    Key? key,
    required this.smallView,
  }) : super(key: key);
}
```

**Properties:**
- `bool smallView`: Whether to use compact view mode

**Features:**
- Document editing interface
- Toolbar integration
- Plugin support
- Responsive layout

## Mind Editor Components

### MindEditor

The main editor widget for document editing.

```dart
class MindEditor extends StatefulWidget {
  const MindEditor({Key? key}) : super(key: key);
}
```

**Features:**
- Document editing interface
- Toolbar integration
- Screenshot functionality
- Plugin support

**Methods:**
```dart
void open(Document doc)
void close()
```

**Example:**
```dart
MindEditor(
  key: Key('main_editor'),
)
```

### MindEditField

Core editing field that handles text input and document rendering.

```dart
class MindEditField extends StatefulWidget {
  final Controller controller;
  final bool isReadOnly;
  final FocusNode focusNode;
  final Document document;
  
  const MindEditField({
    Key? key,
    required this.controller,
    this.isReadOnly = false,
    required this.focusNode,
    required this.document,
  }) : super(key: key);
}
```

**Properties:**
- `Controller controller`: Main application controller
- `bool isReadOnly`: Whether the field is read-only
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

**Features:**
- Text input handling
- Keyboard management
- Scroll handling
- Block rendering
- Gesture recognition

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
class MindEditBlock extends StatefulWidget {
  final ParagraphDesc texts;
  final Controller controller;
  final bool readOnly;
  
  const MindEditBlock({
    Key? key,
    required this.texts,
    required this.controller,
    this.readOnly = false,
  }) : super(key: key);
}
```

**Properties:**
- `ParagraphDesc texts`: Block content
- `Controller controller`: Main controller
- `bool readOnly`: Read-only mode

**Features:**
- Text rendering and editing
- Formatting support
- Selection handling
- Block state management

## Layout Components

### LargeScreenView

Layout for large screens with multiple panels.

```dart
class LargeScreenView extends StatefulWidget {
  const LargeScreenView({Key? key}) : super(key: key);
}
```

**Features:**
- Multi-panel layout
- Resizable panels
- Document navigator integration
- Document view integration

### ResizableView

Resizable panel component.

```dart
class ResizableView extends StatefulWidget {
  final Widget child;
  final double initialWidth;
  final double minWidth;
  final double maxWidth;
  
  const ResizableView({
    Key? key,
    required this.child,
    required this.initialWidth,
    required this.minWidth,
    required this.maxWidth,
  }) : super(key: key);
}
```

**Properties:**
- `Widget child`: Child widget to display
- `double initialWidth`: Initial width
- `double minWidth`: Minimum width
- `double maxWidth`: Maximum width

**Features:**
- Drag-to-resize functionality
- Width constraints
- Smooth resizing animation

**Example:**
```dart
ResizableView(
  initialWidth: 300,
  minWidth: 200,
  maxWidth: 500,
  child: DocumentNavigator(smallView: false),
)
```

## Welcome and Onboarding

### WelcomeView

Welcome screen for new users.

```dart
class WelcomeView extends StatefulWidget {
  const WelcomeView({Key? key}) : super(key: key);
}
```

**Features:**
- Welcome message
- Getting started guide
- User registration/login
- Feature highlights

### SplashPage

Application splash screen with initialization.

```dart
class SplashPage extends StatefulWidget {
  final Future<void> Function() onInit;
  
  const SplashPage({
    Key? key,
    required this.onInit,
  }) : super(key: key);
}
```

**Properties:**
- `Future<void> Function() onInit`: Initialization function

**Features:**
- Loading animation
- Initialization progress
- Error handling
- Automatic transition

**Example:**
```dart
SplashPage(
  onInit: appInit,
)
```

## Settings Components

### SettingPageLargeScreen

Settings page for large screens.

```dart
class SettingPageLargeScreen extends StatefulWidget {
  const SettingPageLargeScreen({Key? key}) : super(key: key);
}
```

**Features:**
- Multi-column layout
- Category organization
- Real-time preview
- Advanced options

### SettingPageSmallScreen

Settings page for small screens.

```dart
class SettingPageSmallScreen extends StatefulWidget {
  const SettingPageSmallScreen({Key? key}) : super(key: key);
}
```

**Features:**
- Single-column layout
- Tabbed interface
- Mobile-optimized controls
- Simplified options

## User Management Components

### SignInPage

User sign-in interface.

```dart
class SignInPage extends StatefulWidget {
  const SignInPage({Key? key}) : super(key: key);
}
```

**Features:**
- User authentication
- Password input
- Error handling
- Registration link

### InputPasswordPage

Password input interface.

```dart
class InputPasswordPage extends StatefulWidget {
  const InputPasswordPage({Key? key}) : super(key: key);
}
```

**Features:**
- Secure password input
- Password validation
- Biometric authentication support
- Forgot password option

### ChangeUserInfoDialog

Dialog for changing user information.

```dart
class ChangeUserInfoDialog extends StatefulWidget {
  const ChangeUserInfoDialog({Key? key}) : super(key: key);
}
```

**Features:**
- User profile editing
- Form validation
- Save/cancel actions
- Real-time preview

### UserInfoSettingMenu

User information settings menu.

```dart
class UserInfoSettingMenu extends StatefulWidget {
  const UserInfoSettingMenu({Key? key}) : super(key: key);
}
```

**Features:**
- User profile display
- Settings access
- Account management
- Logout functionality

## Toolbar Components

### MindEditorToolBar

Main editor toolbar.

```dart
class MindEditorToolBar extends StatefulWidget {
  final Controller controller;
  final BuildContext context;
  
  const MindEditorToolBar({
    Key? key,
    required this.controller,
    required this.context,
  }) : super(key: key);
}
```

**Features:**
- Formatting tools
- Plugin buttons
- Block operations
- Responsive design

**Static Methods:**
```dart
static Widget basic({
  required Key key,
  required Controller controller,
  required BuildContext context,
})
```

### ToolbarButton

Individual toolbar button component.

```dart
class ToolbarButton extends StatefulWidget {
  final IconData icon;
  final AppearanceSetting appearance;
  final String tip;
  final Controller controller;
  final VoidCallback onPressed;
  
  const ToolbarButton({
    Key? key,
    required this.icon,
    required this.appearance,
    required this.tip,
    required this.controller,
    required this.onPressed,
  }) : super(key: key);
}
```

**Properties:**
- `IconData icon`: Button icon
- `AppearanceSetting appearance`: Visual appearance settings
- `String tip`: Tooltip text
- `Controller controller`: Main controller
- `VoidCallback onPressed`: Press callback

**Features:**
- Hover effects
- Tooltip display
- Icon customization
- State management

## Title Bar Components

### TitleBar

Application title bar component.

```dart
class TitleBar extends StatefulWidget {
  const TitleBar({Key? key}) : super(key: key);
}
```

**Features:**
- Document title display
- Navigation breadcrumbs
- Window controls (desktop)
- Status indicators

## Menu Components

### Menu

Application menu component.

```dart
class Menu extends StatefulWidget {
  const Menu({Key? key}) : super(key: key);
}
```

**Features:**
- File operations
- Edit commands
- View options
- Help and settings

## Version Management Components

### VersionPage

Version history and management page.

```dart
class VersionPage extends StatefulWidget {
  const VersionPage({Key? key}) : super(key: key);
}
```

**Features:**
- Version tree visualization
- Version comparison
- Conflict resolution
- Synchronization status

## Utility Components

### WidgetTemplates

Collection of reusable widget templates.

```dart
class WidgetTemplates {
  // Static utility methods for common widgets
}
```

**Features:**
- Common button styles
- Input field templates
- Dialog templates
- Layout helpers

## Responsive Design

### Environment Detection

```dart
class Environment {
  bool isMobile()
  bool isDesktop()
  bool isAndroid()
  bool isIos()
  bool isWindows()
  bool isMac()
  bool isLinux()
  bool isSmallView(BuildContext context)
}
```

**Example:**
```dart
final environment = Environment();
if (environment.isMobile()) {
  // Use mobile-optimized layout
  return MobileLayout();
} else {
  // Use desktop layout
  return DesktopLayout();
}
```

### Responsive Layouts

```dart
// Responsive document view
Widget buildDocumentView(BuildContext context) {
  final isSmallView = environment.isSmallView(context);
  
  if (isSmallView) {
    return DocumentView(smallView: true);
  } else {
    return LargeScreenView();
  }
}
```

## Theme and Styling

### Theme Configuration

```dart
// Material Design 3 theme
ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.fromSeed(
    surfaceTint: Colors.transparent,
    seedColor: Colors.grey.shade600,
    brightness: Brightness.light,
    primary: Colors.grey.shade700,
    secondary: Colors.grey.shade500,
    surface: Colors.white,
    onPrimary: Colors.white,
    onSecondary: Colors.white,
  ),
)
```

### Appearance Settings

```dart
class AppearanceSetting {
  final double iconSize;
  final Color backgroundColor;
  final Color hoverColor;
  final Color activeColor;
  
  const AppearanceSetting({
    required this.iconSize,
    required this.backgroundColor,
    required this.hoverColor,
    required this.activeColor,
  });
}
```

## Best Practices

### 1. Responsive Design

```dart
Widget buildResponsiveWidget(BuildContext context) {
  final screenWidth = MediaQuery.of(context).size.width;
  
  if (screenWidth < 600) {
    return MobileLayout();
  } else if (screenWidth < 1200) {
    return TabletLayout();
  } else {
    return DesktopLayout();
  }
}
```

### 2. State Management

```dart
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  void initState() {
    super.initState();
    // Initialize state
  }
  
  @override
  void dispose() {
    // Clean up resources
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Container(
      // Widget content
    );
  }
}
```

### 3. Error Handling

```dart
Widget buildWithErrorHandling() {
  return Builder(
    builder: (context) {
      try {
        return MyWidget();
      } catch (e) {
        return ErrorWidget('Error: $e');
      }
    },
  );
}
```

### 4. Performance Optimization

```dart
// Use const constructors where possible
const MyWidget({
  Key? key,
  required this.title,
}) : super(key: key);

// Use ListView.builder for large lists
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text(items[index]),
    );
  },
)
```

### 5. Accessibility

```dart
Widget buildAccessibleWidget() {
  return Semantics(
    label: 'Document editor',
    hint: 'Double tap to edit document',
    child: MindEditField(
      controller: controller,
      focusNode: focusNode,
      document: document,
    ),
  );
}
```

## Examples

### Complete Document Editor

```dart
class DocumentEditor extends StatefulWidget {
  final String documentId;
  
  const DocumentEditor({
    Key? key,
    required this.documentId,
  }) : super(key: key);
  
  @override
  _DocumentEditorState createState() => _DocumentEditorState();
}

class _DocumentEditorState extends State<DocumentEditor> {
  final controller = Controller();
  final focusNode = FocusNode();
  Document? document;
  
  @override
  void initState() {
    super.initState();
    _loadDocument();
  }
  
  void _loadDocument() {
    document = controller.docManager.getDocument(widget.documentId);
    if (document != null) {
      controller.openDocument(widget.documentId);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    if (document == null) {
      return Center(child: CircularProgressIndicator());
    }
    
    return Column(
      children: [
        MindEditorToolBar.basic(
          key: Key('toolbar'),
          controller: controller,
          context: context,
        ),
        Expanded(
          child: MindEditField(
            controller: controller,
            focusNode: focusNode,
            document: document!,
          ),
        ),
      ],
    );
  }
  
  @override
  void dispose() {
    focusNode.dispose();
    super.dispose();
  }
}
```

### Responsive Layout

```dart
class ResponsiveLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final environment = Environment();
    final isSmallView = environment.isSmallView(context);
    
    if (isSmallView) {
      return _buildMobileLayout();
    } else {
      return _buildDesktopLayout();
    }
  }
  
  Widget _buildMobileLayout() {
    return Column(
      children: [
        Expanded(
          child: DocumentNavigator(smallView: true),
        ),
        Expanded(
          child: DocumentView(smallView: true),
        ),
      ],
    );
  }
  
  Widget _buildDesktopLayout() {
    return Row(
      children: [
        ResizableView(
          initialWidth: 300,
          minWidth: 200,
          maxWidth: 500,
          child: DocumentNavigator(smallView: false),
        ),
        Expanded(
          child: DocumentView(smallView: false),
        ),
      ],
    );
  }
}
```

This guide provides comprehensive information about UI components in MeshNotes. Follow these patterns and best practices to create responsive, accessible, and performant user interfaces.