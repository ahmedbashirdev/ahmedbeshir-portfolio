# Deploying ahmedbeshir.com

This folder is the source for `https://ahmedbeshir.com` — a static site hosted on Cloudflare Pages.

## Files

- `index.html` — the portfolio
- `Ahmed_Beshir_Resume.pdf` — CV (PDF — primary download)
- `Ahmed_Beshir_Resume.docx` — CV (Word — editable)
- `images/` — project preview SVGs and headshot

## Workflow after git is set up

```bash
# Edit any file, then:
git add .
git commit -m "describe the change"
git push
```

Cloudflare Pages auto-deploys from the `main` branch in ~30s.

## First-time deploy steps

See the prompt I gave Claude Code — it sets up `git init`, creates the GitHub repo, pushes, and tells you what to click next on Cloudflare to connect the repo.
