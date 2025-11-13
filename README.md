# Virtual Assistant (React + Node.js + Gemini)

A voice-enabled virtual assistant project with a React + Vite frontend and an Express + MongoDB backend. The assistant uses a Gemini-like service (configured via an API URL) for intent parsing and responses, can upload assistant images to Cloudinary, and authenticates users with JWT cookies.

## Repository layout

- frontend/ — React + Vite app
  - [frontend/package.json](frontend/package.json)
  - [frontend/vite.config.js](frontend/vite.config.js)
  - [frontend/src/main.jsx](frontend/src/main.jsx)
  - [frontend/src/context/UserContext.jsx](frontend/src/context/UserContext.jsx) — provides [`getGeminiResponse`](frontend/src/context/UserContext.jsx)
  - Pages: [frontend/src/pages/Home.jsx](frontend/src/pages/Home.jsx), [frontend/src/pages/SignIn.jsx](frontend/src/pages/SignIn.jsx), [frontend/src/pages/SignUp.jsx](frontend/src/pages/SignUp.jsx), [frontend/src/pages/Customize.jsx](frontend/src/pages/Customize.jsx), [frontend/src/pages/Customize2.jsx](frontend/src/pages/Customize2.jsx)
  - Components: [frontend/src/components/Card.jsx](frontend/src/components/Card.jsx)

- backend/ — Express API
  - [backend/index.js](backend/index.js)
  - Routes: [backend/routes/auth.routes.js](backend/routes/auth.routes.js), [backend/routes/user.routes.js](backend/routes/user.routes.js)
  - Controllers: [`signUp`/`Login`/`logOut`](backend/controllers/auth.controllers.js), [`getCurrentUser`/`updateAssistant`/`askToAssistant`](backend/controllers/user.controllers.js)
  - Auth middleware: [backend/middlewares/isAuth.js](backend/middlewares/isAuth.js)
  - Multer upload: [backend/middlewares/multer.js](backend/middlewares/multer.js)
  - Cloudinary helper: [backend/config/cloudinary.js](backend/config/cloudinary.js)
  - Token helper: [backend/config/token.js](backend/config/token.js)
  - Gemini wrapper: [backend/gemini.js](backend/gemini.js)
  - Mongoose model: [backend/models/user.model.js](backend/models/user.model.js)
  - DB connector: [backend/config/db.js](backend/config/db.js)

## Features

- Email/password authentication with JWT cookie:
  - Signup: [`signUp`](backend/controllers/auth.controllers.js)
  - Signin: [`Login`](backend/controllers/auth.controllers.js)
  - Logout: [`logOut`](backend/controllers/auth.controllers.js)
- Persisted user profile with assistant name/image and command history (MongoDB).
- File upload (assistant image) handled by multer and uploaded to Cloudinary: [backend/config/cloudinary.js](backend/config/cloudinary.js).
- Gemini integration for parsing commands and returning structured JSON: [backend/gemini.js](backend/gemini.js).
- Frontend uses Web Speech API for speech recognition and speech synthesis: [frontend/src/pages/Home.jsx](frontend/src/pages/Home.jsx).
- Frontend context wraps API calls: [`getGeminiResponse`](frontend/src/context/UserContext.jsx).

## Prerequisites

- Node.js >= 18 recommended
- MongoDB URL (local or cloud)
- Cloudinary account (optional if you want image uploads)
- Gemini API endpoint (or replace with your LLM endpoint)

## Environment variables (backend/.env)

Create `backend/.env` with at least:

```
MONGODB_URL=<your_mongo_connection_string>
JWT_SECRET=<your_jwt_secret>
CLOUDINARY_CLOUD_NAME=<cloud_name>          # optional if not uploading
CLOUDINARY_API_KEY=<api_key>                # optional
CLOUDINARY_API_SECRET=<api_secret>          # optional
GEMINI_API_URL=<your_gemini_api_endpoint>
PORT=5000
```

## Setup & Run

1. Backend
   - Open backend folder and install:
     ```sh
     cd backend
     npm install
     ```
   - Start server (development):
     ```sh
     npm run dev
     ```
   - Server entry: [backend/index.js](backend/index.js) — CORS origin configured to `http://localhost:5173`.

2. Frontend
   - Open frontend folder and install:
     ```sh
     cd frontend
     npm install
     ```
   - Start dev server:
     ```sh
     npm run dev
     ```
   - Frontend entry: [frontend/src/main.jsx](frontend/src/main.jsx). Vite dev server defaults to port 5173.

## API Endpoints

- Auth
  - POST /api/auth/signup -> handled by [`signUp`](backend/controllers/auth.controllers.js)
  - POST /api/auth/signin -> handled by [`Login`](backend/controllers/auth.controllers.js)
  - GET  /api/auth/logout -> handled by [`logOut`](backend/controllers/auth.controllers.js)
  - Routes: [backend/routes/auth.routes.js](backend/routes/auth.routes.js)

- User (requires auth cookie set)
  - GET  /api/user/current -> [`getCurrentUser`](backend/controllers/user.controllers.js)
  - POST /api/user/update -> [`updateAssistant`](backend/controllers/user.controllers.js) (supports multipart/form-data field `assistantImage` or `imageUrl`)
  - POST /api/user/asktoassistant -> [`askToAssistant`](backend/controllers/user.controllers.js)
  - Routes: [backend/routes/user.routes.js](backend/routes/user.routes.js)

## Important implementation details & pointers

- JWT generation uses [`genToken`](backend/config/token.js) and cookie is set in controllers: see [backend/controllers/auth.controllers.js](backend/controllers/auth.controllers.js).
- File upload temp storage: [backend/middlewares/multer.js](backend/middlewares/multer.js) stores files in `./public` and uploads to Cloudinary in [`uploadOnCloudinary`](backend/config/cloudinary.js).
- Gemini prompt & parsing: [backend/gemini.js](backend/gemini.js). The assistant expects a JSON-only reply; parsing is handled in [`askToAssistant`](backend/controllers/user.controllers.js).
- Frontend context function [`getGeminiResponse`](frontend/src/context/UserContext.jsx) calls `/api/user/asktoassistant`.

## Troubleshooting

- CORS issues — ensure frontend origin matches backend CORS config in [backend/index.js](backend/index.js).
- Cookie not sent — requests from frontend use `{ withCredentials: true }` in [frontend/src/context/UserContext.jsx](frontend/src/context/UserContext.jsx). Ensure the browser allows cookies for localhost.
- Cloudinary upload errors — check Cloudinary env vars and that `backend/config/cloudinary.js` can access the file path. Uploaded temp files are removed after upload.
- Gemini API — ensure `GEMINI_API_URL` is reachable and that the expected response format matches the parsing logic in [backend/controllers/user.controllers.js](backend/controllers/user.controllers.js).

## Deployment notes

- Set correct CORS origin and secure cookie flags in production.
- Use secure=true for cookies over HTTPS and tune `sameSite` as needed.
- Use environment-based configs for Cloudinary and Gemini.

## Useful files

- Server entry: [backend/index.js](backend/index.js)
- Auth controllers: [backend/controllers/auth.controllers.js](backend/controllers/auth.controllers.js)
- User controllers: [backend/controllers/user.controllers.js](backend/controllers/user.controllers.js)
- Gemini helper: [backend/gemini.js](backend/gemini.js)
- Frontend context: [frontend/src/context/UserContext.jsx](frontend/src/context/UserContext.jsx)
- Frontend Home (speech): [frontend/src/pages/Home.jsx](frontend/src/pages/Home.jsx)
