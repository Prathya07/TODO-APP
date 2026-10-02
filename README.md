# Deadline - React Native To-Do App (TypeScript) with Auth

Android to-do app built with **React Native CLI + TypeScript**, **Redux Toolkit**, and a
**Node.js / Express / MongoDB** backend with JWT authentication.

```
todo-app/
  backend/            Express + Mongoose API (TypeScript)
    src/config/       MongoDB connection
    src/models/       User, Task schemas
    src/middleware/   JWT auth guard
    src/routes/       /api/auth, /api/tasks
  mobile/             React Native app source (drop into a fresh RN CLI project)
    App.tsx
    src/api/          axios client + token interceptor
    src/store/        Redux Toolkit slices (auth, tasks)
    src/navigation/   auth stack vs app stack
    src/screens/      Login, Register, Tasks, TaskForm
    src/components/   Input, Chip, TaskCard, FilterBar, DateTimeField, ...
    src/utils/        smartSort (priority + deadline), date helpers
    src/theme/        colour tokens
```

## Features

- Register / log in with email + password (bcrypt hashed, JWT, session restored on launch)
- Add and edit tasks: title, description, start date-time, deadline, priority, category
- Mark complete (optimistic update), delete (with confirmation), pull to refresh
- **Smart sort**: score = priority weight + deadline urgency, overdue tasks jump up,
  completed tasks sink (see `mobile/src/utils/smartSort.ts`)
- Other sorts: deadline, priority, start time; filters by status and category
- Urgency strip on each card fills as the deadline approaches (mint -> amber -> coral)
- Progress bar for completed vs total, empty states, inline validation

## 1. Run the backend

Requires Node 18+ and MongoDB (local or Atlas).

```bash
cd backend
cp .env.example .env        # set MONGO_URI and a long random JWT_SECRET
npm install
npm run dev                 # http://localhost:5000
```

API:

| Method | Path                    | Auth | Purpose                 |
|--------|-------------------------|------|-------------------------|
| POST   | /api/auth/register      | no   | create account          |
| POST   | /api/auth/login         | no   | log in                  |
| GET    | /api/auth/me            | yes  | validate stored token   |
| GET    | /api/tasks              | yes  | list my tasks           |
| POST   | /api/tasks              | yes  | create task             |
| PUT    | /api/tasks/:id          | yes  | edit task               |
| PATCH  | /api/tasks/:id/toggle   | yes  | toggle completed        |
| DELETE | /api/tasks/:id          | yes  | delete task             |

## 2. Create the React Native project

The `mobile/` folder holds the app source. Generate the native Android scaffolding with the
CLI, then copy the source in:

```bash
npx @react-native-community/cli@latest init TodoApp
cd TodoApp

npm install @reduxjs/toolkit react-redux axios \
  @react-native-async-storage/async-storage \
  @react-native-community/datetimepicker \
  @react-navigation/native @react-navigation/native-stack \
  react-native-screens react-native-safe-area-context

# copy the app source over the template (replaces App.tsx)
cp -r ../mobile/src ./src
cp ../mobile/App.tsx ./App.tsx
```

Then run on an Android emulator or device:

```bash
npx react-native run-android
```

### Pointing the app at the backend

Edit `API_URL` in `src/api/client.ts`:

- Android emulator: `http://10.0.2.2:5000/api` (default)
- Physical device on USB: run `adb reverse tcp:5000 tcp:5000`, then use `http://localhost:5000/api`
- Physical device on Wi-Fi: use your computer's LAN IP, e.g. `http://192.168.1.20:5000/api`

## Design notes

- **State management:** Redux Toolkit. `authSlice` owns the session; `tasksSlice` owns tasks
  plus list UI state (filter, category, sort). Logging out resets the tasks slice.
- **Auth flow:** token saved in AsyncStorage; an axios interceptor attaches it; on launch
  `/auth/me` validates it. The navigator swaps between the auth and app stacks based on the token.
- **Security:** passwords hashed with bcrypt, every task query is scoped to the JWT's user id,
  inputs validated with zod, login errors do not reveal which emails exist.
