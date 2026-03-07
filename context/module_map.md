# Module Map

## Backend Modules

### 1. Entry Point
**File**: `/backend/src/server.ts`
- **Responsibility**: Application bootstrap and server startup
- **Dependencies**: `app.ts`, `config/db.ts`
- **Key Functions**:
  - Loads environment variables
  - Connects to MongoDB
  - Starts Express server on port 5000

### 2. Application Configuration
**File**: `/backend/src/app.ts`
- **Responsibility**: Express application setup and middleware configuration
- **Dependencies**: `routes/task.routes.ts`, `express`, `cors`, `dotenv`
- **Key Functions**:
  - Configures CORS middleware
  - Configures JSON body parser
  - Mounts task routes at `/api/tasks`
  - Health check endpoint
  - Global error handler
- **Exports**: Default export (Express Application)
- **Current Issue**: Line 4 imports `taskRoutes` but module has no default export

### 3. Database Configuration
**File**: `/backend/src/config/db.ts`
- **Responsibility**: MongoDB connection management
- **Dependencies**: `mongoose`, `dotenv`
- **Key Functions**:
  - `connectDB()`: Async function to connect to MongoDB
  - Uses `MONGO_URI` env var or defaults to localhost
  - Logs connection success or exits on failure
- **Exports**: Default export (`connectDB` function)

### 4. Routes Module (BUG LOCATION)
**File**: `/backend/src/routes/task.routes.ts`
- **Responsibility**: API route definitions and controller mapping
- **Dependencies**: `controllers/task.controller.ts`, `express`
- **Current Code**:
  ```typescript
  import { Router } from 'express';
  import { createTask, getTasks, getTaskById, updateTask, deleteTask }
    from '../controllers/task.controller';

  const router = Router();

  router.post('/', createTask);
  router.get('/', getTasks);
  router.get('/:id', getTaskById);
  router.put('/:id', updateTask);
  router.delete('/:id', deleteTask);

  // ❌ MISSING: export default router;
  ```
- **Expected Export**: `export default router;`
- **Bug**: Router is created but not exported, causing import error in app.ts

### 5. Controllers Module
**File**: `/backend/src/controllers/task.controller.ts`
- **Responsibility**: Business logic and request handling
- **Dependencies**: `models/task.model.ts`, `express`
- **Exports**: Named exports for each controller function
  - `createTask(req, res)`: Validates title, creates task, returns 201 or 400
  - `getTasks(req, res)`: Fetches all tasks sorted by createdAt desc
  - `getTaskById(req, res)`: Validates ID format, fetches single task
  - `updateTask(req, res)`: Validates ID, updates task fields
  - `deleteTask(req, res)`: Validates ID, deletes task
- **Error Handling**: Try-catch blocks return 500 on errors

### 6. Data Model
**File**: `/backend/src/models/task.model.ts`
- **Responsibility**: Mongoose schema definition and database model
- **Dependencies**: `types/task.types.ts`, `mongoose`
- **Schema Definition**:
  ```typescript
  {
    title: String (required)
    description: String (optional)
    completed: Boolean (default: false)
    timestamps: true (auto-generates createdAt, updatedAt)
  }
  ```
- **Exports**:
  - Named export: `ITaskDocument` interface
  - Default export: Mongoose Model

### 7. Type Definitions
**File**: `/backend/src/types/task.types.ts`
- **Responsibility**: TypeScript interface definitions
- **Dependencies**: None
- **Exports**:
  ```typescript
  export interface ITask {
    title: string;
    description?: string;
    completed: boolean;
    createdAt: Date;
  }
  ```

### 8. Testing Module
**File**: `/backend/src/tests/task.controller.test.ts`
- **Responsibility**: Integration tests for API endpoints
- **Dependencies**: `app.ts`, `models/task.model.ts`, `supertest`, `jest`
- **Test Coverage**:
  - POST /api/tasks (5 test cases)
  - GET /api/tasks (3 test cases)
  - GET /api/tasks/:id (4 test cases)
  - PUT /api/tasks/:id (6 test cases)
  - DELETE /api/tasks/:id (4 test cases)
  - Edge cases (3 test cases)

