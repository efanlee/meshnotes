# Networking & P2P Communication Guide

## Overview

MeshNotes uses a peer-to-peer (P2P) networking system for real-time collaborative editing and document synchronization. The networking layer is built on top of the libp2p protocol and provides automatic discovery, version synchronization, and conflict resolution.

## Architecture

### Core Components

1. **NetworkController**: Main network interface running in the main isolate
2. **Network Isolate**: Background isolate handling P2P communication
3. **Village Protocol**: P2P overlay network implementation
4. **Bonjour/mDNS**: Service discovery for local network peers
5. **Version Chain**: Document versioning and synchronization system

### Network Flow

```
Main App ←→ NetworkController ←→ Network Isolate ←→ Village Protocol ←→ P2P Network
                ↓
            Bonjour Discovery
```

## NetworkController API

### Initialization

```dart
class NetworkController {
  void start(Setting settings, String deviceId, UserPrivateInfo userPrivateInfo, String? logPath)
  bool isStarted()
  NetworkStatus getNetworkStatus()
  bool isAlone()
  List<NodeInfo> getNetworkDetails()
}
```

#### Starting the Network

```dart
final network = NetworkController(isolate, receivePort);
network.start(settings, deviceId, userPrivateInfo, logPath);
```

**Parameters:**
- `settings`: Application settings containing network configuration
- `deviceId`: Unique device identifier
- `userPrivateInfo`: User's private information for authentication
- `logPath`: Path for network logs (optional)

### Message Sending

#### Version Broadcasting
```dart
void sendVersionBroadcast(String latestVersion)
```

Sends the latest version hash to all connected peers.

```dart
network.sendVersionBroadcast('abc123def456');
```

#### Version Tree Synchronization
```dart
void sendNewVersionTree(List<VersionDataModel> versionData, int timestamp)
```

Sends a complete version tree to peers.

```dart
final versionData = docManager.getCurrentValidVersionTree();
final timestamp = Util.getTimeStamp();
network.sendNewVersionTree(versionData, timestamp);
```

#### Resource Requests
```dart
void sendRequireVersions(List<String> versions)
void sendRequireVersionTree(String latestVersion)
```

Request specific versions or version trees from peers.

```dart
// Request specific versions
network.sendRequireVersions(['version1', 'version2', 'version3']);

// Request entire version tree
network.sendRequireVersionTree('latest_version_hash');
```

#### Resource Provision
```dart
void sendVersions(List<SendVersions> versions)
```

Send requested versions to peers.

```dart
final versions = docManager.assembleRequireVersions(requiredVersions);
network.sendVersions(versions);
```

### Network Lifecycle

#### Graceful Termination
```dart
Completer<bool>? gracefulTerminate()
```

Safely shutdown the network connection.

```dart
final completer = network.gracefulTerminate();
if (completer != null) {
  await completer.future;
  print('Network terminated gracefully');
}
```

## P2P Network Functions

### Village Protocol

#### Starting a Village
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
- `localPort`: Local port for P2P communication (e.g., "8080")
- `serverList`: Comma-separated list of known server addresses
- `deviceId`: Unique device identifier
- `userInfo`: User's public information
- `connectedCallback`: Callback when new nodes connect
- `messageHandler`: Handler for incoming messages
- `useMulticast`: Whether to use multicast for discovery

**Example:**
```dart
final village = await startVillage(
  localPort: '8080',
  serverList: '192.168.1.100:8080,192.168.1.101:8080',
  deviceId: 'device123',
  userInfo: userInfo,
  connectedCallback: (node) {
    print('Connected to peer: ${node.peer}');
  },
  messageHandler: (message) {
    print('Received message: ${message.type}');
  },
  useMulticast: false,
);
```

### Service Discovery

#### Bonjour/mDNS Discovery

The network automatically handles service discovery using Bonjour (on macOS/iOS) or mDNS (on other platforms).

**Service Configuration:**
- Service Name: 'VillageProtocol'
- Service Type: '_village-v0._udp'
- Port: Configured local port
- Attributes: Device ID

