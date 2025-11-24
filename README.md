# Unraid File Manager API and UI

**How to install:**
Copy the xml file templates on your Unraid templates directory: /config/plugins/dockerMan/templates-user
Both containers have to be in the same docker network.

A RESTful API service for managing files and directories on Unraid servers. Built with FastAPI and designed to run in Docker, this API provides a clean interface for Flutter mobile apps (or any HTTP client) to remotely browse, upload, download, and manage files on your Unraid server.

**✨ NEW: Web UI Available!** A modern, production-ready web interface is now included for visual file management. See [Web UI Setup](#web-ui) below.

## Features

### Core API
- **📁 File Browsing**: Navigate directories and list files with sorting options
- **📤 File Upload**: Upload files with size limits and validation
- **📥 File Download**: Download files directly through the API
- **📂 Directory Operations**: Create and delete directories (with recursive option)
- **✏️ File Operations**: Move, copy, rename, and delete files
- **🔍 Search**: Search for files and directories with wildcard support
- **💾 Disk Usage**: Get disk space information for any path
- **🔒 Security**: API key authentication and configurable permissions
- **🌐 CORS Enabled**: Ready for web and mobile apps
- **🐳 Docker Ready**: Easy deployment with Docker/Docker Compose

### Web UI (Optional)
- **🎨 Material Design 3**: Modern, polished interface with smooth animations
- **🌓 Dark/Light Mode**: Toggle themes with persistent preferences
- **📱 Responsive Design**: Works seamlessly on desktop, tablet, and mobile
- **⚡ Real-time Updates**: Instant feedback for all file operations
- **🔄 Drag & Drop**: Easy file uploads with progress tracking
- **🔍 Live Search**: Filter files as you type
- **✅ Production Ready**: Professional UI suitable for production use

## Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Unraid server with accessible file shares
- Proper permissions to access `/mnt/user` directory

### Installation

1. **Clone this repository:**
   ```bash
   git clone <repository-url>
   cd file_core_api
   ```

2. **Create your environment file:**
   ```bash
   cp .env.example .env
   ```

3. **Edit `.env` and configure your settings:**
   ```env
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
   - **Web UI**: http://localhost:8764 (if enabled)
   - **API Docs**: http://localhost:8000/docs
   - **ReDoc**: http://localhost:8000/redoc

## API Endpoints

### 📁 File Browsing & Information

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
      "mime_type": "application/pdf"
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

**Response:** Returns detailed `FileInfo` object

### 📥 Download & Upload

#### Download File
```http
GET /api/v1/files/download?path=/mnt/user/documents/file.txt
```

Returns the file as an octet-stream for download.

#### Upload File
```http
POST /api/v1/files/upload
Content-Type: multipart/form-data
```

**Form Data:**
- `file`: The file to upload
- `destination_path`: Directory where file should be saved

**Response:**
```json
{
  "success": true,
  "file_path": "/mnt/user/uploads/document.pdf",
  "file_name": "document.pdf",
  "file_size": 1048576,
  "message": "File uploaded successfully"
}
```

### 📂 Directory Operations

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

**Parameters:**
- `path` (required): Directory path to delete
- `recursive` (optional): Delete directory and all contents (default: false)

### 📄 File Operations

#### Delete File
```http
DELETE /api/v1/files/file?path=/mnt/user/documents/old_file.txt
```

#### Move File/Directory
```http
POST /api/v1/files/move
Content-Type: application/json

{
  "source": "/mnt/user/documents/file.txt",
  "destination": "/mnt/user/backup/file.txt",
  "overwrite": false
}
```

#### Copy File/Directory
```http
POST /api/v1/files/copy
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

### 🔍 Utilities

#### Search Files
```http
GET /api/v1/files/search?path=/mnt/user&query=*.pdf&recursive=true
```

**Parameters:**
- `path` (required): Directory to search in
- `query` (required): Search pattern (supports wildcards like `*.pdf`, `document*`)
- `recursive` (optional): Search in subdirectories (default: true)
- `case_sensitive` (optional): Case sensitive search (default: false)
- `file_type` (optional): Filter by type (file/directory)

#### Get Disk Usage
```http
GET /api/v1/files/disk-usage?path=/mnt/user
```

**Response:**
```json
{
  "path": "/mnt/user",
  "total": 1099511627776,
  "used": 549755813888,
  "free": 549755813888,
  "percent": 50.0
}
```

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DEBUG` | Enable debug mode | `false` |
| `CORS_ORIGINS` | Allowed CORS origins (JSON array) | `["http://localhost","http://localhost:8080","http://localhost:3000"]` |
| `BASE_PATH` | Base path for file operations | `/mnt/user` |
| `MAX_UPLOAD_SIZE` | Maximum file upload size in bytes | `10737418240` (10GB) |
| `SHOW_HIDDEN_FILES` | Show hidden files by default | `false` |
| `ALLOWED_EXTENSIONS` | Allowed file extensions (JSON array, `["*"]` for all) | `["*"]` |
| `API_KEY` | API key for authentication (optional) | - |
| `ENABLE_FILE_DELETION` | Enable file deletion operations | `true` |
| `ENABLE_RECURSIVE_DELETE` | Enable recursive directory deletion | `true` |
| `ENABLE_WEB_UI` | Enable the web UI (documentation only, use docker-compose profiles) | `true` |
| `UNRAID_DATA_PATH` | Host path to mount (Docker only) | `/mnt/user` |

### API Key Authentication

To secure your API, set an API key in `.env`:

```env
API_KEY=your_secret_key_here
```

Then include it in requests:

```bash
curl -H "X-API-Key: your_secret_key_here" \
  http://localhost:8000/api/v1/files/browse?path=/mnt/user
```

## Usage Examples

### cURL Examples

**Browse directory:**
```bash
curl "http://localhost:8000/api/v1/files/browse?path=/mnt/user/documents"
```

**Upload file:**
```bash
curl -X POST \
  -F "file=@/path/to/local/file.pdf" \
  -F "destination_path=/mnt/user/uploads" \
  http://localhost:8000/api/v1/files/upload
```

**Download file:**
```bash
curl -O "http://localhost:8000/api/v1/files/download?path=/mnt/user/file.txt"
```

**Search for PDF files:**
```bash
curl "http://localhost:8000/api/v1/files/search?path=/mnt/user&query=*.pdf"
```

### Flutter/Dart Example

```dart
import 'package:http/http.dart' as http;
import 'package:http_parser/http_parser.dart';
import 'dart:convert';
import 'dart:io';

class FileManagerService {
  final String baseUrl = 'http://your-server-ip:8000';
  final String? apiKey = 'your_api_key'; // Optional

  Map<String, String> get headers => apiKey != null
    ? {'X-API-Key': apiKey!}
    : {};

  // Browse directory
  Future<Map<String, dynamic>> browseDirectory(String path) async {
    final response = await http.get(
      Uri.parse('$baseUrl/api/v1/files/browse?path=$path'),
      headers: headers,
    );

    if (response.statusCode == 200) {
      return json.decode(response.body);
    } else {
      throw Exception('Failed to browse directory');
    }
  }

  // Upload file
  Future<Map<String, dynamic>> uploadFile(
    File file,
    String destinationPath
  ) async {
    var request = http.MultipartRequest(
      'POST',
      Uri.parse('$baseUrl/api/v1/files/upload'),
    );

    if (apiKey != null) {
      request.headers['X-API-Key'] = apiKey!;
    }

    request.files.add(
      await http.MultipartFile.fromPath(
        'file',
        file.path,
        contentType: MediaType('application', 'octet-stream'),
      ),
    );

    request.fields['destination_path'] = destinationPath;

    final response = await request.send();
    final responseData = await response.stream.bytesToString();

    if (response.statusCode == 200) {
      return json.decode(responseData);
    } else {
      throw Exception('Failed to upload file');
    }
  }

  // Download file
  Future<void> downloadFile(String path, String savePath) async {
    final response = await http.get(
      Uri.parse('$baseUrl/api/v1/files/download?path=$path'),
      headers: headers,
    );

    if (response.statusCode == 200) {
      await File(savePath).writeAsBytes(response.bodyBytes);
    } else {
      throw Exception('Failed to download file');
    }
  }

  // Delete file
  Future<Map<String, dynamic>> deleteFile(String path) async {
    final response = await http.delete(
      Uri.parse('$baseUrl/api/v1/files/file?path=$path'),
      headers: headers,
    );

    if (response.statusCode == 200) {
      return json.decode(response.body);
    } else {
      throw Exception('Failed to delete file');
    }
  }

  // Rename file
  Future<Map<String, dynamic>> renameFile(
    String path,
    String newName
  ) async {
    final response = await http.put(
      Uri.parse('$baseUrl/api/v1/files/rename'),
      headers: {...headers, 'Content-Type': 'application/json'},
      body: json.encode({
        'path': path,
        'new_name': newName,
      }),
    );

    if (response.statusCode == 200) {
      return json.decode(response.body);
    } else {
      throw Exception('Failed to rename file');
    }
  }

  // Search files
  Future<Map<String, dynamic>> searchFiles(
    String path,
    String query
  ) async {
    final response = await http.get(
      Uri.parse(
        '$baseUrl/api/v1/files/search?path=$path&query=$query&recursive=true'
      ),
      headers: headers,
    );

    if (response.statusCode == 200) {
      return json.decode(response.body);
    } else {
      throw Exception('Failed to search files');
    }
  }
}
```

## Web UI

### Overview

The File Manager Web UI provides a modern, visual interface for managing your files. It features:

- Material Design 3 styling with smooth animations
- Dark and light mode with persistent theme
- Drag-and-drop file uploads
- Real-time search and filtering
- Breadcrumb navigation
- File type icons
- Responsive design for all devices

### Quick Start

**Start with Web UI:**
```bash
docker-compose --profile ui up -d
```

**Access the UI:**
Open http://localhost:8764 in your browser

For detailed setup instructions, see [FRONTEND_SETUP.md](FRONTEND_SETUP.md)

### Screenshots

The Web UI provides:
- Clean file browser with breadcrumb navigation
- File operations via context menus
- Upload dialog with progress tracking
- Dark/light mode toggle
- Real-time file search

### Configuration

The Web UI is controlled via Docker Compose profiles:

**Enable UI:**
```bash
docker-compose --profile ui up -d
```

**Disable UI (API only):**
```bash
docker-compose up -d
```

For more information, see:
- [Frontend README](frontend/README.md) - Detailed UI documentation
- [Frontend Setup Guide](FRONTEND_SETUP.md) - Installation and configuration

## Docker Deployment

### Using Docker Compose (Recommended)

**With Web UI:**
```bash
# Start both API and UI
docker-compose --profile ui up -d

# View logs
docker-compose --profile ui logs -f

# Stop services
docker-compose --profile ui down
```

**API Only:**
```bash
# Start API only (no UI)
docker-compose up -d

# View logs
docker-compose logs -f

# Stop the service
docker-compose down
```

### Using Docker Run

```bash
docker run -d \
  --name file-manager-api \
  -p 8000:8000 \
  -v /mnt/user:/mnt/user \
  -e BASE_PATH=/mnt/user \
  -e API_KEY=your_secret_key \
  file-manager-api
```

### Unraid Deployment

For detailed Unraid setup instructions, including templates and troubleshooting, see:

📖 **[Unraid Setup Guide](UNRAID_SETUP.md)**

This guide covers:
- Docker Compose deployment (recommended)
- Using Unraid Docker templates
- Fixing common issues (UI restart loops, connectivity problems)
- Network configuration for API-UI communication

**Quick Unraid Setup:**
```bash
# Clone and deploy with Docker Compose
cd /mnt/user/appdata
git clone https://github.com/jandrop/file_core_api.git
cd file_core_api
cp .env.example .env
nano .env  # Configure your settings
docker-compose --profile ui up -d
```

Access at: `http://your-unraid-ip:8764`

## Development

### Running Locally (without Docker)

1. **Create a virtual environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Create `.env` file with your settings**

4. **Run the application:**
   ```bash
   python main.py
   ```

The API will be available at http://localhost:8000

### Building the Docker Image

```bash
docker build -t file-manager-api .
```

## Security Considerations

- **Use API Key Authentication**: Always set an API_KEY in production
- **Configure CORS**: Limit CORS_ORIGINS to only your trusted applications
- **File Permissions**: Ensure proper file system permissions are set
- **Disable Dangerous Operations**: Set `ENABLE_FILE_DELETION=false` if needed
- **Reverse Proxy**: Run behind a reverse proxy with HTTPS in production
- **Network Isolation**: Use Docker networks to isolate the container
- **Monitor Access**: Check logs regularly for suspicious activity
- **Path Traversal Protection**: The API includes built-in protection against directory traversal attacks
- **File Size Limits**: Configure MAX_UPLOAD_SIZE to prevent abuse

## Troubleshooting

### Permission Issues

If you get permission errors when accessing files:

1. Check that the Docker container user has proper permissions:
   ```bash
   # On Unraid, you may need to match UID/GID
   # Edit Dockerfile and change: useradd -m -u YOUR_UID appuser
   ```

2. Ensure volumes are mounted correctly in docker-compose.yml

### Cannot Access Files

1. Verify BASE_PATH is set correctly
2. Check that UNRAID_DATA_PATH points to the correct host directory
3. Ensure the directory exists and is accessible

### Upload Fails

1. Check MAX_UPLOAD_SIZE setting
2. Verify destination directory exists
3. Check disk space with `/api/v1/files/disk-usage`

## API Documentation

Once running, interactive API documentation is available at:
- **Swagger UI**: http://localhost:8000/docs 

