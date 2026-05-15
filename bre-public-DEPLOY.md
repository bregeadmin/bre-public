# Deploy to Vercel — first-time setup

Step-by-step. Should take ~5 minutes.

## Prerequisites

- [ ] GitHub account
- [ ] Vercel account (free tier — sign up with GitHub at https://vercel.com/signup)
- [ ] Git installed locally (`git --version` to check)

## Step 1 — Create the GitHub repo

1. Go to https://github.com/new
2. Name: `bre-public`
3. Description: `bre.ge — public website`
4. **Private** (you can switch to public later if you want)
5. **Don't** check "Add a README" or ".gitignore" — we already have them
6. Click **Create repository**

GitHub will show a page with commands. Note the SSH URL like `git@github.com:yourusername/bre-public.git` or HTTPS `https://github.com/yourusername/bre-public.git`.

## Step 2 — Push the files

Open Terminal in the folder where you unzipped `bre-public/`:

```bash
cd path/to/bre-public

# initialise git
git init
git add .
git commit -m "initial: home + guide landing + eat-drink category"

# connect to GitHub
git branch -M main
git remote add origin https://github.com/yourusername/bre-public.git
git push -u origin main
```

Refresh the GitHub page — files should appear.

## Step 3 — Connect Vercel

1. Go to https://vercel.com/new
2. Click **Import** next to `bre-public` (Vercel will list your GitHub repos)
3. **Framework Preset**: leave as **Other** (it's vanilla HTML)
4. **Root Directory**: leave blank (defaults to repo root)
5. Click **Deploy**

Wait ~30 seconds. Vercel will give you a URL like `bre-public-yourname.vercel.app`.

## Step 4 — Test

Open the URL. You should see:
- `/` → the home page
- `/guide` → the guide landing
- `/guide/eat-drink` → the category page
- `/anything-random` → the 404 page

## Step 5 — Every change after this

```bash
# edit files
git add .
git commit -m "what changed"
git push
```

Vercel auto-deploys in ~20 seconds. You'll get a comment on GitHub with the preview URL for each branch.

## Custom domain (later)

When ready to switch `bre.ge` from Tilda to this:

1. Vercel project → **Settings** → **Domains** → **Add Domain** → enter `bre.ge`
2. Vercel will tell you what DNS records to add at your domain registrar
3. Add the records, wait 5-30 mins for propagation
4. Vercel handles SSL automatically

But for now — keep the dev URL, finalise the design, then switch.

## Troubleshooting

- **Push rejected** → you forgot `git pull --rebase` after editing on GitHub directly. Do that and push again.
- **Vercel build failed** → check the build log. Most common: a file path with capital letters that doesn't match. Make sure all file/folder names match exactly.
- **404 on `/guide`** → `vercel.json` should have `cleanUrls: true`. Verify the file exists at the repo root.