**Automatic Discovery:**
```dart
// Discovery is handled automatically
// When a peer is found:
void _onPeerDiscovered(String host, int port, String deviceId) {
  print('Discovered peer: $host:$port (device: $deviceId)');
}
```

## Message Types

### Command Types

```dart
enum Command {
  startVillage,
  terminate,
  newNodeDiscovered,
  sendBroadcast,
  sendVersionTree,
  sendRequireVersions,
  sendVersions,
  villageStarted,
  terminateOk,
  networkStatus,
  nodeStatus,
  receiveBroadcast,
  receiveProvide,
  receiveQuery,
}
```

### Message Structure

```dart
class Message {
  final Command cmd;
  final dynamic parameter;
}
```

### Broadcast Messages

```dart
class BroadcastMessages {
  final Map<String, String> messages;
}
```

**Example:**
```dart
final broadcast = BroadcastMessages(
  messages: {
    'latest_version': 'abc123def456',
    'user_name': 'John Doe',
  }
);
```

### Version Tree Messages

```dart
class SendVersionTreeParameter {
  final VersionChain versionChain;
  final int timestamp;
}
```

### Resource Request Messages

```dart
class SendRequireVersionsParameter {
  final List<String> versions;
}
```

### Resource Provision Messages

```dart
class SendVersionsParameter {
  final List<SendVersions> versions;
}
```

## Network Status

### NetworkStatus Enum

```dart
enum NetworkStatus {
  unknown,
  starting,
  running,
  stopped,
}
```

### Node Information

```dart
class NodeInfo {
  final String peer;
  final NodeStatus status;
  final String? lastSeen;
}
```

### Node Status

```dart
enum NodeStatus {
  inContact,
  disconnected,
  connecting,
}
```

## Configuration

### Network Settings

Configure network behavior through application settings:

```dart
// Server list (comma-separated)
setting.saveSettings([
  SettingData(
    name: 'server_list',
    value: '192.168.1.100:8080,192.168.1.101:8080',
  ),
]);

// Local port
setting.saveSettings([
  SettingData(
    name: 'local_port',
    value: '8080',
  ),
]);
```

### Setting Keys

```dart
class Constants {
  static const String settingKeyServerList = 'server_list';
  static const String settingKeyLocalPort = 'local_port';
  static const String settingDefaultServerList = '';
  static const String settingDefaultLocalPort = '8080';
}
```

## Synchronization Process

### Version Management

#### Document Versioning
```dart
// Generate new version tree
docManager.genNewVersionTree();

// Get current version
final latestVersion = docManager.getLatestVersion();

// Send version broadcast
network.sendVersionBroadcast(latestVersion);
```

#### Version Tree Structure
```dart
class VersionContent {
  final List<VersionContentItem> table;
  final int timestamp;
  final List<String> parentsHash;
}
```

#### Version Content Item
```dart
class VersionContentItem {
  final String docId;
  final String docHash;
  final int updatedAt;
}
```

### Conflict Resolution

#### Merge Process
```dart
// Merge version trees
docManager.mergeVersionTree();

// Handle conflicts
final conflicts = mergeManager.detectConflicts(localVersion, remoteVersion);
for (final conflict in conflicts) {
  // Resolve conflict (automatic or manual)
  final resolution = resolveConflict(conflict);
  mergeManager.applyResolution(conflict, resolution);
}
```

#### Three-Way Merge
The system uses three-way merge for conflict resolution:
1. **Base Version**: Common ancestor
2. **Local Version**: Current local changes
3. **Remote Version**: Incoming remote changes

```dart
final mergedVersion = mergeManager.threeWayMerge(
  baseVersion: commonAncestor,
  localVersion: localChanges,
  remoteVersion: remoteChanges,
);
```

## Error Handling

### Network Errors

```dart
void _handleNetworkError(dynamic error) {
  MyLogger.error('Network error: $error');
  
  // Handle specific error types
  if (error is NetworkTimeoutException) {
    // Retry connection
    _retryConnection();
  } else if (error is PeerDisconnectedException) {
    // Handle peer disconnection
    _handlePeerDisconnection();
  }
}
```

### Connection Retry

