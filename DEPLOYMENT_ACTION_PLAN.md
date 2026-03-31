# 🚀 PrepBridge Free Deployment Plan
**Target:** Vercel (Frontend) + Render (Backend API + ML Service) + MongoDB Atlas (Database)  
**Cost:** Completely FREE (all services have generous free tiers)  
**Status:** ✅ Code pre-checked and ready for deployment

---

## ✅ Pre-Deployment Checks (COMPLETED)

- [x] Removed hardcoded Groq API key from `ml-service/model.py` → Now uses `GROQ_API_KEY` env var
- [x] Fixed hardcoded localhost OAuth URLs in `Frontend/LoginPage.jsx` → Now uses dynamic `VITE_API_BASE_URL`
- [x] Added `groq` package to `ml-service/requirements.txt`
- [x] Verified DB connection uses `MONGO_URI` env var
- [x] Verified all services have proper CORS configuration

---

## 📋 Deployment Order (Critical - Follow Exactly)

### Phase 1: Database Setup ⚙️
### Phase 2: Deploy ML Service 🤖
### Phase 3: Deploy Backend API 🔌
### Phase 4: Deploy Frontend 🎨
### Phase 5: Final Configuration 🔧

---

## Phase 1: MongoDB Atlas Setup (10 mins)

### Step 1.1: Create Free MongoDB Atlas Cluster
1. Go to [mongodb.com/cloud/atlas](https://mongodb.com/cloud/atlas)
2. Sign up with email OR GitHub login
3. Click `Create` → Choose FREE tier (M0)
4. **Cluster name:** `prepbridge-cluster`
5. **Cloud provider:** Any (AWS recommended for US)
6. **Region:** Pick one close to you
7. Click `Create Cluster` (takes ~3 mins)

### Step 1.2: Create Database User
1. In Atlas sidebar → `Database Access`
2. Click `Add Database User`
3. **Username:** `prepbridge_user`
4. **Password:** Generate a strong one (copy it!)
5. **Role:** `Built-in Role: Atlas Admin` (for dev/free tier)
6. Click `Add User`

### Step 1.3: Allow Network Access
1. In Atlas sidebar → `Network Access`
2. Click `Add IP Address`
3. For quick setup: `0.0.0.0/0` (allows all)
   - ⚠️ Later: restrict to just Render's IP in production
4. Click `Confirm`

### Step 1.4: Get Connection String
1. In Atlas → Databases → Click `Connect`
2. Choose `Drivers` → `Node.js`
3. Copy the connection string:
   ```
   mongodb+srv://<username>:<password>@<cluster>.mongodb.net/prepbridge?retryWrites=true&w=majority
   ```
4. Replace:
   - `<username>` with `prepbridge_user`
   - `<password>` with the password you generated
5. **Save this as `MONGO_URI`** (needed for backend)

---

## Phase 2: Deploy ML Service on Render 🤖

### Step 2.1: Push Code to GitHub
1. Open terminal in project root:
   ```bash
   git init
   git add .
   git commit -m "Initial commit: PrepBridge project"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/prepbridge.git
   git push -u origin main
   ```
   ⚠️ If you already have a repo, just push the latest changes.

### Step 2.2: Create Render Web Service
1. Go to [render.com](https://render.com)
2. Sign up with GitHub (recommended)
3. Click `New` → `Web Service`
4. Choose `Connect to GitHub` → Select `prepbridge` repo

### Step 2.3: Configure ML Service
Fill in these settings:

| Setting | Value |
|---------|-------|
| **Name** | `prepbridge-ml` |
| **Root Directory** | `ml-service` |
| **Runtime** | `Python 3` |
| **Build Command** | `pip install -r requirements.txt` |
| **Start Command** | `uvicorn main:app --host 0.0.0.0 --port $PORT` |

### Step 2.4: Add Environment Variables
Click `Advanced` → Add Environment Variables:

| Key | Value |
|-----|-------|
| `GROQ_API_KEY` | *Paste your Groq API key* |

⚠️ **Get your Groq API Key:**
- Go to [console.groq.com](https://console.groq.com)
- Create account → `API Keys`
- Generate and copy your key

### Step 2.5: Deploy
1. Click `Create Web Service`
2. Wait for deployment (watch logs)
3. When ready, copy your service URL: `https://prepbridge-ml-xxxx.onrender.com`

### Step 2.6: Verify ML Service
Test health endpoint:
```
GET https://prepbridge-ml-xxxx.onrender.com/health
```
Expected response:
```json
{"status": "ok", "service": "PrepBridge ML Service"}
```

**✅ Save the URL** (you'll need it for backend env vars)

---

## Phase 3: Deploy Backend API on Render 🔌

### Step 3.1: Create Another Render Web Service
1. In Render → `New` → `Web Service`
2. Connect same GitHub repo (`prepbridge`)

### Step 3.2: Configure Backend
| Setting | Value |
|---------|-------|
| **Name** | `prepbridge-backend` |
| **Root Directory** | `Backend` |
| **Runtime** | `Node` |
| **Build Command** | `npm install` |
| **Start Command** | `npm start` |

### Step 3.3: Add Environment Variables
Click `Advanced` → Add these variables:

**`REQUIRED` - Must Have:**

| Key | Value |
|-----|-------|
| `NODE_ENV` | `production` |
| `MONGO_URI` | *Your MongoDB connection string from Phase 1.4* |
| `JWT_SECRET` | *Generate random: run* `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"` |
| `JWT_EXPIRE` | `7d` |
| `SESSION_SECRET` | *Generate random (use same method as JWT_SECRET)* |
| `ML_SERVICE_URL` | `https://prepbridge-ml-xxxx.onrender.com` |

**`CORS/Frontend` - Add After Frontend Deployed:**

| Key | Value |
|-----|-------|
| `FRONTEND_URL` | (leave blank for now, add after step 4) |
| `FRONTEND_URLS` | (leave blank for now, add after step 4) |

**`OPTIONAL - Email (if you want email features):`**

| Key | Value |
|-----|-------|
| `SMTP_HOST` | your-smtp.com |
| `SMTP_PORT` | 587 |
| `SMTP_USER` | your-email@gmail.com |
| `SMTP_PASS` | your-app-password |
| `FROM_EMAIL` | noreply@prepbridge.com |

**`OPTIONAL - OAuth (if you use Google/GitHub login):`**

| Key | Value |
|-----|-------|
| `GOOGLE_CLIENT_ID` | *from Google Cloud Console* |
| `GOOGLE_CLIENT_SECRET` | *from Google Cloud Console* |
| `GOOGLE_CALLBACK_URL` | `https://prepbridge-backend-xxxx.onrender.com/api/auth/google/callback` |
| `GITHUB_CLIENT_ID` | *from GitHub Settings* |
| `GITHUB_CLIENT_SECRET` | *from GitHub Settings* |
| `GITHUB_CALLBACK_URL` | `https://prepbridge-backend-xxxx.onrender.com/api/auth/github/callback` |

### Step 3.4: Deploy Backend
1. Click `Create Web Service`
2. Wait for build & deployment
3. Copy your backend URL: `https://prepbridge-backend-xxxx.onrender.com`

### Step 3.5: Verify Backend Health
Test:
```
GET https://prepbridge-backend-xxxx.onrender.com/api/health
```

**✅ Save the backend URL** (needed for frontend)

---

## Phase 4: Deploy Frontend on Vercel 🎨

### Step 4.1: Create Vercel Project
1. Go to [vercel.com](https://vercel.com)
2. Sign up with GitHub
3. Click `Add New` → `Project`
4. Import `prepbridge` repository

### Step 4.2: Configure Project
| Setting | Value |
|---------|-------|
| **Project Name** | `prepbridge` |
| **Root Directory** | `Frontend` |
| **Framework Preset** | `Vite` |
| **Build Command** | `npm run build` |
| **Install Command** | `npm install` |
| **Output Directory** | `dist` |

### Step 4.3: Add Environment Variable
Click `Environment Variables`:

| Key | Value |
|-----|-------|
| `VITE_API_BASE_URL` | `https://prepbridge-backend-xxxx.onrender.com/api` |

*(Replace with your actual backend URL from Phase 3)*

### Step 4.4: Deploy
1. Click `Deploy`
2. Wait for build completion
3. Vercel will give you a URL: `https://prepbridge-xxxx.vercel.app`

### Step 4.5: Test Frontend
1. Visit your Vercel URL
2. Test login page (should show Google & GitHub buttons)
3. Try navigating different pages

**✅ Save the frontend URL** (needed for backend update)

---

## Phase 5: Final Configuration 🔧

### Step 5.1: Update Backend CORS Variables (IMPORTANT!)

Now that Vercel has assigned your frontend URL, update backend:

1. Go to Render → `prepbridge-backend` service
2. Click `Environment`
3. Update these:

| Key | Value |
|-----|-------|
| `FRONTEND_URL` | `https://prepbridge-xxxx.vercel.app` |
| `FRONTEND_URLS` | `https://prepbridge-xxxx.vercel.app` |

4. Click `Save Changes`
5. Service will auto-redeploy

### Step 5.2: Update OAuth Redirect URLs (if using OAuth)

If you set up Google/GitHub OAuth:

**Google:**
1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Find your OAuth app
3. Update Authorized Redirect URIs to:
   - `https://prepbridge-backend-xxxx.onrender.com/api/auth/google/callback`

**GitHub:**
1. Go to [GitHub Settings → Developer settings](https://github.com/settings/developers)
2. Find your OAuth app
3. Update Authorization callback URL to:
   - `https://prepbridge-backend-xxxx.onrender.com/api/auth/github/callback`

---

## ✅ Complete Verification Checklist

### Health Checks
- [ ] `GET https://prepbridge-backend-xxxx.onrender.com/api/health` returns `{"status": "ok"}`
- [ ] `GET https://prepbridge-ml-xxxx.onrender.com/health` returns success
- [ ] Frontend loads without CORS errors (check browser console)

### API Connectivity
- [ ] Frontend can fetch data without CORS errors
- [ ] Socket.IO connections work (community/realtime pages)
- [ ] File uploads work (resume, etc.)

### Authentication
- [ ] Can register new user
- [ ] JWT login works
- [ ] Google/GitHub login redirects to provider (if configured)
- [ ] JWT token persists in localStorage

### ML Service
- [ ] Backend can reach ML service
- [ ] AI analysis endpoints work (`/api/roadmap/generate`, etc.)

### Database
- [ ] User data saves to MongoDB
- [ ] Can fetch user profile
- [ ] Admin features work

---

## 🛠️ Troubleshooting Guide

### Issue: CORS Error on Frontend
```
Access to XMLHttpRequest blocked by CORS policy
```
**Fix:**
1. Check backend env vars: `FRONTEND_URL` and `FRONTEND_URLS` are set correctly
2. Redeploy backend after updating
3. Wait 2 mins for changes to propagate

### Issue: OAuth Fails / Redirects to Localhost
```
Redirect URI mismatch
```
**Fix:**
1. Frontend: Check `LoginPage.jsx` is using dynamic backend URL (should be fixed already)
2. Backend: Verify env vars `GOOGLE_CALLBACK_URL`, `GITHUB_CALLBACK_URL` point to Render URL
3. Provider Console: Update redirect URLs in Google/GitHub

### Issue: Backend Can't Connect to ML Service
```
Connection refused to ML_SERVICE_URL
```
**Fix:**
1. Test ML service health: `curl https://prepbridge-ml-xxxx.onrender.com/health`
2. Check backend env: `ML_SERVICE_URL` is exactly correct
3. Check ML service logs in Render for errors

### Issue: Render Service Sleeps (Free Plan)
```
First request slow (30+ seconds)
```
**This is normal!** Free tier services spin down after 15 mins of inactivity.
- **Solution:** Upgrade to paid Render plan ($7/month) for always-on service
- Or: Set up a simple uptime monitor that pings your services every 10 mins

### Issue: MongoDB Connection Timeout
```
MongooseServerSelectionError: connection timed out
```
**Fix:**
1. Check MongoDB Atlas Network Access allows `0.0.0.0/0`
2. Verify `MONGO_URI` is correct (check for copy/paste errors)
3. Ensure username/password match what you created

### Issue: Can't `npm start` in Backend
```
Cannot find module 'express'
```
**Fix:**
1. Render should auto-run `npm install`
2. Check package.json exists in `Backend` folder
3. Try rebuilding: In Render service → `Manual Deploy`

---

## 📊 Cost Summary

| Service | Free Tier | Cost |
|---------|-----------|------|
| **Vercel** | Unlimited static deployments | **$0/month** |
| **Render** ML Service | 750 hours/month | **$0/month** |
| **Render** Backend | 750 hours/month | **$0/month** |
| **MongoDB Atlas** | 5GB storage, shared cluster | **$0/month** |
| **Groq API** | 25 requests/day free | **$0/month** |
| **TOTAL** | | **$0/month** ✅ |

⚠️ **Important Notes:**
- Render free tier services sleep after 15 mins of inactivity
- First request after sleep = ~30 second cold start (normal)
- Upgrade to paid Render plan ($7/month) if you need always-on
- Keep API keys secure - never commit them to code

---

## 🎯 Quick Reference URLs (Fill After Deployment)

```
Frontend:     https://________________.vercel.app
Backend:      https://________________.onrender.com
ML Service:   https://________________.onrender.com
Health Check: https://________________.onrender.com/api/health
```

---

## ✨ You're All Set!

Once you complete all 5 phases + verification, your PrepBridge app will be **live and accessible globally** 🎉

**Next Steps:**
1. Follow the deployment order exactly
2. Save all your URLs and API keys
3. Test each phase before moving to the next
4. Check troubleshooting if anything breaks

**Questions?** Check Render/Vercel logs for specific error messages.

Good luck! 🚀
