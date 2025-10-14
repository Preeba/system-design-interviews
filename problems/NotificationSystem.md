# **Design a Notification System**

## **Functional Requirements**
- Notifications can be sent to users across multiple channels - iOS, Android, email, SMS 
- Notification service must respect user preferences

## **Non-Functional Requirements**
- System should be fault-tolerant, highly available
- No duplicate messages (deliver only once)
- Scale upto 10 million notifications a day

## **Core Entities**
- Notification
- User
- Device
- User preferences

## **API**
- POST /notifications --> notification_id
  ```json
    messageText, recepientUserId, subject, channel[iOS, Android, SMS, email], status[created, pending, failed, success, retrying]
  ```
- GET /notifications
- GET /notifications/{notification_id}

- POST /users --> create user
- PUT /users/{userId}/preferences
  ```json
    {
      "email": true,
      "push":  false,
      "sms":  false
    }
  ```
- GET /users/{userId}/preferences
- POST /template

## **High-Level Design**
![NotificationSystemHelloInterview.png](..%2Fdiagrams%2FNotificationSystemHelloInterview.png)

## **Deep Dive**

### How do we ensure recipients receive a notification exactly once? 
We introduce a deduplication mechanism: 
When a notification event first arrives, we check if it's status. If status is successful, then we will not send the message off to the 3rd party channel provider.

### How does the worked handle errors if the 3rd party provider is not responding for some reason? 
Failures can happen for many reasons: network issues, provider downtime, rate limiting, or message rejection.
The key is resiliency: how your system continues to operate despite failures.
1. Error Detection - what kind of errors - timeouts, 500s, Invalid responses, or provider specific errors like user not found for sms/email etc. 
2. Retry Strategies - In case, max retry threshold (3-5 attempts) reached, a circuit breaker can help. 
3. Observability - Metrics - Success rates, retries, failures per provider. Alerts - Trigger if failure crosses thresholds