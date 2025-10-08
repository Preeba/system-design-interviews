## **Functional Requirements**
- Create a short url from a long url
  - optionally support custom alias
  - Optionally support expiration date
- be redirected to original url from the short url

## **Non-Functional Requirement**
- Low latency on redirects (<200 ms)
- The system should be support 100 DAU and 1B urls
- Ensure uniqueness of short code
- System should be reliable and available 99.99% of the time (High Availability, eventual consistency)

## Core Entities
- Original url
- Short url
- User
```sql
    url {
    shortUrl/customAlias
    longUrl
    creationTime
    expirationTime
    userId
    }
```

```sql
    user {
    userId...
    }
```

## API
// Shorten a url
- POST /urls ->  shortUrl
    ```json
      originalUrl,
      alias?,
      expirationTime?
    ```
  // redirection
- GET /urls/{shortCode} --> redirect to OriginalUrl

## **High Level Design**
![urlshortening.png](..%2Fdiagrams%2Furlshortening.png)

  - 301 redirect (permanent redirect, the resource has been permanently moved to the target URL. Browser cache this response, and subsequent requests for the same url will go directly to long url, bypassing the server)
  - 302 redirect (temporary,  the resource is temporarily located at a different URL. Browsers do not cache this response, ensuring that future requests for the short URL will always go through our server first)

A 302 redirect is preferred because:
- It gives us more control over the redirection process, allowing us to update or expire links as needed
- It prevents browsers from caching the redirect, which could cause issues if we need to change or delete the short URL in the future.
- It allows us to track click statistics for each short URL (even though this is out of scope for this design)

### **Cache eg.Redis**
  - read through, LRU cache. (key: shortCode, value: longUrl)

## **Deep Dives**

### **Ensure uniqueness of shorten code**
  - Base62 encoding, 0-9, A-Z, a-z  ( for 1B urls, 10^9 10 characters)
  - Hash the long url (md5(longUrl)-> hash -> base62(hash))

### **Scale**
Assessing from left to right of the diagram, 
- redundancy of the primary server (or go with microservice -- split the services into redirect-service/url-shortening-service or read-write separation and scale them), 
- redundancy at DB layer, cache for the read-service(redirect-service)