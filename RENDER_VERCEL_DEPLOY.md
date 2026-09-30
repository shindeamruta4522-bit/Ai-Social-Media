# Deploy to Vercel and Render

This project is ready for a split deployment:

- **Vercel** serves the React/Vite frontend from `frontend`.
- **Render** serves the FastAPI API from `backend` using `render.yaml`.
- **MongoDB Atlas** stores application data and a managed Redis-compatible service stores the cache.

This is the recommended option for this repository. It keeps the API on a long-running web service, which is a better match for OAuth callbacks, reports, and uploads than a serverless function.

## 1. Choose the public service names

The Render Blueprint creates `ai-social-media-api-shindeamruta`, whose default URL is:

```text
https://ai-social-media-api-shindeamruta.onrender.com
```

If that name is unavailable, change `name` in `render.yaml` before deploying and use the matching `https://<name>.onrender.com` URL in every setting below.

## 2. Deploy the frontend to Vercel

1. In Vercel, import `shindeamruta4522-bit/Ai-Social-Media`.
2. Set **Root Directory** to `frontend` and select the **Vite** preset.
3. Add every value from `frontend/.env.example` to Vercel's Production environment.
4. Set `VITE_API_BASE_URL` to:

   ```text
   https://ai-social-media-api-shindeamruta.onrender.com/api
   ```

5. Deploy. Copy the resulting frontend URL, for example `https://ai-social-media-frontend.vercel.app`.

`frontend/vercel.json` already keeps React routes such as `/dashboard` and `/reports` working when opened directly.

## 3. Create the MongoDB and Redis services

Create a MongoDB Atlas database and a Redis-compatible cache, then keep their connection strings ready for Render. MongoDB must accept connections from Render, and Redis must support TLS if its URL uses `rediss://`.

## 4. Deploy the API to Render

1. In Render, open **New > Blueprint** and connect the same GitHub repository.
2. Render reads `render.yaml` automatically. Keep the `backend` root directory, Python runtime, build command, start command, and `/api/health` check unchanged.
3. During Blueprint setup, enter the requested secrets. Use these production values for the URL-related settings:

   ```text
   FRONTEND_URL=https://<your-frontend>.vercel.app
   BACKEND_URL=https://ai-social-media-api-shindeamruta.onrender.com
   CORS_ORIGINS=https://<your-frontend>.vercel.app
   GOOGLE_REDIRECT_URI=https://ai-social-media-api-shindeamruta.onrender.com/api/providers/youtube/callback
   META_REDIRECT_URI=https://ai-social-media-api-shindeamruta.onrender.com/api/providers/instagram/callback
   ```

4. Supply `MONGO_URI`, `REDIS_URL`, the Firebase values, OAuth credentials, and X source token from `backend/.env.example`. Set `FIRST_ADMIN_EMAIL` and `ADMIN_EMAILS` to the email address that should administer the app.
5. Create the Blueprint and wait for the health check to pass.

## 5. Configure identity providers

- Add the Vercel frontend hostname to Firebase Authentication's Authorized Domains.
- Add `GOOGLE_REDIRECT_URI` to the Google OAuth web client.
- Add `META_REDIRECT_URI` to the Meta app's valid OAuth redirect URIs.

## 6. Verify

Open these URLs after Render reports a successful deploy:

```text
https://ai-social-media-api-shindeamruta.onrender.com/api/health
https://<your-frontend>.vercel.app
```

The health endpoint returns `status: "ok"`. Its `redis.connected` field should be `true` after `REDIS_URL` is configured.

## Production storage note

The Blueprint starts on Render's Free plan for a low-cost first deployment. Free services can restart and have an ephemeral filesystem, so avatar uploads and generated reports can disappear after a restart or redeploy.

For persistent uploads and reports, change the API to a paid Render plan and attach a persistent disk at `/var/data`. The Blueprint already directs `UPLOADS_DIR` and `REPORTS_DIR` to that location. For larger deployments, move user files and reports to object storage instead.

## All-Vercel alternative

The repository also includes the existing Vercel-only guide in `VERCEL_DEPLOY.md`. That arrangement requires managed MongoDB and Redis and does not provide durable local file storage for uploads or reports.
