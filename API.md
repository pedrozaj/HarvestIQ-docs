# HarvestIQ API Documentation

## Backend API

**Base URL:** `https://harvestiq-backend-production.up.railway.app`

---

### Health Check

```
GET /
```

**Response:**
```json
{
  "message": "HarvestIQ API is running!",
  "version": "1.0.0",
  "database": "PostgreSQL",
  "storage": "Cloudflare R2",
  "endpoints": {
    "health": "GET /",
    "settings": "GET /api/settings",
    "files": "GET /api/files"
  }
}
```

---

### Settings

#### Get All Settings

```
GET /api/settings
```

**Response:**
```json
{
  "site_title": "Build Smarter with HarvestIQ",
  "site_subtitle": "Built for builders",
  "site_description": "The all-in-one construction management platform..."
}
```

#### Get Setting by Key

```
GET /api/settings/:key
```

**Response:**
```json
{
  "key": "site_title",
  "value": "Build Smarter with HarvestIQ"
}
```

#### Update Setting

```
PUT /api/settings/:key
Content-Type: application/json
```

**Request Body:**
```json
{
  "value": "New Value"
}
```

**Response:**
```json
{
  "key": "site_title",
  "value": "New Value",
  "message": "Setting updated successfully"
}
```

---

### Files

#### List All Files

```
GET /api/files
```

**Response:**
```json
[
  {
    "id": 1,
    "filename": "1703358000000-document.pdf",
    "original_name": "document.pdf",
    "mime_type": "application/pdf",
    "size": 1234567,
    "storage_key": "1703358000000-document.pdf",
    "project_id": null,
    "uploaded_by": null,
    "created_at": "2025-12-23T21:00:00.000Z",
    "updated_at": "2025-12-23T21:00:00.000Z"
  }
]
```

#### Get File Metadata

```
GET /api/files/:id
```

**Response:**
```json
{
  "id": 1,
  "filename": "1703358000000-document.pdf",
  "original_name": "document.pdf",
  "mime_type": "application/pdf",
  "size": 1234567,
  "storage_key": "1703358000000-document.pdf",
  "created_at": "2025-12-23T21:00:00.000Z"
}
```

#### View File in Browser

```
GET /api/files/:id/view
```

**Response:** File content with `Content-Disposition: inline` header, allowing PDFs and images to display directly in the browser.

#### Download File

```
GET /api/files/:id/download
```

**Response:** File content with `Content-Disposition: attachment` header, forcing browser to download.

#### Upload File

```
POST /api/files
Content-Type: multipart/form-data
```

**Request Body:**
- `file` - The file to upload (required)

**Response:**
```json
{
  "id": 1,
  "filename": "1703358000000-document.pdf",
  "original_name": "document.pdf",
  "mime_type": "application/pdf",
  "size": 1234567,
  "storage_key": "1703358000000-document.pdf",
  "created_at": "2025-12-23T21:00:00.000Z"
}
```

**Example (JavaScript):**
```javascript
const formData = new FormData();
formData.append('file', fileInput.files[0]);

const response = await fetch('https://harvestiq-backend-production.up.railway.app/api/files', {
  method: 'POST',
  body: formData,
});

const result = await response.json();
console.log('Uploaded:', result.original_name);
```

#### Delete File

```
DELETE /api/files/:id
```

**Response:**
```json
{
  "message": "File deleted successfully"
}
```

---

## Error Responses

All endpoints return errors in this format:

```json
{
  "error": "Error message description"
}
```

**Status Codes:**
- `200` - Success
- `201` - Created
- `400` - Bad Request
- `404` - Not Found
- `500` - Internal Server Error
- `503` - Service Unavailable (storage not configured)

---

## CORS

The API has CORS enabled:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type
```

