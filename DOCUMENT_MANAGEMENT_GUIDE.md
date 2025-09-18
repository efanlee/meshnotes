# Document Management Guide

## Overview

The document management system in MeshNotes handles the creation, editing, storage, and synchronization of documents. It provides a hierarchical structure for organizing content and supports real-time collaborative editing with conflict resolution.

## Architecture

### Core Components

1. **DocumentManager**: Central manager for all documents
2. **Document**: Individual document representation
3. **ParagraphDesc**: Document paragraph/block representation
4. **DbHelper**: Database operations
5. **VersionManager**: Version control and synchronization

### Document Structure

```
Document
├── Title (ParagraphDesc)
├── Paragraph 1 (ParagraphDesc)
├── Paragraph 2 (ParagraphDesc)
└── ...
```

## DocumentManager API

### Initialization

```dart
class DocumentManager {
  DocumentManager({required DbHelper db});
}
```

### Document Operations

#### Creating Documents

```dart
String newDocument()
bool createDocument(String title, String content)
```

**Examples:**
```dart
// Create empty document
final docId = docManager.newDocument();

// Create document with content
final success = docManager.createDocument(
  'My Document',
  'Initial content\nSecond line\nThird line'
);
```

#### Opening and Closing Documents

```dart
void openDocument(String docId)
void closeDocument()
```

**Example:**
```dart
// Open a document
docManager.openDocument('doc123');

// Close current document
docManager.closeDocument();
```

#### Document Retrieval

```dart
Document? getCurrentDoc()
String? getCurrentDocId()
Document? getDocument(String docId, {bool getDirectly = false})
List<DocDataModel> getAllDocuments()
```

**Examples:**
```dart
// Get current document
final currentDoc = docManager.getCurrentDoc();

// Get specific document
final doc = docManager.getDocument('doc123');

// Get all documents
final allDocs = docManager.getAllDocuments();
for (final docData in allDocs) {
  print('Document: ${docData.title} (${docData.docId})');
}
```

#### Document Deletion

```dart
void deleteDocument(String docId)
void deleteCurrentDocument()
```

**Example:**
```dart
// Delete specific document
docManager.deleteDocument('doc123');

// Delete current document
docManager.deleteCurrentDocument();
```

### Version Management

#### Version Tree Operations

```dart
void genNewVersionTree({String? parent})
void tryToGenNewVersionTreeAndOverrideCurrent()
void mergeVersionTree()
String getLatestVersion()
List<VersionDataModel> getCurrentValidVersionTree()
List<VersionDataModel> getCurrentRawVersionTree()
```

**Examples:**
```dart
// Generate new version tree
docManager.genNewVersionTree();

// Get latest version
final latestVersion = docManager.getLatestVersion();

// Get version tree
final versionTree = docManager.getCurrentValidVersionTree();
print('Version tree has ${versionTree.length} nodes');
```

#### Version Synchronization

```dart
List<SendVersions> assembleRequireVersions(List<String> requiredVersions)
void assembleResources(List<UnsignedResource> resources)
```

**Example:**
```dart
// Assemble versions for sending
final requiredVersions = ['version1', 'version2'];
final sendVersions = docManager.assembleRequireVersions(requiredVersions);

// Assemble resources from network
final resources = <UnsignedResource>[];
docManager.assembleResources(resources);
```

### Status Management

#### Modification Tracking

```dart
bool hasModified()
void setModified()
```

**Example:**
```dart
if (docManager.hasModified()) {
  print('Document has unsaved changes');
  docManager.setModified();
}
```

#### Busy State

```dart
bool isBusy()
bool isGenerating()
bool isSyncing()
```

**Example:**
```dart
if (docManager.isBusy()) {
  print('Document manager is busy, cannot perform operation');
  return;
}
```

## Document API

### Document Creation

#### Factory Constructors

```dart
factory Document.loadByNode(DbHelper db, DocDataModel docNode, DocumentManager parent)
factory Document.createDocument(DbHelper db, String title, String content, DocumentManager parent, int now)
```

**Examples:**
```dart
// Load existing document
final doc = Document.loadByNode(db, docNode, parent);

// Create new document
final doc = Document.createDocument(
  db,
  'New Document',
  'Initial content',
  parent,
  Util.getTimeStamp()
);
```

