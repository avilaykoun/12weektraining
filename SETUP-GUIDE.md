# Get the app on your iPhone properly — no Mac needed (~15 min, free)

You'll host the app at a free permanent web address, then install it from Safari.
After this, **updates are instant** (no more re-downloading files) and **your data
survives every update**.

## One-time setup (all on your iPhone)

**1. Create a free GitHub account**
Safari → github.com → Sign up.

**2. Create a repository**
Tap **+** → **New repository** → name it `training` → tick **Public** → Create.

**3. Upload the app files**
In the repo: **Add file → Upload files**, and upload these 6 files from this folder:
- `index.html`
- `manifest.json`
- `sw.js`
- `icon-180.png`, `icon-192.png`, `icon-512.png`

Tap **Commit changes**.

**4. Turn on GitHub Pages**
Repo → **Settings** → **Pages** → under "Branch" choose `main` and `/ (root)` → **Save**.
After ~1 minute your app is live at:

    https://YOUR-USERNAME.github.io/training/

**5. Move your training data across (important!)**
- Open your **current** app (the old home-screen icon) → Tools → Backup → **Export backup** (saves a file).
- Open the **new web address** in Safari → Tools → Backup → **Restore** → pick that file.
- Check your logged sessions are there.

**6. Install it**
Still on the new address in Safari: **Share → Add to Home Screen**.
You'll get the proper icon, full-screen app, and it works offline.
Delete the old home-screen icon once you're happy.

## Updating the app from now on

When I give you a new `index.html`:
1. GitHub → your repo → tap `index.html` → pencil (✏️) icon → select-all, paste the new file's contents → Commit. (Or delete + re-upload the file.)
2. Also edit `sw.js` and bump the `VERSION` line (e.g. `v36` → `v37`).
3. Open the app — it picks up the new build on the next launch or two. **Your data stays.**

## Notes
- Public repo means the code is technically visible at that address, but your
  training DATA never leaves your phone — it lives only in the app's local storage.
  If you'd rather the code weren't public, GitHub Pages on private repos needs a
  paid plan; alternatives like Netlify host private-source sites free.
- Still keep occasional **Export backup** files — that's your off-device safety net.
