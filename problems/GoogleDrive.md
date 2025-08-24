
### **Google Drive** ###
Google Drive is a file hosting system powered by Google. It offers cloud file storage and synchronization service, allowing users to store their data on remote servers. Besides storing the file on these servers, Google Drive will also synchronize their files across multiple devices that they use and share it with other users as requested. Dropbox, OneDrive and Google Photos are similar applications that handle file storage and sharing for massive amounts of files for millions of users.

### **Functional Requirement**
1. Users can upload and download files from any device they are logged in.
2. Sync files across multiple devices
3. Users can share files with other users
4. See file revisions
5. Send a notification when a file is edited, deleted or shared

### **Non-Functional Requirements**

1. Reliability - Data loss is unacceptable
2. Scalability - Should be able to handle high volumes of traffic
3. High Availability - Users should be able to use the system when some servers are offline or have network errors etc
4. Fast sync speed
5. Minimum possible network bandwidth should be utilized for file synchronization

---

## **APIs**
Primarily need 3 APIs - Upload a file, download a file and get file revisions

1. **Upload a file to Google Drive**
    1. Simple upload: Use this upload type when file size is small
    2. Resumable upload: Use this upload type when file size is large and there is a high chance of network interruption

   Example: **_POST /files/upload?uploadType=resumable_**  Param as data: Local file to be uploaded

       Resumable upload follows 3 steps:
       1. Send initial request to retrieve the resumable URL
       2. Upload data and monitor upload state
       3. If upload is disturbed, resume the upload

2. **Download a file from Google Drive**

   **_GET /files/download**_
   Params: Path: download file path
   Example:
    ```json
      { 
        "path": "/recipes/soup/best_soup.txt"
      }
    
3. **Get file revisions**

   **_GET /files/revisions_**
   Params: Path - the path to the file you want to get file history
   limit - max number of revisions to return
   Example:
     ```json
         { 
           "path": "/recipes/soup/best_soup.txt",
           "limit":  20
         }
     ```
---


### **Behind the Scenes: How Resumable Upload Works**
The process involves **three main phases**:

1. **Session Initiation**
2. **Chunked Uploads**
3. **Resumption & Completion Handling**

---

### **1. Session Initiation (Getting an Upload URL)**
- The client first sends a `POST` request to the API with `uploadType=resumable`.
- The server responds with an **Upload Session URL**, which is valid for a period of time (usually 24 hours).
- This URL acts as a dedicated endpoint to upload the file in chunks.

#### **Behind the Scenes**
- The server registers a new **upload session** and assigns it a unique `upload_id`.
- It keeps track of the file metadata, the total expected size, and the current state of the upload.

---

### **2. Chunked Uploads (Actual File Uploading in Parts)**
- The client uploads the file in **small chunks**, typically using `PUT` requests.
- Each request includes:
    - `Content-Range: bytes <start>-<end>/<total_size>` to specify the chunk being uploaded.
    - `Content-Length: <size>` to indicate how much data is in the chunk.

#### **Behind the Scenes**
- The server **validates** each chunk (e.g., correct order, size, integrity).
- It **stores** each chunk in temporary storage.
- If successful, the server updates the session progress.
- It **does NOT finalize** the file until all chunks are uploaded.

---

### **3. Resumption (Handling Network Failures & Resuming Uploads)**
If a network interruption occurs, the client:
1. Queries the server for the last successfully uploaded chunk by sending a `PUT` request without a body.
2. The server responds with the **last received byte range**.
3. The client resumes uploading from that point instead of restarting from the beginning.

#### **Behind the Scenes**
- The server **persists the upload progress** (e.g., last received byte offset).
- If an upload fails, the server sends an `HTTP 308 Resume Incomplete` status.
- The client retries from the last successful chunk using the same session URL.

---

### **4. Finalization (Completing the Upload)**
- When the last chunk is received, the server **validates the entire file**.
- If everything is correct, the file is **assembled & stored permanently**.
- The server then **returns a success response (200 OK or 201 Created)**.

#### **Behind the Scenes**
- The system verifies that all chunks are **received and in order**.
- It **assembles** the final file (e.g., merges chunks).
- The file is **moved from temporary to permanent storage**.

---

### **Key Benefits**
✅ **Resilient to Network Issues** – If an upload fails, it resumes from the last successful chunk.  
✅ **Efficient for Large Files** – Reduces bandwidth waste by not restarting the upload.  
✅ **Parallel Processing** – Some APIs allow parallel chunk uploads for speed.

---

### **Example of Resumption in Action**
1. **Client Starts Uploading a 1GB File** in 10MB chunks.
2. **At 500MB, the network disconnects.**
3. **Client asks the server for the last successful byte.**
    - Server responds: "Last received byte is 499MB."
4. **Client resumes upload from 500MB** instead of restarting.

---
The `uploadType=resumable` parameter is designed for large file uploads and ensures robustness in case of network failures.

### **Comparison to Other Upload Methods**
| Upload Type            | Suitable For                  | Key Feature                           |
|------------------------|-------------------------------|---------------------------------------|
| `uploadType=media`     | Small files (few MBs)         | Single `POST` request                 |
| `uploadType=multipart` | Small/Medium files + metadata | Single request with metadata & file   |
| `uploadType=resumable` | Large files (GBs)             | Chunked upload, recovery from failure |

---
