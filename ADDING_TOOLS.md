# Adding a New Tool to limitbreak.no

This document explains the **only file you need to edit** when adding a new tool.
The landing page is driven entirely by `tools.json` — the HTML and CSS never change.

---

## The Single Source of Truth: `tools.json`

Every tool on the page is defined as an object in `tools.json`.

### Example entry (live tool)

```json
{
  "id": "pulse",
  "name": "Pulse",
  "tagline": "Social media attribution tracker",
  "description": "Track, score, and attribute social media posts across platforms. Built for HEROIC esports.",
  "url": "https://pulse.limitbreak.no",
  "status": "live",
  "color": "#c8ff32",
  "icon": "📡"
}
```

### Example entry (coming soon)

```json
{
  "id": "my-new-tool",
  "name": "Coming Soon",
  "tagline": "New tool in development",
  "description": "Something new is being built. Stay tuned.",
  "url": null,
  "status": "coming-soon",
  "color": "#ffffff",
  "icon": "⚡"
}
```

---

## Field Reference

| Field         | Type              | Required | Description |
|---------------|-------------------|----------|-------------|
| `id`          | string            | ✅        | Unique slug. Use kebab-case (e.g. `"my-tool"`). Never reuse an ID. |
| `name`        | string            | ✅        | Display name shown on the card. |
| `tagline`     | string            | ✅        | Short subtitle under the name (1 line). |
| `description` | string            | ✅        | 1–2 sentence description of what the tool does. |
| `url`         | string \| null    | ✅        | Full URL including `https://`. Set to `null` if not live yet. |
| `status`      | `"live"` \| `"coming-soon"` | ✅ | Controls badge, click behaviour, and opacity. |
| `color`       | hex string        | ✅        | Accent color for the top gradient line on the card. Pick a distinct color per tool. |
| `icon`        | emoji             | ✅        | Single emoji shown in the card header. |

---

## Step-by-Step: Adding a New Tool

### When the tool goes live

1. Open `tools.json`
2. Add a new object to the array (anywhere — order = display order)
3. Set `"status": "live"` and provide the real `"url"`
4. Remove or update the corresponding placeholder `"coming-soon"` entry (match by `id`)
5. Commit & push — Railway auto-deploys

```bash
# Example commit message
git add tools.json
git commit -m "feat: add <tool-name> to limitbreak.no"
git push
```

### When you want to tease a tool before it's ready

1. Open `tools.json`
2. Add a new object with `"status": "coming-soon"` and `"url": null`
3. Commit & push

**That's it. Never edit `index.html` or `assets/style.css` just to add a tool.**

---

## Rules to Preserve Existing Tools

- **Never delete an object** from `tools.json` unless you want to remove it from the page
- **Never reuse IDs** — if you remove a tool and add it back, it's fine to reuse the ID, but don't accidentally duplicate it
- Tools are rendered in array order — put important/live tools first
- The landing page fetches `tools.json` fresh on each load, so changes are instant after deploy

---

## Deployment

This site deploys automatically via Railway when you push to `main`.

- **Repo:** github.com/heroic/limitbreak (or wherever it's hosted)
- **Railway project:** limitbreak-no
- **Domain:** limitbreak.no → configured in Railway → Settings → Custom Domains

Static files are served by nginx (see `nginx.conf`). No build step required.

---

## Colors Used by Existing Tools

Avoid reusing these to keep tools visually distinct:

| Tool    | Color     |
|---------|-----------|
| Pulse   | `#c8ff32` |

Pick something from this palette for new tools:
`#7ec4ff` `#ff7e7e` `#c4a8ff` `#5cffca` `#ffd27e` `#ff9ef5`
