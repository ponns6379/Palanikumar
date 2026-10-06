# Upload to GitHub → deploy on Vercel

Vercel has no writable disk and no embedded database, so you need two free external services:
**Supabase** (PostgreSQL database + file storage). Total time: about 15 minutes.

## 1. Put the code on GitHub
1. Create an empty repository on github.com (e.g. `palanikumar-site`, no README/licence). Private is fine.
2. In this folder run:
```bash
git init -b main          # skip if a .git folder already exists
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<your-username>/palanikumar-site.git
git push -u origin main
```
`.env` files, `node_modules`, `.next` and uploads are git-ignored — secrets never reach GitHub.

## 2. Create the Supabase project
1. supabase.com → **New project** (choose a region near Chennai, e.g. Mumbai/Singapore). Save the database password.
2. **Database URL:** Project → *Connect* → *Transaction pooler* (port 6543) → copy the URI and put your password in it.
   Looks like `postgresql://postgres.xxxx:PASSWORD@aws-0-ap-south-1.pooler.supabase.com:6543/postgres`
3. **File storage:** Storage → *New bucket* → name `site-uploads` → tick **Public bucket**.
4. **Keys:** Project Settings → API → copy the *Project URL* and the **service_role** key (keep it secret; server only).

## 3. Import into Vercel
1. vercel.com → **Add New → Project** → import your GitHub repo (Framework: Next.js, auto-detected).
2. Add these **Environment Variables** before deploying:

| Name | Value |
|---|---|
| `AUTH_SECRET` | long random text, 32+ chars (`openssl rand -hex 32`) |
| `DATABASE_URL` | the Supabase pooler URI from step 2 |
| `SUPABASE_URL` | the Supabase Project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | the service_role key |
| `SUPABASE_BUCKET` | `site-uploads` |
| `NEXT_PUBLIC_SITE_URL` | your final address, e.g. `https://your-project.vercel.app` (change when you add a custom domain) |

3. Click **Deploy**. On the first visit the tables are created and filled with the biodata content automatically.

## 4. Create the admin login (once, from your computer)
```bash
npm install
DATABASE_URL='<same Supabase URL>' ADMIN_EMAIL=you@example.com ADMIN_NAME='Your Name' ADMIN_PASSWORD='a-long-password-12+' npm run create-admin
```
Then open `https://<your-site>/admin` and sign in.

## 5. After going live
- Admin → *Settings & CV*: set the public site URL, check the CV download.
- Custom domain: Vercel → Settings → Domains, then update `NEXT_PUBLIC_SITE_URL` and redeploy.
- Every `git push` to `main` redeploys automatically.

## Limits on Vercel
- Uploads are capped at **4 MB** per file (Vercel's request limit). Compress larger images before uploading.
- Uploads fail with a clear message if the Supabase variables are missing.
