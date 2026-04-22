# Smart Campus API

A RESTful API for managing Rooms and Sensors across a university campus,
built using JAX-RS (Jersey) and Java EE 8.

---

## API Overview

The Smart Campus API provides endpoints to:
- Manage campus **Rooms**
- Register and monitor **Sensors**
- Record and retrieve **Sensor Readings**
- Handle errors gracefully with custom exception mappers

Base URL: `http://localhost:8080/SmartCampusAPI/api/v1`

---

## Technology Stack

- Java EE 8
- JAX-RS (Jersey 2.41)
- Jackson (JSON)
- Apache Tomcat 9
- Maven
- NetBeans 24

---

## How to Build and Run

### Prerequisites
- Java JDK 17 or higher
- Apache Tomcat 9
- NetBeans 24 (or any Maven-supported IDE)

### Steps

1. Clone the repository:  https://github.com/ThinaraSenarathne/SmartCampusAPI.git
   
2. Open the project in NetBeans:
   - File → Open Project
   - Navigate to the cloned folder
   - Click Open

3. Build the project:
   - Right-click project → Clean and Build

4. Run the project:
   - Right-click project → Run
   - Server: Apache Tomcat or TomEE

5. API is now running at: http://localhost:8080/SmartCampusAPI/api/v1

---

## Sample curl Commands

### 1. Discovery Endpoint
```bash
curl -X GET http://localhost:8080/SmartCampusAPI/api/v1
```

### 2. Create a Room
```bash
curl -X POST http://localhost:8080/SmartCampusAPI/api/v1/rooms \
-H "Content-Type: application/json" \
-d "{\"id\":\"LIB-301\",\"name\":\"Library Quiet Study\",\"capacity\":50}"
```

### 3. Get All Rooms
```bash
curl -X GET http://localhost:8080/SmartCampusAPI/api/v1/rooms
```

### 4. Create a Sensor
```bash
curl -X POST http://localhost:8080/SmartCampusAPI/api/v1/sensors \
-H "Content-Type: application/json" \
-d "{\"id\":\"TEMP-001\",\"type\":\"Temperature\",\"status\":\"ACTIVE\",\"currentValue\":0.0,\"roomId\":\"LIB-301\"}"
```

### 5. Get Sensors filtered by type
```bash
curl -X GET http://localhost:8080/SmartCampusAPI/api/v1/sensors?type=Temperature
```

### 6. Add a Sensor Reading
```bash
curl -X POST http://localhost:8080/SmartCampusAPI/api/v1/sensors/TEMP-001/readings \
-H "Content-Type: application/json" \
-d "{\"value\":23.5}"
```

### 7. Delete a Room
```bash
curl -X DELETE http://localhost:8080/SmartCampusAPI/api/v1/rooms/LIB-301
```

---

## Report — Answers to Questions

### Part 1.1 — JAX-RS Resource Lifecycle

### Question: In your report, explain the default lifecycle of a JAX-RS Resource class. Is a
new instance instantiated for every incoming request, or does the runtime treat it as a
singleton? Elaborate on how this architectural decision impacts the way you manage and
synchronize your in-memory data structures (maps/lists) to prevent data loss or race conditions.

- By default, JAX-RS creates a new instance of a resource class for every
incoming HTTP request. This is called request-scoped lifecycle. This means
each request gets its own object, which avoids thread-safety issues between
requests. However, since a new instance is created per request, you cannot
store shared data inside instance variables of the resource class. To share
data across requests, we use a static DataStore class with static HashMaps.
These static data structures are shared across all instances, so they must
be managed carefully to avoid race conditions in a multi-threaded environment.

### Part 1.2 — HATEOAS

### Question: Why is the provision of ”Hypermedia” (links and navigation within responses)
considered a hallmark of advanced RESTful design (HATEOAS)? How does this approach
benefit client developers compared to static documentation?

- HATEOAS (Hypermedia as the Engine of Application State) means that API
responses include links to related resources and actions. This is considered
a hallmark of advanced RESTful design because it makes the API
self-documenting. Client developers do not need to memorize or hardcode
URLs — they can discover available actions from the responses themselves.
This reduces coupling between client and server, making the API easier to
evolve without breaking existing clients.

### Part 2.1 — ID-only vs Full Object Returns

### Question: When returning a list of rooms, what are the implications of returning only
IDs versus returning the full room objects? Consider network bandwidth and client side
processing.

- Returning only IDs in a list reduces network bandwidth and response size,
which is beneficial when the client only needs to know what exists.
However, it requires the client to make additional requests to fetch details
for each item, increasing the number of round trips. Returning full objects
increases payload size but reduces the number of requests needed. For most
use cases, returning full objects is preferred as it reduces client-side
complexity and latency.

### Part 2.2 — DELETE Idempotency

### Question: Is the DELETE operation idempotent in your implementation? Provide a detailed
justification by describing what happens if a client mistakenly sends the exact same DELETE
request for a room multiple times.

- Yes, DELETE is idempotent in this implementation. The first DELETE request
removes the room and returns 200 OK. Any subsequent DELETE request for the
same room ID will return 404 Not Found because the room no longer exists.
The server state remains the same after the first deletion — the room is
gone regardless of how many times the request is repeated. This satisfies
the definition of idempotency: multiple identical requests produce the same
server state.