---

## Frontend Modules

### 1. Entry Point
**File**: `/frontend/src/main.tsx`
- **Responsibility**: React application bootstrap
- **Dependencies**: `App.tsx`, React, ReactDOM
- **Key Functions**:
  - Mounts React app to DOM
  - Applies strict mode

### 2. Main Application Component
**File**: `/frontend/src/App.tsx`
- **Responsibility**: State management and component orchestration
- **Dependencies**: `components/TaskForm`, `components/TaskList`, `api/taskApi`
- **State**:
  - `tasks`: Array of ITask objects
  - `loading`: Boolean for loading state
- **Key Functions**:
  - `fetchTasks()`: Loads all tasks on mount
  - `handleTaskCreated()`: Creates new task and updates state
  - `handleToggleTask()`: Updates task completion status
  - `handleDeleteTask()`: Deletes task and updates state
- **Exports**: Default export (App component)

### 3. API Client Module
**File**: `/frontend/src/api/taskApi.ts`
- **Responsibility**: HTTP communication with backend API
- **Dependencies**: `axios`, `types/task.ts`
- **Base URL**: `http://localhost:5000/api/tasks`
- **Exports**: Named exports for each API function
  - `getTasks()`: GET all tasks
  - `getTask(id)`: GET single task
  - `createTask(task)`: POST new task
  - `updateTask(id, task)`: PUT task update
  - `deleteTask(id)`: DELETE task

### 4. Type Definitions
**File**: `/frontend/src/types/task.ts`
- **Responsibility**: TypeScript interface definitions
- **Dependencies**: None
- **Exports**:
  ```typescript
  export interface ITask {
    _id?: string;
    title: string;
    description?: string;
    completed: boolean;
    createdAt?: string;
    updatedAt?: string;
  }

  export interface CreateTaskInput {
    title: string;
    description?: string;
  }

  export interface UpdateTaskInput {
    title?: string;
    description?: string;
    completed?: boolean;
  }
  ```

### 5. Task Form Component
**File**: `/frontend/src/components/TaskForm.tsx`
- **Responsibility**: Task creation UI
- **Props**: `onTaskCreated(title: string, description: string)`
- **Dependencies**: `api/taskApi`, React hooks

### 6. Task List Component
**File**: `/frontend/src/components/TaskList.tsx`
- **Responsibility**: Task list container and rendering
- **Props**: `tasks`, `onToggle`, `onDelete`
- **Dependencies**: `components/TaskItem`, Framer Motion

### 7. Task Item Component
**File**: `/frontend/src/components/TaskItem.tsx`
- **Responsibility**: Individual task display
- **Props**: `task`, `onToggle`, `onDelete`
- **Dependencies**: Lucide React icons

---

## Import/Export Dependencies Graph

### Backend Critical Path (with bug)
```
server.ts
  ↓ imports
app.ts ❌ (line 4: import fails)
  ↓ should import
task.routes.ts ❌ (line 18: missing export)
  ↓ imports
task.controller.ts ✓
  ↓ imports
task.model.ts ✓
  ↓ imports
task.types.ts ✓
```

### Frontend Working Path
```
main.tsx
  ↓ imports
App.tsx ✓
  ↓ imports
taskApi.ts + TaskForm + TaskList ✓
  ↓ imports
task.ts (types) ✓
```

## Module Interaction Patterns

### Backend Pattern
1. **Routes** import **Controllers** (named imports)
2. **Controllers** import **Models** (default import)
3. **Models** import **Types** (named imports)
4. **App** imports **Routes** (default import) ❌ THIS IS BROKEN

### Frontend Pattern
1. **Components** import **API functions** (named imports)
2. **Components** import **Types** (named imports)
3. **App** imports **Components** (default imports)
4. **API** imports **Types** (type-only imports)

## Fix Required

**File**: `/backend/src/routes/task.routes.ts`
**Line**: 18 (add after line 17)
**Required Code**:
```typescript
export default router;
```

This will allow the import in `app.ts` (line 4) to work correctly:
```typescript
import taskRoutes from './routes/task.routes';
```
