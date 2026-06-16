# Publishing CN Conductor Trainer to GitHub Pages

One-time setup, then the live site auto-updates on every push.

## Prerequisites
- Git installed — https://git-scm.com
- A free GitHub account — https://github.com

## 1. Open a terminal in this folder
In File Explorer, open `C:\projects\CN Conductor Trainer`, right-click → **Git Bash Here** (or **Open in Terminal**).

## 2. Initialize and make the first commit
```
git init
git config user.name "Jordan Doerksen"
git config user.email "doerksen.jordan@gmail.com"
git add .
git commit -m "CN Conductor Trainer — initial commit"
git branch -M main
```

## 3. Create the GitHub repo and push
**Web way:** open https://github.com/new — name it `cn-conductor-trainer` (or anything), set **Public**, and do **NOT** add a README or .gitignore (we already have them) → **Create repository**. Then run (use your username):
```
git remote add origin https://github.com/<your-username>/cn-conductor-trainer.git
git push -u origin main
```
The first push opens a GitHub sign-in — log in to authorize.

**Or, if you have the GitHub CLI (`gh`):**
```
gh repo create cn-conductor-trainer --public --source=. --remote=origin --push
```

## 4. Turn on GitHub Pages
Repo → **Settings** → **Pages** → Source: **Deploy from a branch** → Branch **main**, folder **/ (root)** → **Save**.
After ~1 minute, your site is live at:
```
https://<your-username>.github.io/cn-conductor-trainer/
```
`index.html` (the hub) loads automatically.

## 5. Updating the live site — the "auto-refresh"
Whenever files change, run:
```
git add .
git commit -m "what changed"
git push
```
GitHub Pages redeploys in about a minute and the live URL updates. That's the refresh-on-push you wanted.

## Notes
- `.gitignore` keeps the official CROR PDF out of the public repo — we shouldn't republish the rulebook; it stays on your PC.
- Public repo = free Pages, and there are no secrets in here.
- Preview locally anytime by double-clicking `index.html`.
- We edit files here in Cowork (they save straight into this folder on your PC), so after a change you just run the step-5 commands to push it live.
