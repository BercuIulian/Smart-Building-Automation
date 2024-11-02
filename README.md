# Smart-Building-Automation

#### Bercu Iulian, FAF-211

**In order to run the app**, have Docker installed on your machine and execute the following command:
```
docker-compose build
```

**In order to run the docker image** execute the following command:for Parking_Service:

```
docker-compose up
```

### 1. Assess application suitability

Smart building automation involves integrating and managing various building systems like lighting, HVAC, security, and energy monitoring. This complexity makes it an ideal candidate for microservices, where independent services can manage distinct functions, ensuring scalability, flexibility, and fault isolation.

* **Multiple Components:** Systems like lighting, temperature control, and security can function independently but require coordinated management, making microservices essential.
* **Scalability**: Each service (e.g., HVAC, lighting) can be independently updated, scaled, or maintained without affecting others.
* **Real-time Control**: Microservices support real-time monitoring and control through protocols like WebSockets, crucial for smart buildings.

**Real-World Example**: **<i>Google Nest</i>** uses microservices to independently manage devices like thermostats, cameras, and alarms, ensuring real-time control and easy system scaling.



### 2. Define service boundaries

* **Device Control Service:** Manages control over all smart devices, including lighting, HVAC, and security devices for different building areas (e.g., floors or rooms).
* **Notification & Schedule Service:** Handles building-wide notifications (e.g., alerts for security breaches or system failures) and schedules for automated operations (like turning off lights or adjusting HVAC based on time or occupancy).

![pad](https://github.com/user-attachments/assets/3965a725-6766-45ed-b895-029eb5840903)


### 3. Choose Technology Stack and Communication Patterns

* **Gateway** These will be implemented in Python since it's efficient for tasks like routing, handling load, and communication between services. The Framework used will be Flask.

* **Services:** The microservices (Device Control and Notification/Schedule handling) will be developed in C#. Each service will manage its own database (likely through SQL Server or PostgreSQL).

* **Communication Patterns:** For Inter-Service Communication will be used RESTful APIs for direct service-to-service interactions, and for Real-Time Communication will be used WebSockets  for immediate status updates to users.



### 4. Design Data Management

#### 1. Device Control Service
* **Base URL** `/api/devices`
* **Endpoints:**

#####  1.1 GET `/api/devices` - get the list of all devices
* **Request:** None
* **Response** (JSON):

```json
{
    "id": 1,
    "name": "AC Unit 1",
    "status": "online",
    "location": "Floor 1",
    "settings": {
      "temperature": 22,
      "mode": "cooling"
    }
  }
```


#####  1.2 POST `/api/devices` - add a new device
* **Request:** (JSON):

```json
{
  "name": "New Device",
  "location": "Floor 3",
  "settings": {
    "temperature": 20,
    "mode": "heating"
  }
}
```

* **Response** (JSON):

```json
{
  "id": 3,
  "message": "Device added successfully"
}
```

#####  1.3 PUT `/api/devices/{id}` - update device settings
* **Request:** (JSON):

```json
{
  "settings": {
    "temperature": 24,
    "mode": "cooling"
  }
}
```

* **Response** (JSON):

```json
{
  "message": "Device settings updated"
}
```

#####  1.4 GET `/api/devices/{id}` - get the status of a specific device

* **Response** (JSON):

```json
{
  "id": 1,
  "name": "AC Unit 1",
  "status": "online",
  "location": "Floor 1",
  "settings": {
    "temperature": 22,
    "mode": "cooling"
  }
}
```

#####  Error messages

* **400 Bad Request:**

```json
{
  "message": "Invalid request data provided."
}
```

* **404 Not Found:**

```json
{
  "message": "Device not found."
}
```

* **500 Internal Server Error:**

```json
{
  "message": "An internal error occurred. Please try again later."
}
```

* **405 Method Not Allowed:**

```json
{
  "message": "HTTP method not supported for this endpoint."
}
```


#### 2. Notification and Scheduling Service
* **Base URL** `/api/notifications`
* **Endpoints:**

#####  2.1 POST `/api/notifications` - Send a notification to users based on events
* **Request:** (JSON):

```json
{
  "userId": 123,
  "deviceId": 1,
  "message": "The AC Unit 1 has been turned off",
  "timestamp": "2024-09-19T10:45:00"
}
```

* **Response** (JSON):

```json
{
  "message": "Notification sent successfully"
}
```

* **400 Bad Request:**

```json
{
  "message": "Invalid request body."
}
```

* **404 Not Found:**

```json
{
  "message": "User or device not found."
}
```

* **500 Internal Server Error:**

```json
{
  "message": "An unexpected error occurred while sending the notification."
}
```

#####  2.2 POST `/api/notifications` - Schedule an action on a device
* **Request:** (JSON):

```json
{
  "userId": 123,
  "deviceId": 1,
  "action": "turn off",
  "time": "2024-09-19T15:00:00"
}
```

* **Response** (JSON):

```json
{
  "message": "Action scheduled successfully"
}
```

* **400 Bad Request:**

```json
{
  "message": "Missing or invalid parameters."
}
```

* **404 Not Found:**

```json
{
  "message": "User or device not found."
}
```

* **500 Internal Server Error:**

```json
{
  "message": "Failed to schedule the action."
}
```

* **405 Method Not Allowed:**

```json
{
  "message": "A conflicting schedule already exists for the device."
}
```

#####  2.3 GET `/api/notifications` - Get the list of scheduled actions for a user

* **Response** (JSON):

```json
[
  {
    "deviceId": 1,
    "action": "turn off",
    "time": "2024-09-19T15:00:00"
  },
  {
    "deviceId": 2,
    "action": "set brightness",
    "time": "2024-09-19T18:00:00"
  }
]
```

* **404 Not Found:**

```json
{
  "message": "No scheduled actions found for the user."
}
```

* **500 Internal Server Error:**

```json
{
  "message": "Failed to retrieve scheduled actions."
}
```

#### 3. WebSocket
##### Example1 (Control)

* **Endpoint** `/ws/{id}/control`
* **Request:** (JSON):

```json
{
  "deviceId": "1234",
  "action": "turnOn"
}
```

* **Response** (JSON):

```json
{
  "status": "success",
  "message": "Device 1234 turned on"
}
```

##### Example2 (Control)

* **Endpoint** `/ws/{id}/control`
* **Request:** (JSON):

```json
{
  "deviceId": "light-3-12",
  "action": "turn_on"
}
```

* **Response** (JSON):

```json
{
  "deviceId": "light-3-12",
  "status": "on",
  "message": "Light on floor 3, room 12 turned on successfully",
  "timestamp": "2024-09-20T13:12:45Z"
}
```

* **Response Data Formats:** The data will mainly be transferred in JSON format, which is human-readable, easy to debug, and widely supported across different programming languages.



### 5. Deployment and Scaling

#### 1. Containerization (Docker):

* Docker will be used to create isolated environments for each microservice (device control, notifications). This ensures consistency across development, testing, and production.


#### 2. Orchestration (Kubernetes):

* Deploy services with Kubernetes to automate scaling and management. Each service can run multiple replicas to handle concurrent tasks, with horizontal scaling to adjust based on demand.
* Kubernetes provides automatic load balancing among service replicas to optimize performance.











