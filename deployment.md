# PrepBridge Deployment Guide (Vercel + Render)

This guide deploys your project end to end with:
- Frontend on Vercel
- Backend API on Render
- ML service on Render
- Database on MongoDB Atlas (free tier)

Use this when AWS billing is blocked and you need a full public deployment now.

## 0. Architecture (what you are deploying)

- Frontend (`Frontend`) -> Vercel
- Backend (`Backend`) -> Render Web Service (Node)
- ML Service (`ml-service`) -> Render Web Service (Python)
- MongoDB -> Atlas free cluster

## 1. Prerequisites

- GitHub account
- Vercel account
- Render account
- MongoDB Atlas account
- Project pushed to GitHub

## 2. Pre-deployment code checks (required)

### 2.1 Fix hardcoded localhost OAuth URLs in frontend

Your login page currently uses localhost directly for OAuth start URLs. In production this will break OAuth.

Current file:
- `Frontend/src/features/auth/pages/LoginPage.jsx`

Replace hardcoded values:
- `http://localhost:5000/api/auth/google`
- `http://localhost:5000/api/auth/github`

with dynamic backend base derived from `API_BASE_URL` from `Frontend/src/services/api.js`.

Suggested approach:
- Import `API_BASE_URL`
- Define `const AUTH_BASE_URL = API_BASE_URL.replace(/\/api$/, "");`
- Use:
  - `${AUTH_BASE_URL}/api/auth/google`
  - `${AUTH_BASE_URL}/api/auth/github`

### 2.2 Remove hardcoded secret key from ML service (strongly recommended)

Current file contains a hardcoded Groq key:
- `ml-service/model.py`

Use env var instead:
- `api_key = os.getenv("GROQ_API_KEY")`

Then set `GROQ_API_KEY` in Render environment variables.

If key is already exposed publicly, rotate it in Groq dashboard before deployment.

## 3. Create MongoDB Atlas free database

1. Create Atlas free cluster (M0).
2. Create DB user and password.
3. In Network Access, allow Render outbound access (for quick setup use 0.0.0.0/0, later restrict if needed).
4. Copy connection string:

Example:
`mongodb+srv://<user>:<password>@<cluster>.mongodb.net/prepbridge?retryWrites=true&w=majority`

Save this as `MONGO_URI` for backend.

## 4. Deploy ML service on Render (first)

1. Render -> New -> Web Service
2. Connect your GitHub repo.
3. Configure:
- Name: `prepbridge-ml`
- Root Directory: `ml-service`
- Runtime: Python
- Build Command:
  `pip install -r requirements.txt && pip install groq`
- Start Command:
  `uvicorn main:app --host 0.0.0.0 --port $PORT`

4. Environment variables:
- `GROQ_API_KEY=<your_rotated_key>`

5. Deploy.
6. Verify:
- `https://<your-ml-service>.onrender.com/health`

Expected: JSON with service status.

## 5. Deploy Backend on Render

1. Render -> New -> Web Service
2. Connect same GitHub repo.
3. Configure:
- Name: `prepbridge-backend`
- Root Directory: `Backend`
- Runtime: Node
- Build Command:
  `npm install`
- Start Command:
  `npm start`

4. Add environment variables:

Required:
- `NODE_ENV=production`
- `PORT=10000` (Render also injects PORT automatically; setting is optional)
- `MONGO_URI=<atlas_connection_string>`
- `JWT_SECRET=<strong_random_secret>`
- `JWT_EXPIRE=7d`
- `SESSION_SECRET=<strong_random_secret>`
- `ML_SERVICE_URL=https://<your-ml-service>.onrender.com`

CORS + redirects:
- `FRONTEND_URL=https://<your-frontend>.vercel.app`
- `FRONTEND_URLS=https://<your-frontend>.vercel.app`

Optional email vars:
- `SMTP_HOST`
- `SMTP_PORT`
- `SMTP_USER`
- `SMTP_PASS`
- `FROM_EMAIL`

OAuth (only if using Google/GitHub login):
- `GOOGLE_CLIENT_ID`
- `GOOGLE_CLIENT_SECRET`
- `GOOGLE_CALLBACK_URL=https://<your-backend>.onrender.com/api/auth/google/callback`
- `GITHUB_CLIENT_ID`
- `GITHUB_CLIENT_SECRET`
- `GITHUB_CALLBACK_URL=https://<your-backend>.onrender.com/api/auth/github/callback`

5. Deploy.
6. Verify backend health:
- `https://<your-backend>.onrender.com/api/health`

## 6. Deploy Frontend on Vercel

1. Vercel -> Add New -> Project
2. Import same GitHub repo.
3. Configure project:
- Root Directory: `Frontend`
- Framework Preset: Vite
- Build Command: `npm run build`
- Output Directory: `dist`

4. Add environment variable:
- `VITE_API_BASE_URL=https://<your-backend>.onrender.com/api`

5. Deploy.
6. Open Vercel URL and test login/dashboard routes.

## 7. Post-deploy update (important order)

After Vercel gives final domain:
1. Go to backend service env vars on Render.
2. Update:
- `FRONTEND_URL=https://<actual-vercel-domain>.vercel.app`
- `FRONTEND_URLS=https://<actual-vercel-domain>.vercel.app`
3. Redeploy backend.

If using OAuth:
1. Update OAuth app callback URLs in Google/GitHub developer consoles to Render backend callback URLs.
2. Ensure frontend redirect target route exists:
- `/oauth/callback`

## 8. Full verification checklist

### 8.1 Health checks
- Backend: `GET /api/health` returns success
- ML: `GET /health` returns success

### 8.2 API connectivity
- Frontend can fetch data from backend without CORS errors
- Backend can call ML service endpoints successfully

### 8.3 OAuth
- Google/GitHub login opens provider screen
- Callback redirects to frontend `/oauth/callback` with token

### 8.4 Socket.IO
- Community/realtime pages connect without transport/CORS failure

## 9. Common issues and fixes

### Issue: CORS blocked for origin

Fix:
- Set backend env:
  - `FRONTEND_URL=https://<vercel-domain>`
  - `FRONTEND_URLS=https://<vercel-domain>`
- Redeploy backend.

### Issue: OAuth redirects to localhost

Fix:
- Remove hardcoded localhost URLs from frontend login page.
- Ensure backend `FRONTEND_URL` points to Vercel domain.
- Verify provider callback URLs point to Render backend domain.

### Issue: Backend cannot reach ML service

Fix:
- Confirm `ML_SERVICE_URL` in backend env is exact Render ML URL (without trailing slash preferred).
- Check ML service logs in Render.

### Issue: Render service sleeps (free plan)

Fix:
- First request after idle may be slow (cold start). This is normal on free plan.

## 10. Recommended production hardening (after first successful deploy)

- Move all secrets to platform env vars only.
- Rotate any leaked keys.
- Add custom domain to Vercel and Render.
- Add uptime monitor for health endpoints.
- Add basic log alerting in Render.

## 11. Quick rollback strategy

If a deploy breaks:
1. In Vercel/Render, rollback to previous successful deployment.
2. Re-check only changed env vars.
3. Test health endpoints first, then frontend flow.

## 12. Final target URLs (fill these)

- Frontend: `https://________________.vercel.app`
- Backend: `https://________________.onrender.com`
- ML Service: `https://________________.onrender.com`
- Backend Health: `https://________________.onrender.com/api/health`
- ML Health: `https://________________.onrender.com/health`

---

If you follow this in order (ML -> Backend -> Frontend -> backend env update), your deployment should be fully working on Vercel + Render.