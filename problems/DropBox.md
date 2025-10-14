# Design Dropbox or Google Drive

Dropbox is a cloud-based file storage service that allows users to store and share files. It provides a secure and reliable way to store and access files from anywhere, on any device.

## **Functional Requirements**
- User should be able to upload a file from any device
- User should be able to download a file from any device
- User should be able to share a file with other users and view files shared with them
- User can automatically sync files across devices

## **Non-Functional Requirements**
- Prioritizing availability over consistency (meaning it's okay for user with whom the file is shared sees a stale version of the file)
- low latency upload and download
- Support large files 
  - resumable uploads
- High data integrity (sync accuracy is high)
- Durability – no data loss (multi-replica storage)

## **Core Entities**
- file - raw data that's uploaded/downloaded/shared
- fileMetadata - metadata associated with this file
- user - user of our system

## **API/Interface**
- POST /files/upload -> 200
    - body: file and file metadata

- GET /files/{fileId}/download -> file & file metadata    
- POST /files/{fileId}/share 
  - body: {user[] // users to share the file with }
- GET /files/{fileId}/changes?since={timestamp} -> FileMetadata[]

## **High-Level Design**
![dropbox.png](..%2Fdiagrams%2Fdropbox.png)

*   **Uploader**: This is the client that uploads the file. It could be a web browser, a mobile app, or a desktop app. It is also responsible for proactively identifying local changes and pushing the updates to remote storage.
*   **Downloader**: This is the client that downloads the file. Of course, this can be the same client as the uploader, but it doesn't have to be. We separate them in our design for clarity. It is also responsible for determining when a file it has locally has changed on the remote server and downloading these changes.
*   **LB & API Gateway**: This is the load balancer and API Gateway that sits in front of our application servers. It's responsible for routing requests to the appropriate server and handling things like SSL termination, rate limiting, and request validation.
*   **File Service**: The file service is only responsible for writing to and from the file metadata db as well as generating presigned URLs using the S3 SDK. It doesn't actually handle the file upload or download. It's just a middleman between the client and S3.
*   **File Metadata DB**: This is where we store metadata about the files. This includes things like the file name, size, MIME type, and the user who uploaded the file. We also store a shared files table here that maps files to users who have access to them. We use this table to enforce permissions when a user tries to download a file.
*   **S3**: This is where the files are actually stored. We upload and download files directly to and from S3 using the presigned URLs we get from the file server.
*   **CDN**: This is a content delivery network that caches files close to the user to reduce latency. We use the CDN to serve files to the downloader. Optional for the design as it's expensive to have CDN.

## **Deep Dives**

### Support for Large file uploads with Resumable option
Cloud service providers like Amazon S3 have a feature called **Multipart Upload** that allows uploading large objects in parts. The client breaks the file into parts[chunks] and uploads each part to S3. S3 then combines the parts into a single object. They even provide a handy JavaScript SDK which will handle all of the chunking and uploading for you.

With S3 multipart uploads, event notifications only trigger when the entire multipart upload is completed (when all parts are assembled), not for individual part uploads. For tracking individual part progress, you'd need to use S3's ListParts API or verify parts exist using HEAD requests.

Before cloud providers had this option, below is the approach taken in practice. Here is what will happen when a user uploads a large file:

* The client will chunk the file into 5-10Mb pieces and calculate a fingerprint for each chunk. A fingerprint is a mathematical calculation that generates a unique hash value based on the content of the file. It will also calculate a fingerprint for the entire file, this becomes the fileId.
* The client will send a GET request to fetch the FileMetadata for the file with the given fileId (fingerprint) in order to see if it already exists -- in which case, we can resume the upload.
* If the file does not exist, the client will POST a request to /files/presigned-url to get a presigned URL for the file. The backend will save the file metadata in the FileMetadata table with a status of "uploading" and the chunks array will be a list of the chunk fingerprints with a status of "not-uploaded".
* The client will then upload each chunk to S3 using the presigned URL. After each chunk is uploaded, the client sends a PATCH request to our backend with the chunk status and ETag. Our backend can then verify the chunk upload with S3 (using HEAD requests or ListParts API) before updating the chunks field in the FileMetadata table to mark the chunk as "uploaded".
* Once all chunks in our chunks array are marked as "uploaded", the backend will update the FileMetadata table to mark the file as "uploaded".

### low latency upload and download
We've already touched on a few ways to speed up both download and upload respectively, but there is still more we can do to make the system as fast as possible. To recap, for download we used a CDN to cache the file closer to the user. This made it so that the file doesn't have to travel as far to get to the user, reducing latency and speeding up download times. For upload, chunking, beyond being useful for resumable uploads, also plays a significant role in speeding up the upload process.

we can also utilize compression to speed up both uploads and downloads. Compression reduces the size of the file, which means fewer bytes need to be transferred. We can compress a file on the client before uploading it and then decompress it on the server after it's uploaded. We can also compress the file on the server before sending it to the client and then rely on the client to decompress it.
