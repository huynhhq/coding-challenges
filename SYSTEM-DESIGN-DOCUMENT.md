# Real-Time Vocabulary Quiz Design System

## System Design Document
### Architecture Diagram
![Architecture Diagram](./architecture-diagram.png)

### Component Description
1. **Web Application**
Role: Serves as the front end where users interact with the application. It provides the user interface for taking quizzes and viewing scores.

2. **CMS (Content Management System)**
Role: Manages the content related to quizzes, such as questions, answers, and user guides. It allows administrators to update quiz content without needing to modify the application code.

3. **Sentry**
Role: A tool for real-time error tracking and monitoring. Sentry helps identify, track, and resolve issues as they happen in production, improving application reliability and user experience.

4. **Internet Gateway**
Role: Acts as a router to manage traffic between the internet and the internal network, ensuring that requests reach the correct services inside the network.

5. **API Gateway**
Role: Serves as the single entry point for all API requests. It routes requests to the appropriate backend services, handles load balancing, and may include other functions such as authentication, rate limiting, and request validation.

6. **User Service**
**Role**: Manages user-related functionalities like sign-up, login, user profile management, and user authentication.

7. **Quiz Service**
Role: Manages all operations related to quizzes, including storing quizzes, retrieving quiz questions, and managing user responses.

8. **Leaderboard Service**
**Role**: Maintains the leaderboard by calculating scores and rankings in real time, providing competitive features for the quiz takers.

9. **Websocket Service**
**Role**: Handles real-time communication between the client and the server. It's crucial for sending real-time updates to the client, such as leaderboard changes or quiz results.

10. **Grafana**
**Role**: An analytics and monitoring platform used to visualize application metrics and logs, enabling performance monitoring and troubleshooting.

11. **MySQL DB**
**Role**: The primary relational database used for storing structured data such as user information, quiz results, and historical data.

12. **Redis Cache**
**Role**: An in-memory data structure store used as a database cache to enhance performance by caching frequently accessed data, such as session states and leaderboard data.

13. **Mongo DB**
**Role**: A NoSQL database used for storing unstructured data or data with varied schemas, such as logs, user-generated content, or complex nested data structures.

14. **Application Load Balancer**
**Role**: Distributes incoming application traffic across multiple targets, such as EC2 instances, in multiple Availability Zones, which increases the availability of your application.

15. **Auto Scaling Group**
**Role**: Automatically adjusts the number of EC2 instances in response to traffic demand, maintaining performance and minimizing costs.

16. **Subnet & Availability Zones**
**Role**: Subnets divide a network into multiple blocks, enhancing security and performance. Availability Zones offer the ability to deploy instances in separate locations to avoid a single point of failure.

### Data Flow
**Step 1**: User Accesses the Web Application
**Action**: A user logs into the web application via the frontend interface.
**Data Flow**: The user's login credentials are sent from the Web Application to the API Gateway.

**Step 2**: User Authentication
**Action**: The API Gateway forwards the login request to the User Service for authentication.
**Data Flow**: User credentials are verified against the data stored in the MySQL DB. Assuming successful authentication, a session token (e.g., JWT) is generated and returned to the user.

**Step 3**: User Joins a Quiz
**Action**: Once authenticated, the user selects a quiz to join.
**Data Flow**: The quiz request is sent through the API Gateway, which routes it to the Quiz Service.

**Step 4**: Quiz Data Retrieval
**Action**: The Quiz Service retrieves quiz data.
**Data Flow**: This might involve fetching quiz questions stored in MongoDB. If quiz data is frequently accessed, it may be cached in Redis Cache to speed up retrieval.

**Step 5**: Quiz Participation
**Action**: The user answers quiz questions, and responses are sent back to the server.
**Data Flow**: User responses are sent to the Quiz Service through the API Gateway. The service processes the responses, calculates scores, and updates the user's score in the database.

