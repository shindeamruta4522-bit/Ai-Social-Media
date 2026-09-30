# Deploy on Render

This repository can deploy entirely on Render from `render.yaml`:

- `ai-social-media-frontend-shindeamruta` is a Vite static site.
- `ai-social-media-api-shindeamruta` is the FastAPI API.
- MongoDB Atlas stores application data and a managed Redis-compatible service stores the cache.

The default public URLs are:

```text
https://ai-social-media-frontend-shindeamruta.onrender.com
https://ai-social-media-api-shindeamruta.onrender.com
```

If either Render service name is unavailable, change it in `render.yaml`. Also update `VITE_API_BASE_URL`, `FRONTEND_URL`, `BACKEND_URL`, `CORS_ORIGINS`, and both OAuth callback URLs to match the chosen names.

## 1. Create the MongoDB and Redis services

Create a MongoDB Atlas database and a Redis-compatible cache, then keep their connection strings ready for Render. MongoDB must accept connections from Render, and Redis must support TLS if its URL uses `rediss://`.

## 2. Deploy both services on Render

1. In Render, open **New > Blueprint** and connect the same GitHub repository.
2. Render reads `render.yaml` and creates both services. Keep the frontend static build, API root directory, Python runtime, build command, start command, and `/api/health` check unchanged.
3. During Blueprint setup, enter the requested secrets. Use these production values for the URL-related settings:

   ```text
   FRONTEND_URL=https://ai-social-media-frontend-shindeamruta.onrender.com
   BACKEND_URL=https://ai-social-media-api-shindeamruta.onrender.com
   CORS_ORIGINS=https://ai-social-media-frontend-shindeamruta.onrender.com
   GOOGLE_REDIRECT_URI=https://ai-social-media-api-shindeamruta.onrender.com/api/providers/youtube/callback
   META_REDIRECT_URI=https://ai-social-media-api-shindeamruta.onrender.com/api/providers/instagram/callback
   ```

4. Supply `MONGO_URI`, `REDIS_URL`, the backend Firebase values, OAuth credentials, and X source token from `backend/.env.example`. Set `FIRST_ADMIN_EMAIL` and `ADMIN_EMAILS` to the email address that should administer the app.
5. Supply the `VITE_FIREBASE_*` values from `frontend/.env.example` for the static site. `VITE_API_BASE_URL` is already configured by the Blueprint.
6. Create the Blueprint and wait for the API health check to pass.

## 3. Configure identity providers

- Add `ai-social-media-frontend-shindeamruta.onrender.com` to Firebase Authentication's Authorized Domains.
- Add `GOOGLE_REDIRECT_URI` to the Google OAuth web client.
- Add `META_REDIRECT_URI` to the Meta app's valid OAuth redirect URIs.

## 4. Verify

Open these URLs after Render reports a successful deploy:

```text
https://ai-social-media-api-shindeamruta.onrender.com/api/health
https://ai-social-media-frontend-shindeamruta.onrender.com
```

The health endpoint returns `status: "ok"`. Its `redis.connected` field should be `true` after `REDIS_URL` is configured.

## Production storage note

The Blueprint starts on Render's Free plan for a low-cost first deployment. Free services can restart and have an ephemeral filesystem, so avatar uploads and generated reports can disappear after a restart or redeploy.

For persistent uploads and reports, change the API to a paid Render plan and attach a persistent disk at `/var/data`. The Blueprint already directs `UPLOADS_DIR` and `REPORTS_DIR` to that location. For larger deployments, move user files and reports to object storage instead.

## Vercel alternative

The repository also includes the existing Vercel-only guide in `VERCEL_DEPLOY.md`. That arrangement requires managed MongoDB and Redis and does not provide durable local file storage for uploads or reports.
