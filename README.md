# Bootcamp Hub

Three self-study bootcamps in one GitHub Pages site — no build step required.

## Bootcamps

| File | Bootcamp | Days |
|------|----------|------|
| `cyber.html` | Cybersecurity (Network+ → Security+ → Linux+ → Python → CySA+ → DevSecOps) | 98 |
| `ibm.html` | IBM Python + Machine Learning (Coursera) | 55 |
| `ai.html` | AI Engineering (full ML/AI engineer path) | 390 (13 months) |

## Deploy to GitHub Pages

1. Create a new GitHub repo (e.g. `bootcamp-hub`)
2. Copy all four files (`index.html`, `cyber.html`, `ibm.html`, `ai.html`) to the repo root
3. Go to **Settings → Pages → Source → Deploy from branch → main / (root)**
4. Your site will be live at `https://<your-username>.github.io/bootcamp-hub/`

## Local preview

Open `index.html` directly in a browser — **or** run a local server to avoid iframe same-origin issues:

```bash
# Python 3
python3 -m http.server 8080
# Then open http://localhost:8080
```

## Tech stack

- React 18.3.1 (UMD CDN — no npm/webpack)
- Babel standalone 7.23.5 (in-browser JSX transpilation)
- Google Fonts (Inter + JetBrains Mono on the hub shell)
- `localStorage` for progress persistence (namespaced per bootcamp: `bc_*`, `ibc_*`, `aicamp_*`)

## Notes

- Progress is saved automatically in your browser's `localStorage`
- Each bootcamp loads lazily — only when you first click its tab
- The cybersecurity bootcamp is ~1.5 MB (it contains all lesson content inline)
