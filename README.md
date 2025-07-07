# Cloudflare Worker Google Drive to R2 Uploader

A web-based tool for uploading large files from Google Drive to Cloudflare R2 storage using chunked uploads via Cloudflare Workers.

![Screenshot](Screenshot%202025-07-05%20173707.png)

## Features

- **Google Drive Integration**: Direct upload from Google Drive using file IDs
- **Chunked Uploads**: Splits large files into manageable chunks for reliable uploads
- **Progress Tracking**: Real-time progress bar and status updates
- **Session Management**: Automatic session creation and cleanup
- **R2 Integration**: Direct upload to Cloudflare R2 storage
- **Folder Support**: Optional folder organization in R2
- **OAuth2 Authentication**: Secure Google Drive API access

## Setup

### 1. Google Drive API Configuration

The worker is pre-configured with Google Drive API credentials. If you need to use your own:

1. Create a Google Cloud Project
2. Enable Google Drive API
3. Create OAuth2 credentials
4. Get a refresh token
5. Update the `authConfig` object in `index.js`:

```javascript
const authConfig = {
    client_id: "YOUR_CLIENT_ID",
    client_secret: "YOUR_CLIENT_SECRET", 
    refresh_token: "YOUR_REFRESH_TOKEN",
    secret_key: "YOUR_API_KEY"
};
```

### 2. Deploy the Cloudflare Worker

You need to deploy the Cloudflare Worker that handles the chunked uploads. The worker supports these endpoints:

- `GET /?gdid=<google_drive_file_id>` - Create upload session from Google Drive
- `GET /upload?session=<session_id>&chunk=<chunk_number>&folder=<optional_folder>` - Upload chunk
- `GET /progress?session=<session_id>` - Check upload progress
- `GET /clear?session=<session_id>` - Clear session data
- `GET /debug?session=<session_id>` - Debug session information

### 3. Configure the Frontend

Update the `WORKER_BASE` constant in `index.php` with your deployed worker URL:

```javascript
const WORKER_BASE = "https://your-worker.your-subdomain.workers.dev";
```

## Usage

### Getting Google Drive File ID

1. Open the file in Google Drive
2. The file ID is in the URL: `https://drive.google.com/file/d/FILE_ID_HERE/view`
3. Copy the `FILE_ID_HERE` part

### Upload Process

1. Open `index.php` in a web browser
2. Enter the Google Drive file ID you want to upload
3. Optionally specify an R2 folder path
4. Click "Start Upload" to begin the chunked upload process
5. Monitor progress and wait for completion

### API Usage Examples

```javascript
// Start session with Google Drive ID
fetch('/?gdid=1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms')

// Upload chunks
fetch('/upload?session=session_abc123&chunk=0&folder=my_folder')
fetch('/upload?session=session_abc123&chunk=1&folder=my_folder')

// Check progress
fetch('/progress?session=session_abc123')

// Clear session
fetch('/clear?session=session_abc123')
```

## How It Works

1. **Session Creation**: Worker fetches file metadata from Google Drive API
2. **Chunk Calculation**: File is split into 25MB chunks
3. **Multipart Upload**: R2 multipart upload is initialized
4. **Chunk Download**: Each chunk is downloaded from Google Drive using Range headers
5. **Chunk Upload**: Chunks are uploaded to R2 as multipart parts
6. **Completion**: Multipart upload is completed when all chunks are uploaded

## Files

- `index.php` - Main web interface for the uploader
- `index.js` - Cloudflare Worker code with Google Drive integration

## Requirements

- Web server with PHP support
- jQuery (loaded from CDN)
- Deployed Cloudflare Worker with R2 integration
- Google Drive API access

## Environment Variables

The Cloudflare Worker requires these environment variables:

- `UPLOADS_KV` - KV namespace for session storage
- `R2` - R2 bucket binding

## License

MIT License 