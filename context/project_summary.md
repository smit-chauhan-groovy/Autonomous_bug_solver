# Project Summary

## Project Overview
This is a **Full-Stack Task Management Application** built with:
- **Backend**: Node.js, Express.js, TypeScript, MongoDB/Mongoose
- **Frontend**: React, TypeScript, Vite, TailwindCSS, Framer Motion
- **Testing**: Jest, Supertest, MongoDB Memory Server

## Purpose
A CRUD (Create, Read, Update, Delete) task management system that allows users to:
- Create tasks with titles and descriptions
- View all tasks in a list
- Toggle task completion status
- Delete tasks
- Real-time updates with smooth animations

## Technology Stack

### Backend
- **Express.js**: REST API server
- **TypeScript**: Type-safe JavaScript
- **MongoDB/Mongoose**: NoSQL database with ODM
- **CORS**: Cross-origin resource sharing
- **Jest**: Testing framework
- **Supertest**: HTTP assertion library
- **Nodemon**: Development server with auto-reload

### Frontend
- **React 19**: UI library
- **TypeScript**: Type-safe JavaScript
- **Vite**: Build tool and dev server
- **Axios**: HTTP client for API requests
- **TailwindCSS**: Utility-first CSS framework
- **Framer Motion**: Animation library
- **Lucide React**: Icon library

## Project Structure

```
Demo-Project/
├── backend/                    # Express.js API server
│   ├── src/
│   │   ├── config/
│   │   │   └── db.ts          # MongoDB connection
│   │   ├── controllers/
│   │   │   └── task.controller.ts  # Task business logic
│   │   ├── models/
│   │   │   └── task.model.ts  # Mongoose Task schema
│   │   ├── routes/
│   │   │   └── task.routes.ts # Task API routes
│   │   ├── types/
│   │   │   └── task.types.ts  # TypeScript interfaces
│   │   ├── tests/
│   │   │   └── task.controller.test.ts  # API tests
│   │   ├── app.ts             # Express app configuration
│   │   └── server.ts          # Server entry point
│   ├── package.json
│   ├── tsconfig.json
│   └── jest.config.js
│
├── frontend/                   # React application
│   ├── src/
│   │   ├── api/
│   │   │   └── taskApi.ts     # API client functions
│   │   ├── components/
│   │   │   ├── TaskForm.tsx   # Task creation form
│   │   │   ├── TaskItem.tsx   # Individual task display
│   │   │   └── TaskList.tsx   # Task list container
│   │   ├── types/
│   │   │   └── task.ts        # TypeScript interfaces
│   │   ├── App.tsx            # Main app component
│   │   └── main.tsx           # React entry point
│   ├── package.json
│   └── vite.config.ts
│
└── Autonomous_bug_solver/      # Bug fixing pipeline
    ├── agents/                 # Agent scripts
    ├── context/                # Project context (this directory)
    ├── logs/                   # Execution logs
    ├── orchestrator/           # Pipeline orchestration
    ├── state/                  # Pipeline state management
    └── tasks/                  # Bug task definitions
```

## Key Features

### API Endpoints
- `POST /api/tasks` - Create a new task
- `GET /api/tasks` - Get all tasks (sorted by createdAt desc)
- `GET /api/tasks/:id` - Get a single task by ID
- `PUT /api/tasks/:id` - Update a task
- `DELETE /api/tasks/:id` - Delete a task

### Data Model
**Task Schema:**
```typescript
{
  title: string (required)
  description: string (optional)
  completed: boolean (default: false)
  createdAt: Date (auto-generated)
  updatedAt: Date (auto-generated)
}
```

## Development Workflow

### Backend Development
```bash
cd backend
npm run dev          # Start development server (port 5000)
npm run build        # Compile TypeScript
npm run test         # Run tests
npm run test:watch   # Run tests in watch mode
```

### Frontend Development
```bash
cd frontend
npm run dev          # Start Vite dev server (port 5173)
npm run build        # Build for production
npm run preview      # Preview production build
```

## Current Bug

**Bug ID**: d234c746-6cde-476e-a294-0c483f7d6e38
**Title**: Route Issue
**Error**: TypeScript build error - Module '"./routes/task.routes"' has no default export
**Location**: backend/src/app.ts:4:8

The issue is in `/backend/src/routes/task.routes.ts` where the router is created but not exported, causing an import error in `/backend/src/app.ts`.

## Additional Notes

### MongoDB Configuration
- Default connection: `mongodb://localhost:27017/taskdb`
- Test database: `mongodb://localhost:27017/taskdb_test`
- Can be overridden with `MONGO_URI` environment variable

### Port Configuration
- Backend: 5000 (default)
- Frontend: 5173 (Vite default)

### TypeScript Configuration
- Backend: CommonJS modules, ES2020 target
- Frontend: ES modules (type: "module" in package.json)
