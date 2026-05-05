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

## One-time setup needed: Calendly

The "Book a 30-min call" button in the contact section currently links to `https://calendly.com/ahmedbeshir/30min` — that URL won't work until you create the account.

1. Sign up free at https://calendly.com (use info@ahmedbeshir.com)
2. Claim handle `ahmedbeshir` → your URL becomes `calendly.com/ahmedbeshir`
3. Create a new event type: **30-Minute Meeting**
   - Custom URL: `30min`
   - Duration: 30 min
   - Connect your Google Calendar so it auto-blocks busy times
   - Set availability: e.g. weekdays 10am–5pm Cairo time
4. The button will work without any code change.

If you want to use a different Calendly handle, edit `index.html` and replace `calendly.com/ahmedbeshir/30min` with your URL, then push.
