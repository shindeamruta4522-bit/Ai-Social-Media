# Deploy to Vercel

Deploy this repository as two Vercel projects. The frontend is a Vite single-page app and the backend is a FastAPI Vercel Function.

## 1. Deploy the API

1. In Vercel, import `shindeamruta4522-bit/Ai-Social-Media` as a new project.
2. Set **Root Directory** to `backend`.
3. Select the **FastAPI** framework preset, leave the build and output settings at their defaults, and deploy.
4. Add the production environment variables from `backend/.env.example`, replacing every `your-...` placeholder with the deployed Vercel URLs.
5. Redeploy after adding the variables.

Use managed, internet-accessible services for `MONGO_URI` and `REDIS_URL`; localhost services cannot be reached from Vercel. Configure the OAuth callback URLs with the backend project's production domain:

```text
https://<backend-project>.vercel.app/api/providers/youtube/callback
https://<backend-project>.vercel.app/api/providers/instagram/callback
```

## 2. Deploy the frontend

1. Import the same repository again as a second Vercel project.
2. Set **Root Directory** to `frontend`.
3. Select the **Vite** framework preset.
4. Add the variables in `frontend/.env.example` for Production and Preview.
5. Set `VITE_API_BASE_URL` to `https://<backend-project>.vercel.app/api`.
6. Deploy or redeploy the frontend.

The `frontend/vercel.json` rewrite keeps direct visits to routes such as `/dashboard` and `/reports` inside the React application.

## 3. Complete the service configuration

After the frontend URL is known, set these backend production variables and redeploy the API:

```text
FRONTEND_URL=https://<frontend-project>.vercel.app
CORS_ORIGINS=https://<frontend-project>.vercel.app
BACKEND_URL=https://<backend-project>.vercel.app
```

Add the frontend Vercel domain to Firebase Authentication's Authorized Domains list. Add the two backend callback URLs above to Google Cloud and Meta, respectively.

## 4. Verify

```text
https://<backend-project>.vercel.app/api/health
https://<frontend-project>.vercel.app
```

## Storage note

Vercel Functions can write only to temporary `/tmp` storage. This repo now uses `/tmp/ai-social-media` automatically on Vercel, so the API can start and generate reports. Avatar uploads are not durable between function instances; use an object store such as Vercel Blob, S3, or Cloudflare R2 before treating uploads as production-persistent.