### Part 3.1 — @Consumes and Content-Type Mismatch

### Question: We explicitly use the @Consumes (MediaType.APPLICATION_JSON) annotation on
the POST method. Explain the technical consequences if a client attempts to send data in
a different format, such as text/plain or application/xml. How does JAX-RS handle this
mismatch?

- The @Consumes(MediaType.APPLICATION_JSON) annotation tells JAX-RS that the
endpoint only accepts requests with Content-Type: application/json. If a
client sends data with a different Content-Type such as text/plain or
application/xml, JAX-RS will automatically reject the request and return
HTTP 415 Unsupported Media Type. The request never reaches the resource
method. This protects the API from receiving data it cannot process.

### Part 3.2 — @QueryParam vs Path Parameter for Filtering

### Question: You implemented this filtering using @QueryParam. Contrast this with an alternative design where the type is part of the URL path (e.g., /api/vl/sensors/type/CO2). Why
is the query parameter approach generally considered superior for filtering and searching
collections?

- Using @QueryParam for filtering (e.g. /sensors?type=CO2) is superior to
using a path parameter (e.g. /sensors/type/CO2) because query parameters
are optional by nature. The base resource /sensors still returns all sensors
when no filter is provided. Path parameters imply a specific resource
identity, which is semantically incorrect for filtering. Query parameters
are the standard REST convention for filtering, searching, and sorting
collections.

### Part 4.1 — Sub-Resource Locator Pattern

### Question: Discuss the architectural benefits of the Sub-Resource Locator pattern. How
does delegating logic to separate classes help manage complexity in large APIs compared
to defining every nested path (e.g., sensors/{id}/readings/{rid}) in one massive controller class?

- The Sub-Resource Locator pattern delegates request handling to a separate
class based on the URL path. Instead of defining all nested endpoints in
one large controller, each sub-resource gets its own dedicated class. This
improves code organisation, readability, and maintainability. In large APIs
with many nested resources, this pattern prevents resource classes from
becoming too large and complex. It also allows sub-resources to be tested
and developed independently.

### Part 5.2 — HTTP 422 vs 404

### Question: Why is HTTP 422 often considered more semantically accurate than a standard

-HTTP 404 means the requested URL resource was not found. HTTP 422 means
the request was understood and the URL was valid, but the content of the
request body was semantically incorrect. When a client POSTs a sensor with
a roomId that does not exist, the URL /sensors is valid and found, but the
referenced room inside the JSON payload does not exist. Therefore 422 is
more semantically accurate because the problem is with the data content,
not the URL itself.

### Part 5.2 — Security Risks of Stack Traces

### Question: From a cybersecurity standpoint, explain the risks associated with exposing
internal Java stack traces to external API consumers. What specific information could an
attacker gather from such a trace?

- 404 when the issue is a missing reference inside a valid JSON payload
Exposing Java stack traces to external API consumers is a serious security
risk. Stack traces reveal internal package names and class structures,
library names and versions (which can be cross-referenced with known
vulnerabilities), file paths on the server, and the internal logic flow of
the application. Attackers can use this information to identify outdated
libraries with known exploits, map the internal architecture of the system,
and craft targeted attacks. The global ExceptionMapper prevents this by
returning only a generic error message.

### Part 5.3 — JAX-RS Filters for Logging

### Question: Why is it advantageous to use JAX-RS filters for cross-cutting concerns like
logging, rather than manually inserting Logger.info() statements inside every single resource method?

- Using JAX-RS filters for cross-cutting concerns like logging is superior
to manually inserting Logger.info() in every resource method because filters
are applied automatically to every request and response without modifying
business logic code. This follows the separation of concerns principle.
If logging needs to be changed or disabled, only the filter class needs to
be modified. Manual logging in every method leads to code duplication,
inconsistency, and makes maintenance harder.

---

## Project Structure
```
SmartCampusAPI/
├── src/main/java/com/smartcampus/
│   ├── SmartCampusApplication.java
│   ├── model/
│   │   ├── Room.java
│   │   ├── Sensor.java
│   │   └── SensorReading.java
│   ├── resource/
│   │   ├── DiscoveryResource.java
│   │   ├── RoomResource.java
│   │   ├── SensorResource.java
│   │   └── SensorReadingResource.java
│   ├── store/
│   │   └── DataStore.java
│   ├── exception/
│   │   ├── RoomNotEmptyException.java
│   │   ├── RoomNotEmptyExceptionMapper.java
│   │   ├── LinkedResourceNotFoundException.java
│   │   ├── LinkedResourceNotFoundExceptionMapper.java
│   │   ├── SensorUnavailableException.java
│   │   ├── SensorUnavailableExceptionMapper.java
│   │   └── GlobalExceptionMapper.java
│   └── filter/
│       └── ApiLoggingFilter.java
└── src/main/webapp/WEB-INF/
└── web.xml
```
---

*Developed for 5COSC022W Client-Server Architectures Coursework 2025/26*
*University of Westminster*
