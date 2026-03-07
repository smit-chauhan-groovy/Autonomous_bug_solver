# Architecture Map

## System Architecture Overview

### Architecture Pattern
**Client-Server Architecture** with REST API communication

```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend (React)                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ TaskForm │  │ TaskList │  │ TaskItem │  │  taskApi │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└────────────────────────────┬────────────────────────────────┘
                             │ HTTP (Axios)
                             │ REST API
┌────────────────────────────┴────────────────────────────────┐
│                    Backend (Express.js)                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                     app.ts                            │   │
│  │  - Middleware (CORS, JSON parser)                    │   │
│  │  - Route registration                                │   │
│  │  - Error handling                                    │   │
│  └──────────────────────────────────────────────────────┘   │
│                              │                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                task.routes.ts                         │   │
│  │  POST   /api/tasks      → createTask                 │   │
│  │  GET    /api/tasks      → getTasks                   │   │
│  │  GET    /api/tasks/:id  → getTaskById                │   │
│  │  PUT    /api/tasks/:id  → updateTask                 │   │
│  │  DELETE /api/tasks/:id  → deleteTask                 │   │
│  └──────────────────────────────────────────────────────┘   │
│                              │                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              task.controller.ts                       │   │
│  │  - Request validation                                │   │
│  │  - Business logic                                    │   │
│  │  - Response formatting                               │   │
│  └──────────────────────────────────────────────────────┘   │
│                              │                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                task.model.ts                          │   │
│  │  - Mongoose Schema definition                         │   │
│  │  - MongoDB ODM model                                 │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────┘
                             │
                    ┌────────┴────────┐
                    │   MongoDB       │
                    │   Database      │
                    └─────────────────┘
```

## Data Flow

### Create Task Flow
```
User Input (TaskForm)
    ↓
taskApi.createTask()
    ↓
HTTP POST /api/tasks
    ↓
Express Router (task.routes.ts)
    ↓
Controller (task.controller.ts)
    ↓
Mongoose Model (task.model.ts)
    ↓
MongoDB
    ↓
Response → TaskForm → Update UI
```

### Get Tasks Flow
```
Component Mount (App.tsx)
    ↓
taskApi.getTasks()
    ↓
HTTP GET /api/tasks
    ↓
Express Router (task.routes.ts)
    ↓
Controller (task.controller.ts)
    ↓
Mongoose Model (task.model.ts)
    ↓
MongoDB Query (sort by createdAt desc)
    ↓
Response → TaskList → Render Tasks
```

## Module Communication

### Frontend Communication
- **TaskForm** → calls **taskApi** functions
- **TaskList** → receives tasks from **App** state
- **App** → orchestrates all task operations
- **taskApi** → makes HTTP requests to backend

### Backend Communication
- **app.ts** → imports and mounts routes
- **task.routes.ts** → imports controller functions
- **task.controller.ts** → imports Mongoose model
- **task.model.ts** → defines MongoDB schema

## Error Handling Strategy

### Frontend Error Handling
```typescript
try {
  await taskApi.createTask(data);
  // Update UI on success
} catch (error) {
  console.error("Failed to create task:", error);
  // Show error to user
}
```

### Backend Error Handling
1. **Controller Level**: Try-catch blocks return 500 on errors
2. **Route Level**: MongoDB ID validation returns 404 for invalid IDs
3. **App Level**: Global error handler catches unhandled errors
4. **Response Format**: `{ message: string, error?: any }`

## Import/Export Patterns

### Backend Pattern (CommonJS-style with TypeScript)
```typescript
// Named exports for controllers
export const createTask = async (...) => { ... };

// Default export for routes
export default router;

// Default export for app
export default app;
```

### Frontend Pattern (ES Modules)
```typescript
// Named exports for API functions
export const getTasks = async (...) => { ... };

// Default export for components
export default App;

// Type-only exports
export interface ITask { ... }
```

## Current Bug Location in Architecture

```
app.ts (line 4)
    ↓
import taskRoutes from './routes/task.routes';
    ↓
❌ ERROR: No default export found
    ↓
task.routes.ts (line 10-16)
    - Creates router
    - Adds routes
    - ❌ MISSING: export default router
```

## Key Architectural Decisions

1. **Separation of Concerns**: Routes, controllers, and models are in separate files
2. **Type Safety**: TypeScript interfaces defined in both backend and frontend
3. **RESTful Design**: Standard HTTP methods and status codes
4. **Middleware Pattern**: CORS and JSON parsing middleware in app.ts
5. **Database Abstraction**: Mongoose ODM for MongoDB operations
6. **Testing Strategy**: Integration tests using Supertest + Jest
