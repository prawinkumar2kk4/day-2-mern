# 🚀 MERN Stack Employee Management System (Day 2)

[![Node.js](https://img.shields.io/badge/Node.js-v20+-43853d?style=flat-square&logo=node.js)](https://nodejs.org)
[![React](https://img.shields.io/badge/React-v19.2.5-61dafb?style=flat-square&logo=react)](https://react.dev)
[![MongoDB](https://img.shields.io/badge/MongoDB-v7.2.0-47a248?style=flat-square&logo=mongodb)](https://www.mongodb.com)
[![Express](https://img.shields.io/badge/Express-v5.2.1-90c53f?style=flat-square&logo=express)](https://expressjs.com)
[![License](https://img.shields.io/badge/License-ISC-blue?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/Build-Success-brightgreen?style=flat-square)](#)

---

### 🌐 Live Production Links
* **Production Frontend (Vercel)**: ⚛️ [https://day-2-mern.vercel.app](https://day-2-mern.vercel.app)
* **Production Backend API (Render)**: 🔧 [https://day-2-mern.onrender.com](https://day-2-mern.onrender.com)

---

## 📖 Overview

The **MERN Employee Management System** is an enterprise-caliber, lightweight full-stack administration portal built during Day 2 of the MERN Stack Class. It serves as a unified workspace for HR operations to monitor, register, edit, and offboard employees seamlessly.

### 🎯 Problem Statement
Modern fast-growing organizations frequently suffer from fragmented and manual employee registry processes. Spreadsheets lead to:
- **Data Inconsistencies**: Duplicate records, outdated contact info, and unsynchronized data.
- **Security Vulnerabilities**: Plaintext storage of sensitive workplace positions and levels.
- **High Latency**: Poor user experience during high-volume roster reads and modifications.
- **No Shared Access**: Lack of centralized, real-time synchronization among concurrent HR managers.

### ✨ Solution
This application delivers:
- **Centralized Document-Oriented Database**: Powered by MongoDB Atlas with connection clustering.
- **Reactive State Management**: Built on React 19 to handle high-performance state synchronization without jarring full-page refreshes.
- **Optimized RESTful Gateway**: An Express.js micro-framework utilizing low-level native MongoDB driver queries for speed.
- **Responsive Layout Design**: A modern, mobile-friendly interface designed with Tailwind CSS utility classes.

### 👥 Target Users
- **HR Administrators**: Perform onboarding, roster updates, and department logging.
- **Engineering Managers**: Quickly audit experience levels and internal team roles.
- **IT Auditors**: Inspect administrative actions, levels, and security vectors.

### 💼 Real-World Importance
As organizations scale, employee registries form the baseline structural identity for authorization, payroll systems, and communications. This system serves as a scalable, high-performance foundation for centralized identity mapping.

---

## 🧠 System Architecture

### 📊 Architecture Diagram

```mermaid
graph TB
    Client["🖥️ Browser Client<br/>(React 19 + Tailwind CSS)"]
    Network["🌐 HTTP / REST API<br/>(JSON Payloads - Port 5050)"]
    Backend["🔧 Backend App Layer<br/>(Express 5.2.1 Router)"]
    Database["🗄️ Database Layer<br/>(MongoDB Atlas Cluster)"]
    
    Client -->|1. User Interaction| Client
    Client -->|2. AJAX fetch requests| Network
    Network -->|3. Route Handler| Backend
    Backend -->|4. Native Driver Query| Database
    Database -->|5. Raw Document Array| Backend
    Backend -->|6. JSON Response| Network
    Network -->|7. React State Hook Update| Client
    Client -->|8. DOM Re-render| Client
```

### 🏗️ Explanation

#### Full Data Flow & Request Lifecycle
1. **Trigger**: An HR administrator clicks the "Delete" button or submits the employee registration form.
2. **Client State Processing**: React triggers state transition hooks (`useState`, `useEffect`) and uses the native browser `fetch` API to construct JSON payloads.
3. **HTTP Transport**: The request travels securely over HTTP REST paths to the Express backend running on Port `5050`.
4. **Express Middleware Routing**: The backend processes CORS configurations, parses the incoming raw JSON stream (`express.json()`), and matches the endpoint route in `routes/record.js`.
5. **Database Resolution**: The backend issues an asynchronous query using the official `mongodb` native driver's `collection` utilities.
6. **Persistence State Update**: The MongoDB Atlas Cluster performs document writes/deletes and returns confirmation metadata to Express.
7. **Hydration & Render**: The Express server formats the result, applies suitable status headers (`200 OK`, `204 No Content`), and streams a JSON payload back to the client. The client updates state arrays, forcing React to efficiently swap nodes in the virtual DOM.

#### System Design Decisions
- **Express.js Router over Mongoose (Day 2 Focus)**: Utilizing the official MongoDB native driver directly rather than an ODM (Mongoose) provides low-level performance gains, reducing middleware execution overhead for rapid CRUD prototyping.
- **Vite Build Engine**: Replaced Webpack to reduce local Hot Module Replacement (HMR) times from seconds to milliseconds.
- **Single-Page Application (SPA) Client Routing**: Built with `react-router-dom` to support client-side paths, optimizing server asset deliveries by transmitting the HTML skeleton exactly once.

#### Scaling Strategy
- **Horizontal Scaling**: Backend servers can be cloned behind an Nginx load balancer. Since Express sessions are stateless (utilizing direct DB-level reads or future JWTs), client requests can hit any running node.
- **Database Sharding**: MongoDB's native support for horizontal partitioning (sharding) allows the `employees` collection to easily distribute records across distinct storage nodes using a hashed shard key on `_id` or `name`.

---

## 🔄 Application Flow

### 📌 Flowchart

```mermaid
flowchart TD
    Start([👤 User visits Home Portal]) --> FetchRecords[Fetch Employees via GET /record]
    FetchRecords --> DBCheck{Database connected?}
    
    DBCheck -->|No| ShowError[Display connection error state]
    DBCheck -->|Yes| RenderTable[Render interactive table]
    
    RenderTable --> UserChoice{HR Admin action?}
    
    UserChoice -->|1. View Roster| RenderTable
    
    UserChoice -->|2. Click Create| RouteCreate[Navigate to /create]
    RouteCreate --> InputForm[Fill Employee Name, Position, Level]
    InputForm --> SubmitCreate[Submit via POST /record]
    SubmitCreate --> SuccessRedirect[Reset form & navigate home]
    SuccessRedirect --> FetchRecords
    
    UserChoice -->|3. Click Edit| RouteEdit[Navigate to /edit/:id]
    RouteEdit --> LoadForm[Load record details via GET /record/:id]
    LoadForm --> ModifyForm[Modify employee attributes]
    ModifyForm --> SubmitEdit[Submit via PATCH /record/:id]
    SubmitEdit --> SuccessRedirect
    
    UserChoice -->|4. Click Delete| SubmitDelete[Submit via DELETE /record/:id]
    SubmitDelete --> ClientFilter[Filter local state records array]
    ClientFilter --> RenderTable
```

---

## 🔁 Sequence Diagram

```mermaid
sequenceDiagram
    actor HRAdmin as 👤 HR Administrator
    participant Client as ⚛️ React 19 Client
    participant Express as 🔧 Express.js Server
    participant Mongo as 🗄️ MongoDB Atlas
    
    HRAdmin->>Client: Visits Portal Dashboard
    activate Client
    Client->>Express: GET /record (Fetch all employees)
    activate Express
    Express->>Mongo: collection.find({}).toArray()
    activate Mongo
    Mongo-->>Express: Return raw employee documents
    deactivate Mongo
    Express-->>Client: Send JSON array + HTTP 200 OK
    deactivate Express
    Client-->>HRAdmin: Display Interactive Employee Directory Table
    deactivate Client

    HRAdmin->>Client: Modifies Level & Submits Update Form
    activate Client
    Client->>Express: PATCH /record/:id {name, position, level}
    activate Express
    Express->>Mongo: collection.updateOne(query, updates)
    activate Mongo
    Mongo-->>Express: Update Acknowledged {modifiedCount: 1}
    deactivate Mongo
    Express-->>Client: Send Update Status Payload + HTTP 200 OK
    deactivate Express
    Client->>Client: Redirect to "/" & update state arrays
    Client-->>HRAdmin: Display Success Notification & Hydrated List
    deactivate Client
```

---

## 🧩 Module Breakdown

### 🖥️ Client (Frontend Modules)
1. **`main.jsx`**: Bootstraps the React 19 application, initializes structural styles, and sets up `react-router-dom` browser routers.
2. **`App.jsx`**: Acts as the master layout skeleton, mounting the responsive navigation bar and defining routed endpoints (`/`, `/create`, `/edit/:id`).
3. **`Navbar.jsx`**: Houses the visual identification header, company branding, and CTA button to create a new employee.
4. **`RecordList.jsx`**: Handles the primary administrative dashboard. Manages state hook arrays, executes HTTP `GET` roster queries on mount, and dispatches API `DELETE` transactions.
5. **`Record.jsx`**: A dual-purpose dynamic form component that handles both registration and modification. It evaluates route parameters to populate form fields dynamically during edit flows.

### 🔧 Server (Backend Modules)
1. **`server.js`**: Instantiates the Express application, configures global CORS headers, establishes body parsers, registers path routers, and listens on Port `5050`.
2. **`connection.js`**: Establishes connections with the MongoDB Atlas Cluster using the official native driver, validates connectivity using a deployment ping, and exports the database cursor.
3. **`routes/record.js`**: Standardizes the REST API routes for mapping HTTP request actions to specific MongoDB Atlas queries.

### 🗄️ Database Layer
- **`employees` Database**: Contains the core transactional documents.
- **`records` Collection**: Houses documents with schemas representing individual employee profiles.

---

## ✨ Features

### 🟢 Basic Features
- **Roster Indexing**: Responsive grid displaying all recorded workforce personnel.
- **Onboarding Form**: Intuitive input forms to register a full employee's name, position, and experience tier.
- **Dynamic Deletion**: Remove redundant profiles with immediate reactive client UI filtering.

### 🟡 Advanced Features
- **Polymorphic Form Submissions**: Single component handles both record creation (POST) and updates (PATCH) depending on contextual URL parameters.
- **CORS Mitigation**: Dynamic handling of Cross-Origin Resource Sharing to allow secure local cross-port operations.
- **Real-Time Client-Side Routing**: Navigates between screens in milliseconds without making repetitive document requests for static assets.

### 🔴 Expert Features (Class Roadmap)
- **State Stabilization**: Auto-canceling duplicate state array re-fetches within `useEffect` dependencies.
- **Robust Exception Logging**: Intercepts server-side Mongo connection dropouts to prevent API server crashes.
- **URI Encoding**: Clean connection setups handling special characters within MongoDB administrative credentials.

---

## 🧰 Tech Stack (BEGINNER → ADVANCED → EXPERT)

### ⚛️ React 19 (Frontend)
- **Beginner**: Renders modular user interface elements using structured JSX.
- **Advanced**: Employs React Hooks (`useState` for form fields, `useEffect` for lifecycle requests) and client-side routing.
- **Expert**: Incorporates optimal Virtual DOM reconciliation through key properties and state separation to minimize re-render counts.

### ⚡ Vite (Development Environment)
- **Beginner**: Boots up a lightweight local development web server.
- **Advanced**: Leverages Hot Module Replacement (HMR) to preserve client-state across code saves.
- **Expert**: Runs optimized rollups to compile codebases into minified production assets.

### 🔧 Express.js 5.2.1 (Backend API Framework)
- **Beginner**: Configures active listeners on port `5050` to handle HTTP transactions.
- **Advanced**: Standardizes route mappings (`/record`, `/record/:id`) for structured JSON payloads.
- **Expert**: Employs non-blocking middleware chains to perform cross-origin request filtering and parse incoming requests.

### 🗄️ MongoDB Native Driver 7.2.0 (Database Gateway)
- **Beginner**: Connects Express directly to an Atlas cloud database instance.
- **Advanced**: Performs standard non-blocking queries utilizing async/await patterns.
- **Expert**: Interfaces with MongoDB Atlas Clusters using connection pooling, replication pings, and structured `ObjectId` query maps.

---

## 📂 Project Structure (OPTIMIZED)

### 📁 Current Structure
```text
MERN/
├── client/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Navbar.jsx
│   │   │   ├── Record.jsx
│   │   │   └── RecordList.jsx
│   │   ├── App.css
│   │   ├── App.jsx
│   │   ├── index.css
│   │   └── main.jsx
│   ├── tailwind.config.js
│   ├── vite.config.js
│   └── package.json
└── server/
    ├── db/
    │   └── connection.js
    ├── routes/
    │   └── record.js
    ├── config.env
    ├── server.js
    └── package.json
```

### 💎 Recommended Enterprise Structure
```text
MERN/
├── client/
│   ├── src/
│   │   ├── assets/                 # SVGs, custom graphics, brand assets
│   │   ├── components/
│   │   │   ├── common/             # Reusable UI controls (Buttons, Loaders, Input fields)
│   │   │   │   ├── Button.jsx
│   │   │   │   └── Input.jsx
│   │   │   └── features/           # Feature-specific components
│   │   │       ├── Navbar.jsx
│   │   │       ├── RecordForm.jsx  # Refactored Record.jsx
│   │   │       └── RecordList.jsx
│   │   ├── hooks/                  # Custom React Hooks
│   │   │   └── useRecordApi.js     # Abstracted API query actions
│   │   ├── App.jsx
│   │   ├── index.css
│   │   └── main.jsx
│   ├── .env.example                # Shared frontend configuration variables
│   ├── tailwind.config.js
│   └── package.json
├── server/
│   ├── config/                     # Centralized environment files
│   │   └── config.env
│   ├── controllers/                # Decoupled business logic controllers
│   │   └── recordController.js
│   ├── db/
│   │   └── connection.js
│   ├── middleware/                 # Security, logging, validation middlewares
│   │   ├── errorHandler.js
│   │   └── inputValidator.js
│   ├── routes/
│   │   └── record.js
│   ├── .env.example                # Shared backend environment templates
│   ├── server.js
│   └── package.json
├── docker-compose.yml              # Single-command environment orchestration
├── Dockerfile.client               # Multi-stage production client compiler
├── Dockerfile.server               # Production server containerizer
└── README.md
```

---

## ⚙️ Installation & Setup (UNIVERSAL + FOOLPROOF)

### 🖥️ System Requirements
- **Node.js**: v18.0.0 or higher (LTS recommended)
- **npm**: v9.0.0 or higher
- **MongoDB**: Access to MongoDB Atlas (Free Tier cluster works perfectly)
- **Supported OS**: Windows 10/11, macOS (Intel/Apple Silicon), Linux (Ubuntu, Debian, CentOS)

---

### 🔧 Step-by-Step Setup

#### 1️⃣ Clone the Repository
```bash
git clone https://github.com/prawinkumar2kk4/day-2-mern.git
cd day-2-mern
```

#### 2️⃣ Setup Environment Variables
- Create a `config.env` file in the `/server` folder:
  ```env
  # server/config.env
  ATLAS_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/employees?retryWrites=true&w=majority
  PORT=5050
  ```
- *Optional*: Create a `.env` file in `/client` to configure custom API paths:
  ```env
  # client/.env
  VITE_API_URL=http://localhost:5050
  ```

#### 3️⃣ Setup and Run the Backend Server
```bash
cd server
npm install
npm run start # or node server.js
```
*Expected console output:*
```text
Server listening on port 5050
Pinged your deployment. You successfully connected to MongoDB!
```

#### 4️⃣ Setup and Run the Frontend Client
```bash
# Open a new terminal window
cd client
npm install
npm run dev
```
*Expected console output:*
```text
  VITE v8.0.10  ready in 150 ms
  ➜  Local:   http://localhost:5173/
```

---

### ▶️ Run Modes

| Mode | Command (Client) | Command (Server) | Description |
| :--- | :--- | :--- | :--- |
| **Development** | `npm run dev` | `node server.js` | Enables HMR, active source mapping, and detailed console debugging logs. |
| **Production** | `npm run build` | `NODE_ENV=production node server.js` | Compiles client assets into highly optimized, static HTML/CSS bundles. |

---

### 🐳 Docker Setup

This setup allows you to run the entire stack within isolated containers using Docker.

#### 1. Create a `Dockerfile.server` inside `/server`
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 5050
CMD ["node", "server.js"]
```

#### 2. Create a `Dockerfile.client` inside `/client`
```dockerfile
# Stage 1: Build static assets
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve static assets via Nginx
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#### 3. Create a `docker-compose.yml` in the Root Directory
```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./server
      dockerfile: Dockerfile.server
    ports:
      - "5050:5050"
    environment:
      - ATLAS_URI=mongodb+srv://your_username:your_password@cluster.mongodb.net/employees?retryWrites=true&w=majority
      - PORT=5050

  frontend:
    build:
      context: ./client
      dockerfile: Dockerfile.client
    ports:
      - "80:80"
    depends_on:
      - backend
```

#### 4. Spin Up the Containers
```bash
docker-compose up --build
```
- Access the **Frontend** at: `http://localhost` (Port 80)
- Access the **Backend API** at: `http://localhost:5050`

---

## 🔐 Security & Restrictions

### 🛡️ Authentication & Authorization
- **Current Development State**: Designed for Day 2 class prototyping, this initial build uses open routes to simplify CRUD functional analysis.
- **Recommended Production Upgrade**: Integrate **JSON Web Tokens (JWT)** generated at user login and validated through an Express middleware layer.

### 🚫 Client-Side Security Vectors (Anti-Tampering)
For high-security operations, the following components can be integrated into `App.jsx` to restrict user actions:
- **Tab Switch Detection**: Intercepts active visibility state changes.
  ```javascript
  useEffect(() => {
    const handleVisibilityChange = () => {
      if (document.hidden) {
        console.warn("User has switched tabs - Administrative warning flagged.");
      }
    };
    document.addEventListener("visibilitychange", handleVisibilityChange);
    return () => document.removeEventListener("visibilitychange", handleVisibilityChange);
  }, []);
  ```
- **Copy-Paste Restrictions**: Prevents users from pasting unverified data into employee input forms.
  ```javascript
  const preventPaste = (e) => {
    e.preventDefault();
    alert("Manual typing required for security verification.");
  };
  ```

### 🔒 Data Protection Best Practices
- **Credential Hiding**: Never hardcode connection strings. Keep them inside `config.env` and add this file to your `.gitignore`.
- **Object Sanitation**: Parse parameters into explicit `ObjectId` wrappers to mitigate NoSQL injection risks.

---

## 📡 API Design

The backend server exposes the following endpoints on port `5050`:

### 1️⃣ Fetch All Employees
- **Endpoint**: `GET /record`
- **Response (`200 OK`)**:
  ```json
  [
    {
      "_id": "651a2c3b4d5e6f7a8b9c0d1e",
      "name": "Jane Doe",
      "position": "Staff Engineer",
      "level": "Senior"
    }
  ]
  ```

### 2️⃣ Fetch Single Employee
- **Endpoint**: `GET /record/:id`
- **Response (`200 OK`)**:
  ```json
  {
    "_id": "651a2c3b4d5e6f7a8b9c0d1e",
    "name": "Jane Doe",
    "position": "Staff Engineer",
    "level": "Senior"
  }
  ```

### 3️⃣ Create Employee Record
- **Endpoint**: `POST /record`
- **Payload**:
  ```json
  {
    "name": "John Smith",
    "position": "UX Architect",
    "level": "Mid"
  }
  ```
- **Response (`204 No Content`)**: Empty response body confirming successful creation.

### 4️⃣ Update Employee Record
- **Endpoint**: `PATCH /record/:id`
- **Payload**:
  ```json
  {
    "name": "John Smith",
    "position": "Lead Designer",
    "level": "Senior"
  }
  ```
- **Response (`200 OK`)**:
  ```json
  {
    "acknowledged": true,
    "modifiedCount": 1,
    "upsertedId": null,
    "upsertedCount": 0,
    "matchedCount": 1
  }
  ```

### 5️⃣ Delete Employee Record
- **Endpoint**: `DELETE /record/:id`
- **Response (`200 OK`)**:
  ```json
  {
    "acknowledged": true,
    "deletedCount": 1
  }
  ```

---

## 🗄️ Database Design

### 📊 ER Diagram

```mermaid
erDiagram
    EMPLOYEE_DATABASE ||--|{ RECORDS_COLLECTION : contains
    RECORDS_COLLECTION {
        ObjectId _id PK
        string name
        string position
        string level
    }
```

### 🧾 Explanation
- **NoSQL Schema Flexibility**: Records are stored as dynamic BSON documents.
- **`_id` Primary Key**: MongoDB automatically assigns a 12-byte unique `ObjectId` primary key to index documents.
- **Recommended Indexing**: Create an index on `name` to speed up employee lookup queries:
  ```javascript
  db.records.createIndex({ name: 1 });
  ```

---

## 🚀 DevOps & Deployment

### ⚙️ Deployment Diagram

```mermaid
graph TD
    Developer["👨‍💻 Local Developer<br/>(Git Commit & Push)"] -->|1. Push code| GitHub["🐙 GitHub<br/>(Remote Main Branch)"]
    GitHub -->|2. Trigger action webhook| CI_Pipeline["⚙️ CI/CD Pipeline<br/>(GitHub Actions Workflow)"]
    CI_Pipeline -->|3. Dockerize| ContainerRegistry["🐳 Docker Hub<br/>(Image Registry)"]
    ContainerRegistry -->|4. Pull updated image| ProdServer["☁️ Cloud Host Server<br/>(Render / AWS ECS / PM2)"]
    ProdServer -->|5. Connect| AtlasDB["🗄️ MongoDB Atlas<br/>(Database Cluster)"]
```

### ⚙️ Production Deployment Checklist
1. Build the production client bundle using `npm run build` in `/client`.
2. Configure a reverse proxy like Nginx to serve the build artifacts on Port `80`.
3. Set up PM2 to keep the backend server running in the background:
   ```bash
   npm install -g pm2
   pm2 start server.js --name "employee-management-api"
   ```

---

## 📈 Scalability & Performance

- **Minimize State Re-fetches**: Optimized the dependency array of the list retrieval `useEffect` from `[records.length]` to `[]` to prevent infinite rendering loops.
- **Payload Compression**: It is recommended to register Express compression middlewares to reduce JSON payload file transfer sizes.
- **Database Caching**: For high-volume rosters, integrate Redis to cache database read operations.

---

## 📊 Use Cases
- **Internal HR Portal**: A clean dashboard for startups to easily track current team roles and experience levels.
- **Project Resource Allocator**: Gives engineering managers a quick way to audit experience levels when assigning tasks.
- **Day 2 MERN Educational Reference**: A clean template demonstrating direct integration of MongoDB native drivers without complex ODM overhead.

---

## 🎯 Benefits

### 💻 Technical Benefits
- **Pure Native Database Operations**: Demonstrates direct use of MongoDB native cursors.
- **Asynchronous Flow Management**: Practices clean handling of asynchronous JavaScript using async/await structures.
- **Dynamic Client Routing**: Showcases client-side page transitions using React Router DOM.

### 💼 Business Benefits
- **Zero Software Licensing Overhead**: Leverages open-source tools to build a custom HR directory.
- **Increased Administrative Efficiency**: Saves time compared to maintaining employee registries in manual spreadsheets.

---

## 🔮 Future Enhancements
- **JWT-Based Authentication**: Introduce secure user login routes and access control.
- **Fuzzy Search & Filters**: Add input search bars to filter employees by name, role, or level instantly.
- **Automated CSV Backups**: Enable HR managers to export employee rosters to spreadsheets with one click.

---

## 🧹 Project Optimization Report

### ❌ Code Quality Issues Resolved
- **Infinite Retrieval Loop**: Fixed the infinite network request loop in `RecordList.jsx` by properly configuring the dependency array.
- **Eliminated Dead Packages**: Identified and removed the redundant `env` dependency, consolidating variable management under `dotenv`.

### 🎨 UI & DX (Developer Experience) Improvements
- **VS Code Unknown `@tailwind` at-rule Warnings**: Created a local `.vscode/settings.json` file configuring the VS Code CSS engine to ignore Tailwind-specific at-rules (resolving the common `unknownAtRules` warnings):
  ```json
  {
    "css.lint.unknownAtRules": "ignore"
  }
  ```
- **Perfect Column & Header Alignment**: Cleaned up the global `#root` container style in `index.css` by removing `text-align: center` and the unnecessary `border-inline` lines, which allows the table headers and record rows to align perfectly (matching the reference design).

### 🛡️ Security Fixes
- **Credentials Hidden**: Added the root `config.env` to the `.gitignore` configuration to prevent Atlas credentials from being pushed to public repositories.

---

## 📸 Screenshots

### 🖼️ Portal Dashboard View
![Portal Dashboard View](./screenshot.png)

---

## 🤝 Contribution Guide
1. **Fork** the Repository.
2. **Create a Feature Branch**: `git checkout -b feature/amazing-feature`.
3. **Commit Your Changes**: `git commit -m "feat: added fuzzy search filter"`.
4. **Push to the Branch**: `git push origin feature/amazing-feature`.
5. **Open a Pull Request**.

---

## 📜 License
This project is licensed under the **ISC License**. See the `LICENSE` file for details.