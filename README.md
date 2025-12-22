# Unraid File Manager API and UI

A RESTful API service for managing files and directories on Unraid servers. Built with FastAPI and designed to run in Docker, this API provides a clean interface for Flutter mobile apps (or any HTTP client) to remotely browse, upload, download, and manage files on your Unraid server.

**Web UI Available!** A modern, production-ready web interface is included with advanced features like dual-pane mode, hardlink management, and more.

## Table of Contents

- [Installation](#installation)
- [UI Screenshots](#ui-screenshots)
- [Features](#features)
  - [Core API](#core-api)
  - [Web UI Features](#web-ui-features)
- [Quick Start](#quick-start)
- [API Endpoints](#api-endpoints)
  - [File Browsing & Information](#file-browsing--information)
  - [Download & Upload](#download--upload)
  - [Directory Operations](#directory-operations)
  - [File Operations](#file-operations)
  - [Hardlink Operations](#hardlink-operations)
  - [Background Operations](#background-operations)
  - [Utilities](#utilities)
- [Configuration](#configuration)
  - [Environment Variables](#environment-variables)
  - [API Key Authentication](#api-key-authentication)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)
- [API Documentation](#api-documentation)
- [License](#license)
- [Support](#support)

## Installation

**How to install:**

Copy the xml file templates on your Unraid templates directory: `/boot/config/plugins/dockerMan/templates-user`

Both containers have to be in the same docker network.

<img width="1015" height="872" alt="image" src="https://github.com/user-attachments/assets/748ed7f1-c0f4-47c4-8b24-4a87a04a6bff" />
<img width="1078" height="866" alt="image" src="https://github.com/user-attachments/assets/cedb7b18-96b6-4f55-93cd-a6bf33a795cc" />

## UI Screenshots

<img width="1311" height="955" alt="image" src="https://github.com/user-attachments/assets/6002d34f-090e-4717-ba4d-d6943603577d" />
<img width="822" height="504" alt="image" src="https://github.com/user-attachments/assets/c3c798b7-1e92-4582-818e-957f5774b89e" />
<img width="1310" height="947" alt="image" src="https://github.com/user-attachments/assets/b36fa35b-b590-411f-97d4-d2e13b7d7ccb" />

## Features

### Core API

#### File Operations
- **File Browsing**: Navigate directories with sorting (name, size, date, type)
- **File Upload**: Upload files with progress tracking and size limits
- **File Download**: Download files with HTTP Range support for streaming
- **Move/Copy/Rename/Delete**: Full file management with background operation support
- **Search**: Search files with wildcard support (`*.pdf`, `document*`)

#### Directory Operations
- **Create Directories**: With optional parent creation
- **Delete Directories**: With recursive option
- **Calculate Size**: Get actual size of directories including subdirectories

#### Advanced Features
- **Background Operations**: Large copy/move operations run in background with progress tracking
- **Hardlink Support**:
  - Detect hardlink count for files (`hardlink_count` in file info)
  - Compare two files (hash, size, inode)
  - Replace file with hardlink to save disk space
  - Create hardlinks to existing files
- **Video Streaming**: HTTP Range Request support for smooth video playback
- **ETag Caching**: Efficient directory listing with HTTP caching
- **Disk Usage**: Get disk space information for any path

#### Security & Configuration
- **API Key Authentication**: Optional but recommended for production
- **CORS Enabled**: Ready for web and mobile apps
- **PUID/PGID Support**: linuxserver.io compatible user/group configuration
- **Path Traversal Protection**: Built-in security against directory traversal attacks
- **Configurable Permissions**: Enable/disable deletion, recursive operations

### Web UI Features

#### Interface Modes
- **Single Pane Mode**: Traditional file browser with breadcrumb navigation
- **Dual Pane Mode**: Midnight Commander-style two-panel interface
  - Drag & drop files between panels
  - Keyboard shortcuts (F5-F9, Tab, etc.)
  - Synchronized navigation option
  - Directory comparison mode (highlight unique/different files)
  - Resizable panels with drag separator
  - Link files between panels (create hardlinks)

#### File Management
- **Multi-Select**: Select multiple files with Ctrl/Shift+Click or long-press on mobile
- **Bulk Operations**: Copy, move, delete multiple files at once
- **Clipboard**: Cut/Copy/Paste files with visual feedback
- **Context Menus**: Right-click for file operations
- **Drag & Drop Upload**: Drop files anywhere to upload with progress
- **Live Search**: Filter files as you type

#### Filtering System
- **Hardlink Filter**: Show all / with hardlinks / without hardlinks
- **File Type Filter**: Show all / files only / folders only
- **Hidden Files Toggle**: Show or hide hidden files
- **Active Filter Badge**: Visual indicator of active filters

#### Preview & Editing
- **Image Preview**: Full-screen image viewer with zoom and navigation
- **Video Preview**: Video player with seeking, fullscreen, and gallery navigation
- **Smart Thumbnails**: Auto-generated at 30% duration to avoid black intros
- **Text Editor**: Built-in code editor with syntax highlighting
- **Scroll Position Restoration**: Return to same position after viewing files

#### User Experience
- **Material Design 3**: Modern, polished interface with smooth animations
- **Theme Support**: Light, Dark, and System-preference modes
- **Responsive Design**: Works on desktop, tablet, and mobile
- **Internationalization**: English and Spanish languages
- **Progress Dialogs**: Visual feedback for all operations
- **What's New Dialog**: Changelog shown on version updates
- **Keyboard Shortcuts**: Full keyboard navigation support

#### Dual Pane Keyboard Shortcuts
| Key | Action |
|-----|--------|
| Tab | Switch active panel |
| F5 | Copy to other panel |
| F6 | Move to other panel |
| F7 | Create new folder |
| F8 | Delete selected |
| F9 | Link files (hardlink) |
| Ctrl+1/2 | Focus left/right panel |
| Ctrl+A | Select all |
| Ctrl+D | Deselect all |
| Ctrl+R | Refresh |
| Ctrl+= | Equalize panel sizes |
| F1 | Show keyboard shortcuts |

## Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Unraid server with accessible file shares
- Proper permissions to access `/mnt/user` directory

### Installation via Docker Compose

1. **Clone this repository:**
   ```bash
   cd /mnt/user/appdata
   git clone https://github.com/jandrop/file_core_api.git
   cd file_core_api
   ```

2. **Create your environment file:**
   ```bash
   cp .env.example .env
   ```

3. **Edit `.env` and configure your settings:**
   ```env
   # User/Group (match your Unraid user - find with: id nobody)
   PUID=99
   PGID=100
   UMASK=002

   # File paths
   BASE_PATH=/mnt/user
   API_KEY=your_secret_key_here
   MAX_UPLOAD_SIZE=10737418240
   ```

4. **Start the services:**

   **With Web UI (Recommended):**
   ```bash
   docker-compose --profile ui up -d
   ```

   **API Only:**
   ```bash
   docker-compose up -d
   ```

5. **Access the interfaces:**
   - **Web UI**: http://your-unraid-ip:8764 (if enabled)
   - **API Docs**: http://your-unraid-ip:8000/docs
   - **ReDoc**: http://your-unraid-ip:8000/redoc

## API Endpoints

### File Browsing & Information

#### List Directory Contents
```http
GET /api/v1/files/browse?path=/mnt/user/documents&sort_by=name&sort_order=asc
```

**Parameters:**
- `path` (required): Directory path to browse
- `show_hidden` (optional): Show hidden files (true/false)
- `sort_by` (optional): Sort by field (name, size, modified, type)
- `sort_order` (optional): Sort order (asc, desc)

**Response:**
```json
{
  "path": "/mnt/user/documents",
  "parent": "/mnt/user",
  "items": [
    {
      "name": "document.pdf",
      "path": "/mnt/user/documents/document.pdf",
      "type": "file",
      "size": 1048576,
      "modified": "2024-01-15T10:30:00",
      "permissions": "rw-r--r--",
      "owner": "nobody",
      "group": "users",
      "is_hidden": false,
      "extension": "pdf",
      "mime_type": "application/pdf",
      "hardlink_count": 1
    }
  ],
  "total_items": 1,
  "total_size": 1048576
}
```

#### Get File/Directory Info
```http
GET /api/v1/files/info?path=/mnt/user/documents/file.txt
```

### Download & Upload

#### Download File
```http
GET /api/v1/files/download?path=/mnt/user/documents/file.txt
```

Supports HTTP Range requests for video streaming.

#### Preview File
```http
GET /api/v1/files/preview?path=/mnt/user/video.mp4
```

Returns file with correct MIME type for browser preview.

#### Upload File
```http
POST /api/v1/files/upload
Content-Type: multipart/form-data
```

**Form Data:**
- `file`: The file to upload
- `destination_path`: Directory where file should be saved

### Directory Operations

#### Create Directory
```http
POST /api/v1/files/directory
Content-Type: application/json

{
  "path": "/mnt/user/documents",
  "name": "new_folder",
  "create_parents": true
}
```

#### Delete Directory
```http
DELETE /api/v1/files/directory?path=/mnt/user/old_folder&recursive=true
```

#### Calculate Directory Size
```http
POST /api/v1/files/calculate-size
Content-Type: application/json

{
  "paths": ["/mnt/user/documents", "/mnt/user/media"]
}
```

### File Operations

#### Delete File
```http
DELETE /api/v1/files/file?path=/mnt/user/documents/old_file.txt
```

#### Move File/Directory
```http
POST /api/v1/files/move?background=true
Content-Type: application/json

{
  "source": "/mnt/user/documents/file.txt",
  "destination": "/mnt/user/backup/file.txt",
  "overwrite": false
}
```

Set `background=true` for large operations to get progress tracking.

#### Copy File/Directory
```http
POST /api/v1/files/copy?background=true
Content-Type: application/json

{
  "source": "/mnt/user/documents/file.txt",
  "destination": "/mnt/user/backup/file.txt",
  "overwrite": false
}
```

#### Rename File/Directory
```http
PUT /api/v1/files/rename
Content-Type: application/json

{
  "path": "/mnt/user/documents/old_name.txt",
  "new_name": "new_name.txt"
}
```

### Hardlink Operations

#### Compare Two Files
```http
POST /api/v1/files/compare
Content-Type: application/json

{
  "file1": "/mnt/user/documents/file1.txt",
  "file2": "/mnt/user/backup/file1.txt",
  "compare_hash": true
}
```

**Response:**
```json
{
  "file1": "/mnt/user/documents/file1.txt",
  "file2": "/mnt/user/backup/file1.txt",
  "are_identical": true,
  "same_size": true,
  "same_hash": true,
  "file1_size": 1024,
  "file2_size": 1024,
  "file1_hash": "a1b2c3d4...",
  "file2_hash": "a1b2c3d4...",
  "already_hardlinked": false
}
```

#### Replace File with Hardlink
```http
POST /api/v1/files/replace-hardlink
Content-Type: application/json

{
  "source": "/mnt/user/documents/original.txt",
  "target": "/mnt/user/backup/copy.txt",
  "force": false
}
```

Replaces the target file with a hardlink to the source, saving disk space.

#### Create Hardlink
```http
POST /api/v1/files/hardlink
Content-Type: application/json

{
  "source": "/mnt/user/documents/original.txt",
  "destination": "/mnt/user/links/link.txt"
}
```

### Background Operations

#### Get Operation Status
```http
GET /api/v1/files/operations/{operation_id}
```

**Response:**
```json
{
  "operation_id": "uuid",
  "operation_type": "copy",
  "status": "in_progress",
  "source": "/mnt/user/source",
  "destination": "/mnt/user/dest",
  "total_items": 100,
  "processed_items": 45,
  "total_bytes": 104857600,
  "processed_bytes": 47185920,
  "progress_percentage": 45.0,
  "current_file": "file45.txt"
}
```

#### List All Operations
```http
GET /api/v1/files/operations
```

#### Cancel Operation
```http
DELETE /api/v1/files/operations/{operation_id}
```

### Utilities

#### Search Files
```http
GET /api/v1/files/search?path=/mnt/user&query=*.pdf&recursive=true
```

**Parameters:**
- `path` (required): Directory to search in
- `query` (required): Search pattern (supports wildcards)
- `recursive` (optional): Search in subdirectories (default: true)
- `case_sensitive` (optional): Case sensitive search (default: false)
- `file_type` (optional): Filter by type (file/directory)

#### Get Disk Usage
```http
GET /api/v1/files/disk-usage?path=/mnt/user
```

#### Get API Configuration
```http
GET /api/v1/files/config
```

Returns base_path and other public configuration.

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PUID` | User ID for file operations | `99` |
| `PGID` | Group ID for file operations | `100` |
| `UMASK` | File creation mask | `022` |
| `DEBUG` | Enable debug mode | `false` |
| `CORS_ORIGINS` | Allowed CORS origins (JSON array) | `["http://localhost",...]` |
| `BASE_PATH` | Base path for file operations | `/mnt/user` |
| `MAX_UPLOAD_SIZE` | Maximum upload size in bytes | `10737418240` (10GB) |
| `SHOW_HIDDEN_FILES` | Show hidden files by default | `false` |
| `ALLOWED_EXTENSIONS` | Allowed extensions (JSON array) | `["*"]` |
| `API_KEY` | API key for authentication | - (disabled) |
| `ENABLE_FILE_DELETION` | Enable file deletion | `true` |
| `ENABLE_RECURSIVE_DELETE` | Enable recursive deletion | `true` |
| `UNRAID_DATA_PATH` | Host path to mount | `/mnt/user` |

**Common PUID/PGID values for Unraid:**
- `PUID=99`, `PGID=100` - nobody:users (recommended)
- `PUID=0`, `PGID=0` - root (full access, less secure)

### API Key Authentication

When `API_KEY` is configured, all endpoints require the `X-API-Key` header:

```bash
curl -H "X-API-Key: your_secret_key_here" \
  http://localhost:8000/api/v1/files/browse?path=/mnt/user
```

Generate a secure key:
```bash
openssl rand -base64 32
```

Configure in Web UI: Menu > API Key > Enter key > Save

## Security Considerations

- **Use API Key Authentication**: Always set an API_KEY in production
- **Configure CORS**: Limit CORS_ORIGINS to trusted applications
- **File Permissions**: Use appropriate PUID/PGID settings
- **Disable Dangerous Operations**: Set `ENABLE_FILE_DELETION=false` if needed
- **Reverse Proxy**: Run behind HTTPS proxy in production
- **Network Isolation**: Use Docker networks to isolate containers
- **Path Traversal Protection**: Built-in protection included

## Troubleshooting

### Permission Issues

Configure PUID/PGID to match your Unraid user:
```bash
id nobody  # uid=99(nobody) gid=100(users)
```

### Upload Fails

1. Check MAX_UPLOAD_SIZE setting
2. Verify destination directory exists
3. Check disk space with `/api/v1/files/disk-usage`

### Video Won't Stream

Ensure the file path is accessible and the MIME type is correctly detected.

### UI Restart Loops / "host not found in upstream" Error

If you see this error in the UI logs:
```
host not found in upstream "file-manager-api" in /etc/nginx/conf.d/default.conf
```

This means the UI container cannot find the API container. **Solutions:**

**Option 1: Verify the --link parameter (Recommended)**
1. The API container MUST be named exactly `FileManagerAPI`
2. The UI container MUST have this in Extra Parameters: `--link FileManagerAPI:file-manager-api`
3. Reinstall the UI container if the link wasn't set correctly

**Option 2: Create a Docker network manually**
```bash
# Create network
docker network create file-manager-network

# Connect both containers
docker network connect file-manager-network FileManagerAPI
docker network connect file-manager-network FileManagerUI

# Add network alias for API
docker network disconnect file-manager-network FileManagerAPI
docker network connect --alias file-manager-api file-manager-network FileManagerAPI

# Restart UI
docker restart FileManagerUI
```

**Option 3: Use Host network mode**
Set both containers to use `host` network mode instead of `bridge`.

**Verification:**
```bash
# Check container names
docker ps --format "table {{.Names}}\t{{.Status}}"

# Test connectivity from UI to API
docker exec FileManagerUI ping -c 2 file-manager-api

# Check UI logs
docker logs FileManagerUI --tail 50
```

## API Documentation

Interactive documentation available at:
- **Swagger UI**: http://your-unraid-ip:8000/docs
- **ReDoc**: http://your-unraid-ip:8000/redoc

## License

MIT License

## Support

For issues and questions, please open an issue on GitHub.
