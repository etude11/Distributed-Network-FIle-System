# Distributed-Network-FIle-System

This is an updated version of the course project submission for "Operating Systems & Networks". This project was completed in a team of 4.

## How to Run

```bash
make                                  # Compiles the code

./ns.out                              # Naming Server Executable
./ss.out <ip_address_of_ns>           # Storage Server Executable
./client.out <ip_address_of_ns>       # Client Executable
```

## File Structure

```
.
├── common
│   ├── common_headers.h
│   ├── helpers.c
│   ├── network_operations.c
│   └── tree_operations.c
│
├── client
│   ├── headers.h
│   ├── main.c
│   └── serveroperations.c
│
├── naming_server
│   ├── cache.c
│   ├── headers.h
│   ├── init.c
│   ├── main.c
│   ├── ns_cl_communication.c
│   ├── ns_ss_communication.c
│   └── storageServerMetaData.c
│
├── storage_server
│   ├── headers.h
│   ├── init.c
│   ├── main.c
│   ├── ss_cl_communication.c
│   └── ss_ns_communication.c
│
└── Makefile
```

## Project Components

### Common

The common directory contains shared code and headers used by all system components.

#### common_headers.h

Contains global headers, macros, constants, and data structures used throughout the project:

- **Global Headers**: Standard C libraries and system headers
- **Macros**: Error checking and color formatting
- **Error Codes**: Standardized codes for various operations
- **Constants**: Network ports, buffer sizes, and configuration values
- **Data Structures**: File information, directory nodes, path handling, and server details

#### helpers.c

Implements general utility functions:
- `split`: Divides strings into tokens based on delimiters

#### network_operations.c

Handles network communication:
- `connectXtoY`: Establishes TCP connections between components
- `sendFileChunks`/`receiveFileChunks`: File transfer functions
- `send_data_in_packets`/`receive_data_in_packets`: Reliable packet transmission

#### tree_operations.c

Manages the directory tree structure:
- `initDirectoryTree`: Creates the root node
- `addDirectoryPath`: Adds files/directories to the tree
- `searchTree`: Locates paths in the tree
- `deleteTree`: Removes nodes and frees memory
- `getAllAccessiblePaths`: Lists available paths
- `callCopyFolderRecursive`: Handles folder copying operations

### Client

The client directory contains code for the client application that interacts with servers.

#### headers.h

Declares client-specific functions and variables:
- External variables for client connections
- Function declarations for user interface and server operations

#### main.c

Implements the client's main functionality:
- `printClientInterface`: Displays user operation menu
- `readFile`: Gets file path input
- `getOperation`: Maps user choices to operation strings
- `checkOption`: Validates and processes user selections
- `main`: Entry point that handles connections and user input

#### serveroperations.c

Implements file operation handlers:
- `readOperation`: Reads files from storage servers
- `writeOperation`: Sends user data to storage servers
- `retrieveOperation`: Gets file metadata
- `streamOperation`: Streams audio files

### Naming Server

The naming_server directory contains code for the central coordinator.

#### headers.h

Declares naming server functions and structures:
- Cache-related structures
- Function prototypes for initialization and operations
- Global variables for directory tree management

#### init.c

Sets up the naming server:
- `initNamingServer`: Initializes directory tree and cache

#### main.c

Core functionality:
- `addToLog`: Records events in log files
- `main`: Sets up threads to handle client and storage server connections

#### cache.c

Implements an LRU cache for path lookups:
- `initCache`: Sets up the cache structure
- `getFromCache`/`putInCache`: Retrieves and stores cache entries
- `deleteFromCache`: Removes entries when files are deleted
- `getIndexOfStorageServer`: Finds server indices for paths

#### ns_cl_communication.c

Manages client communications:
- `hearClientforNS`: Accepts client connections
- `NSReceiveClientOperations`: Processes client requests
- `handleClientOperation`: Handles specific operations like READ, WRITE, CREATE, DELETE

#### ns_ss_communication.c

Handles storage server communications:
- `hearSSforNS`: Processes storage server connections and metadata

#### storageServerMetaData.c

Manages storage server records:
- Functions to add, find, and track connected storage servers

### Storage Server

The storage_server directory contains code for file storage components.

#### headers.h

Declares storage server functions:
- Initialization and communication functions
- File operation handlers

#### main.c

Core functionality:
- `main`: Sets up connections and creates threads for operations

#### init.c

Initializes the storage server:
- `initStorageServer`: Sets up paths and connects to naming server

#### ss_ns_communication.c

Manages naming server communications:
- `SSReceiveNamingServerOperations`: Processes commands from the naming server
- Operation handlers for create, delete, and copy actions

#### ss_cl_communication.c

Handles client communications:
- `hearClientforSS`: Accepts client connections
- `SSReceiveClientOperations`: Processes client requests
- Operation handlers for read, write, retrieve, and stream

## System Design

- **Distributed Architecture**: Network-based system with multiple storage servers
- **Concurrency**: Multithreading for handling multiple connections
- **Synchronization**: Mutex locks for shared resource access
- **Caching**: LRU cache for optimized path lookups
- **Error Handling**: Standardized error codes and reporting
- **File Management**: Hierarchical structure with proper locking

## Assumptions

1. File/folder names cannot contain spaces
2. Storage server initialization must complete before starting another server
3. Naming server sends a struct with storage server details as acknowledgment
4. The `libmpv-dev` package must be installed for audio streaming
5. Clients can only access paths provided by the naming server
6. New storage servers must report index as -1; reconnecting servers use assigned indices
7. Maximum 10 clients per storage server (configurable via SERVER_LOAD)
8. Maximum 100 storage servers per naming server (configurable via MAX_STORAGE_SERVERS)
9. Maximum 10 queued connections per port (configurable via MAX_CONNECTIONS)
10. Write operations require specifying synchronous/asynchronous mode each time
11. Asynchronous writes flush to persistent storage every 15 seconds (configurable)
12. Audio streaming plays in terminal without launching the mpv player