### Document Properties

```dart
String id                    // Document identifier
List<ParagraphDesc> paragraphs  // Document content
bool _hasModified           // Modification flag
int _lastUpdate            // Last update timestamp
```

### Content Management

#### Title Operations

```dart
ParagraphDesc getTitle()
List<String> getTitlePath()
void updateTitle(String title)
```

**Examples:**
```dart
// Get document title
final title = document.getTitle();
print('Document title: ${title.getPlainText()}');

// Update title
document.updateTitle('New Title');

// Get title path (for navigation)
final path = document.getTitlePath();
print('Path: ${path.join(' > ')}');
```

#### Paragraph Operations

```dart
ParagraphDesc? getParagraph(String id)
void insertNewParagraphAfterId(String id, ParagraphDesc newItem)
void insertNewParagraphsAfterId(String id, List<ParagraphDesc> newItems)
void removeParagraph(String id)
void insertTextsAtTheEnd(List<String> texts)
```

**Examples:**
```dart
// Get specific paragraph
final para = document.getParagraph('para123');

// Insert new paragraph
final newPara = ParagraphDesc.fromRawString('New content');
document.insertNewParagraphAfterId('title_id', newPara);

// Insert multiple paragraphs
final newParas = [
  ParagraphDesc.fromRawString('First new paragraph'),
  ParagraphDesc.fromRawString('Second new paragraph'),
];
document.insertNewParagraphsAfterId('para123', newParas);

// Insert text at end
document.insertTextsAtTheEnd(['Line 1', 'Line 2', 'Line 3']);

// Remove paragraph
document.removeParagraph('para123');
```

#### Content Queries

```dart
bool hasContent()
```

**Example:**
```dart
if (document.hasContent()) {
  print('Document has content');
} else {
  print('Document is empty');
}
```

### State Management

#### Editing State

```dart
void setEditingBlockId(String id)
String? getEditingBlockId()
MindEditBlockState? getEditingBlockState()
ParagraphDesc? getEditingBlockDesc()
void clearEditingBlock()
```

**Examples:**
```dart
// Set editing block
document.setEditingBlockId('block123');

// Get editing block
final editingId = document.getEditingBlockId();
final editingState = document.getEditingBlockState();
final editingDesc = document.getEditingBlockDesc();

// Clear editing state
document.clearEditingBlock();
```

#### Block State Management

```dart
void setBlockStateToTreeNode(String id, MindEditBlockState state)
MindEditBlockState? getBlockState(String id)
```

**Example:**
```dart
// Set block state
document.setBlockStateToTreeNode('block123', blockState);

// Get block state
final state = document.getBlockState('block123');
```

#### Selection Management

```dart
void clearTextSelection()
```

**Example:**
```dart
// Clear all text selections
document.clearTextSelection();
```

### Navigation

#### Block Navigation

```dart
String? getPreviousBlockId(String id)
String? getNextBlockId(String id)
```

**Example:**
```dart
// Navigate between blocks
final currentId = 'block123';
final prevId = document.getPreviousBlockId(currentId);
final nextId = document.getNextBlockId(currentId);

if (prevId != null) {
  print('Previous block: $prevId');
}
if (nextId != null) {
  print('Next block: $nextId');
}
```

### Persistence

#### Object Generation

```dart
String genAndSaveObject()
Map<String, String> getRequiredBlocks()
```

**Examples:**
```dart
// Generate and save document object
final docHash = document.genAndSaveObject();
print('Document saved with hash: $docHash');

// Get required blocks for synchronization
final blocks = document.getRequiredBlocks();
for (final entry in blocks.entries) {
  print('Block ${entry.key}: ${entry.value}');
}
```

#### Modification Tracking

```dart
void setModified()
bool getModified()
void clearModified()
int getLastUpdateTime()
```

**Example:**
```dart
// Check and manage modification state
if (document.getModified()) {
  print('Document has been modified');
  document.clearModified();
}

final lastUpdate = document.getLastUpdateTime();
print('Last updated: ${DateTime.fromMillisecondsSinceEpoch(lastUpdate)}');
```

### Document Lifecycle

#### Document Closing

```dart
void closeDocument()
```