```dart
void _retryConnection() {
  Timer.periodic(Duration(seconds: 5), (timer) {
    if (network.isStarted()) {
      timer.cancel();
      return;
    }
    
    // Attempt to restart network
    if (controller.tryStartingNetwork()) {
      timer.cancel();
    }
  });
}
```

## Performance Optimization

### Batch Operations

```dart
// Batch multiple version requests
final allRequiredVersions = <String>[];
for (final version in missingVersions) {
  allRequiredVersions.add(version);
}
network.sendRequireVersions(allRequiredVersions);
```

### Compression

```dart
// Compress large version trees before sending
final compressedData = compressVersionTree(versionData);
network.sendNewVersionTree(compressedData, timestamp);
```

### Caching

```dart
// Cache frequently accessed versions
class VersionCache {
  final Map<String, VersionContent> _cache = {};
  
  VersionContent? getVersion(String hash) {
    return _cache[hash];
  }
  
  void cacheVersion(String hash, VersionContent version) {
    _cache[hash] = version;
  }
}
```

## Security

### Authentication

```dart
// User authentication
final userInfo = UserPrivateInfo(
  userName: 'username',
  publicKey: publicKey,
  privateKey: privateKey,
);

network.start(settings, deviceId, userInfo, logPath);
```

### Encryption

All network communication is encrypted using the user's public/private key pair.

### Message Signing

```dart
// Sign messages before sending
final signedMessage = signMessage(message, privateKey);
network.sendMessage(signedMessage);
```

## Debugging

### Network Logging

```dart
// Enable network logging
MyLogger.init(name: 'network', debug: true);

// Log network events
MyLogger.info('Network started on port $port');
MyLogger.info('Connected to peer: $peerId');
MyLogger.info('Received version: $versionHash');
```

### Network Status Monitoring

```dart
void _monitorNetworkStatus() {
  Timer.periodic(Duration(seconds: 10), (timer) {
    final status = network.getNetworkStatus();
    final nodes = network.getNetworkDetails();
    
    MyLogger.info('Network status: $status');
    MyLogger.info('Connected nodes: ${nodes.length}');
    
    for (final node in nodes) {
      MyLogger.info('Node: ${node.peer}, Status: ${node.status}');
    }
  });
}
```

### Message Tracing

```dart
void _traceMessages() {
  // Log all outgoing messages
  void logOutgoingMessage(Message message) {
    MyLogger.info('Sending: ${message.cmd} - ${message.parameter}');
  }
  
  // Log all incoming messages
  void logIncomingMessage(Message message) {
    MyLogger.info('Received: ${message.cmd} - ${message.parameter}');
  }
}
```

## Best Practices

### 1. Connection Management

```dart
// Always check network status before operations
if (!network.isStarted()) {
  MyLogger.warn('Network not started, cannot send message');
  return;
}

// Handle network failures gracefully
try {
  network.sendVersionBroadcast(version);
} catch (e) {
  MyLogger.error('Failed to send version broadcast: $e');
  // Implement retry logic
}
```

### 2. Resource Management

```dart
// Clean up resources on shutdown
@override
void dispose() {
  final completer = network.gracefulTerminate();
  if (completer != null) {
    completer.future.then((_) {
      MyLogger.info('Network resources cleaned up');
    });
  }
  super.dispose();
}
```

### 3. Error Recovery

```dart
// Implement automatic error recovery
void _handleNetworkFailure() {
  // Wait before retrying
  Timer(Duration(seconds: 5), () {
    if (controller.tryStartingNetwork()) {
      MyLogger.info('Network restarted successfully');
    } else {
      MyLogger.error('Failed to restart network');
    }
  });
}
```

### 4. Performance Monitoring

```dart
// Monitor network performance
void _monitorPerformance() {
  final stopwatch = Stopwatch()..start();
  
  network.sendVersionBroadcast(version);
  
  stopwatch.stop();
  MyLogger.info('Version broadcast took ${stopwatch.elapsedMilliseconds}ms');
}
```

This guide provides comprehensive information about the networking and P2P communication system in MeshNotes. Follow these patterns and best practices to ensure reliable, secure, and efficient network operations.