**Step 6**: Real-Time Communication
**Action**: As the quiz progresses, scores need to be updated in real-time.
**Data Flow**: The Websocket Service facilitates real-time communication. It sends updates to the Leaderboard Service whenever scores change.

**Step 7**: Leaderboard Updates
**Action**: The Leaderboard Service calculates the latest standings based on the new scores.
**Data Flow**: Scores and user identifiers are retrieved and recalculated. This data might be cached in Redis Cache for quick access and sent back to the Websocket Service for real-time user updates.

**Step 8**: Display Updated Leaderboard
**Action**: Updated leaderboard standings are pushed to user clients.
**Data Flow**: Through the Websocket Service, updated leaderboard data is sent in real-time to all connected clients (users participating in the quiz), displaying the latest rankings directly on their web application interfaces.

**(*)Monitoring and Analytics**
Throughout this process, each component logs various metrics and performance data.
**Data Flow**: Grafana can be used to visualize these metrics by pulling data from logs stored in MongoDB or directly monitoring the services and database performance.

**(*)Security and Error Handling**
**Action**: Errors and exceptions (e.g., failed logins, database errors) are tracked.
**Data Flow**: Sentry is integrated to capture real-time errors and exceptions across the application for immediate visibility and debugging.

### Technology Justification
1. **Frontend**: ReactJS
**Why ReactJS**: ReactJS is a popular JavaScript library for building user interfaces, particularly single-page applications where dynamic content needs to be handled smoothly. It's well-suited for real-time applications like a quiz platform because of its efficient rendering and state management capabilities, which are crucial for real-time interactivity and updates.
**Benefits**: Offers a robust ecosystem, including libraries and tools for routing, state management, and more. React's component-based architecture makes it scalable and maintainable, ideal for complex user interfaces.

2. **Backend**: NestJS
**Why NestJS**: NestJS is a progressive Node.js framework for building efficient and scalable server-side applications. It uses TypeScript by default, providing improved code quality and stability through strong typing. NestJS also structures its applications around modularity, which helps in organizing code logically and maintaining large codebases.
**Benefits**: Integrates well with other libraries and supports TypeORM, Mongoose, etc., which can interact seamlessly with SQL and NoSQL databases. It also facilitates easy implementation of RESTful APIs and has built-in support for PWA features and microservices architectures.

3. **API Gateway**
**Tool**: AWS API Gateway.
Justification: Manages incoming API requests and directs them to various services. Provides features like request throttling, authentication, and API version management, which are essential for maintaining the security and efficiency of the APIs.

4. **Database: MySQL, MongoDB, Redis**
**MySQL**: A relational database management system that is perfect for structured data with complex queries.
MongoDB: A NoSQL database, ideal for handling large volumes of data with a flexible schema, useful for storing logs, user-generated content, or nested data structures.
**Redis**: Used as an in-memory data store for caching and fast data retrieval tasks like session management and real-time leaderboards.

5. **Real-Time Communication: Websocket Service**
**Technology**: Socket.IO.
**Justification**: Allows real-time, bi-directional communication between web clients and servers, which is essential for a real-time quiz application where immediate feedback and updates are crucial.

6. **Monitoring: Grafana**
**Why Grafana**: Integrates various data sources seamlessly to provide real-time visualization of metrics and logs, helping monitor application performance and system health effectively.

7. **Error Tracking**: Sentry
**Why Sentry**: Offers real-time error tracking and monitoring, which helps in quickly identifying, investigating, and fixing issues in production environments.

8. **Load Balancer**
**Technology**: AWS Elastic Load Balancer (ELB).
**Justification**: Distributes incoming network traffic across multiple servers, ensuring reliability and high availability. Essential for maintaining application performance under varying loads.

9. **Infrastructure Management**: AWS Auto Scaling, Subnets, Availability Zones
**Auto Scaling**: Automatically adjusts the capacity to maintain steady, predictable performance at the lowest possible cost.
**Subnets and Availability Zones**: Enhance security and network performance with isolated subnets while providing high availability across physical locations.