**Example:**
```dart
// Close document and clean up resources
document.closeDocument();
```

#### Document Updates

```dart
void updateBlocks(Document newDoc)
```

**Example:**
```dart
// Update document with new content (from sync)
document.updateBlocks(newDocument);
```

## ParagraphDesc API

### Paragraph Creation

#### Factory Constructors

```dart
factory ParagraphDesc.fromTitle(String title)
factory ParagraphDesc.fromRawString(String content)
factory ParagraphDesc.buildFromJson({
  required String id,
  required String jsonStr,
  required int time,
  String? extra,
})
```

**Examples:**
```dart
// Create title paragraph
final title = ParagraphDesc.fromTitle('My Document Title');

// Create content paragraph
final content = ParagraphDesc.fromRawString('This is paragraph content');

// Create from JSON
final para = ParagraphDesc.buildFromJson(
  id: 'para123',
  jsonStr: jsonString,
  time: Util.getTimeStamp(),
  extra: 'extra_data',
);
```

### Paragraph Properties

```dart
String getBlockId()           // Unique block identifier
String getPlainText()         // Plain text content
bool isTitle()               // Whether this is a title block
bool hasContent()            // Whether block has content
```

### Paragraph Operations

#### Text Operations

```dart
void setPlainText(String text)
List<TextDesc> getTexts()
void setTexts(List<TextDesc> texts)
```

**Examples:**
```dart
// Set plain text
para.setPlainText('New paragraph content');

// Get text descriptions
final texts = para.getTexts();
for (final text in texts) {
  print('Text: ${text.text}');
}

// Set text descriptions
para.setTexts([TextDesc()..text = 'New text']);
```

#### Block Type and Formatting

```dart
String getBlockType()
void setBlockType(String type)
String getBlockListing()
void setBlockListing(String listing)
int getBlockLevel()
void setBlockLevel(int level)
```

**Examples:**
```dart
// Set block type
para.setBlockType('heading1');

// Set listing type
para.setBlockListing('bullet');

// Set indentation level
para.setBlockLevel(2);
```

#### Database Operations

```dart
void flushDb()
void storeObject(int timestamp)
void drop()
```

**Examples:**
```dart
// Flush to database
para.flushDb();

// Store object with timestamp
para.storeObject(Util.getTimeStamp());

// Drop paragraph
para.drop();
```

#### Document Relationships

```dart
void setDocument(Document doc)
void setPrevious(ParagraphDesc? prev)
void setNext(ParagraphDesc? next)
ParagraphDesc? getPrevious()
ParagraphDesc? getNext()
```

**Example:**
```dart
// Set document relationship
para.setDocument(document);

// Set paragraph chain
para.setPrevious(prevPara);
para.setNext(nextPara);

// Navigate chain
final prev = para.getPrevious();
final next = para.getNext();
```

## Database Operations

### DbHelper API

```dart
class DbHelper {
  Future<void> init()
  
  // Document operations
  String newDocument(int timestamp)
  void insertOrUpdateDoc(String docId, String docHash, int timestamp)
  void updateDocContent(String docId, String content, int timestamp)
  void deleteDocument(String docId)
  DocDataModel? getDoc(String docId)
  List<DocDataModel> getAllDocuments()
  
  // Block operations
  void storeDocBlock(String docId, String blockId, String blockData, int timestamp)
  Map<String, BlockDataModel> getBlockMapOfDoc(String docId)
  BlockDataModel? getRawBlockById(String docId, String blockId)
  List<(String, String)> getAllBlocks()
  
  // Version operations
  void storeVersion(String versionHash, String parents, int timestamp, String createdFrom, String status)
  void storeObject(String hash, String data, int timestamp, String createdFrom, String status)
  VersionDataModel? getVersionData(String versionHash)
  ObjectDataModel? getObject(String hash)
  List<VersionDataModel> getAllVersions()
  bool hasObject(String hash)
  
  // Flag operations
  void setFlag(String name, String value)
  String? getFlag(String name)
  void removeFlag(String name)
}
```

## Version Control

### Version Content Structure

```dart
class VersionContent {
  final List<VersionContentItem> table;
  final int timestamp;
  final List<String> parentsHash;
  
  String getHash()
}
```

### Version Content Item

```dart
class VersionContentItem {
  final String docId;
  final String docHash;
  final int updatedAt;
}
```

### Version Data Model

```dart
class VersionDataModel {
  final String versionHash;
  final String parents;
  final int createdAt;
  final String createdFrom;
  final String status;
  final String syncStatus;
  
  List<String> getParentsList()
}
```

## Best Practices

### 1. Document Lifecycle Management

```dart
// Always check if document exists before operations
final doc = docManager.getDocument(docId);
if (doc == null) {
  print('Document not found: $docId');
  return;
}

// Handle document state properly
if (doc.getModified()) {
  // Save changes before closing
  doc.genAndSaveObject();
}
```

### 2. Error Handling

```dart
try {
  final docId = docManager.newDocument();
  docManager.openDocument(docId);
} catch (e) {
  print('Error creating document: $e');
  // Handle error appropriately
}
```

### 3. Performance Optimization

```dart
// Batch operations when possible
final newParas = <ParagraphDesc>[];
for (final text in texts) {
  newParas.add(ParagraphDesc.fromRawString(text));
}
document.insertNewParagraphsAfterId('parent_id', newParas);
```

### 4. Memory Management

```dart
// Clean up resources when done
@override
void dispose() {
  document?.closeDocument();
  super.dispose();
}
```

### 5. Synchronization

```dart
// Check if document manager is busy before operations
if (docManager.isBusy()) {
  print('Document manager is busy, retrying later');
  Timer(Duration(seconds: 1), () {
    // Retry operation
  });
  return;
}

// Generate version tree after modifications
if (docManager.hasModified()) {
  docManager.genNewVersionTree();
}
```

## Examples

### Complete Document Workflow

```dart
class DocumentWorkflow {
  final DocumentManager docManager;
  
  DocumentWorkflow(this.docManager);
  
  Future<String> createAndEditDocument() async {
    // Create new document
    final docId = docManager.newDocument();
    docManager.openDocument(docId);
    
    final doc = docManager.getCurrentDoc();
    if (doc == null) return '';
    
    // Update title
    doc.updateTitle('My New Document');
    
    // Add content
    final titleId = doc.getTitle().getBlockId();
    final content1 = ParagraphDesc.fromRawString('First paragraph');
    final content2 = ParagraphDesc.fromRawString('Second paragraph');
    
    doc.insertNewParagraphAfterId(titleId, content1);
    doc.insertNewParagraphAfterId(content1.getBlockId(), content2);
    
    // Save changes
    doc.genAndSaveObject();
    
    return docId;
  }
  
  void editDocument(String docId) {
    final doc = docManager.getDocument(docId);
    if (doc == null) return;
    
    // Get all paragraphs
    for (final para in doc.paragraphs) {
      if (!para.isTitle()) {
        // Edit non-title paragraphs
        para.setPlainText('Modified: ${para.getPlainText()}');
      }
    }
    
    // Mark as modified
    doc.setModified();
  }
  
  void deleteDocument(String docId) {
    docManager.deleteDocument(docId);
  }
}
```

### Document Synchronization

```dart
class DocumentSync {
  final DocumentManager docManager;
  final NetworkController network;
  
  DocumentSync(this.docManager, this.network);
  
  void syncDocument() {
    if (!docManager.hasModified()) return;
    
    // Generate new version tree
    docManager.genNewVersionTree();
    
    // Get latest version
    final latestVersion = docManager.getLatestVersion();
    
    // Send version broadcast
    network.sendVersionBroadcast(latestVersion);
    
    // Send version tree if needed
    final versionTree = docManager.getCurrentValidVersionTree();
    if (versionTree.isNotEmpty) {
      network.sendNewVersionTree(versionTree, Util.getTimeStamp());
    }
  }
  
  void handleIncomingVersions(List<String> versions) {
    // Request missing versions
    network.sendRequireVersions(versions);
  }
  
  void handleIncomingResources(List<UnsignedResource> resources) {
    // Assemble resources
    docManager.assembleResources(resources);
    
    // Merge version tree
    docManager.mergeVersionTree();
  }
}
```

This guide provides comprehensive information about document management in MeshNotes. Follow these patterns and best practices to ensure efficient and reliable document operations.