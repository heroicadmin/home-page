# Limitbreak Infrastructure Guide

How limitbreak.no and its subdomains are set up, deployed, and maintained.

## Architecture Overview

```
limitbreak.no (root)        --> one.com static hosting (File Manager)
pulse.limitbreak.no         --> Railway (successful-adventure project)
hemelaga.limitbreak.no      --> Railway (vibrant-emotion project)
map.limitbreak.no           --> Railway (terrific-perfection project)
```

- **Domain registrar & DNS**: one.com (login: sindre.stien@gmail.com)
- **Root domain hosting**: one.com built-in web hosting (static files)
- **Subdomain hosting**: Railway (auto-deploy from GitHub on push to main)
- **GitHub org**: heroicadmin

## The Home Page (limitbreak.no)

### What it is
A static landing page that showcases all Limitbreak tools. It loads tool data from `tools.json` and renders interactive cards with a Three.js particle background.

### File structure
```
home-page/
  index.html      # Single HTML file with inlined CSS + JS
  tools.json      # Tool definitions (fetched at runtime)
  assets/
    style.css     # Standalone CSS (used for local dev only)
  Dockerfile      # For Railway (not used with one.com hosting)
  nginx.conf      # For Railway (not used with one.com hosting)
  railway.toml    # For Railway (not used with one.com hosting)
```

### Where it's hosted
The root domain `limitbreak.no` is hosted directly on **one.com's built-in web hosting**. Files are served from the web root (what you see in File Manager).

> **Why not Railway?** one.com doesn't support root domain CNAME records, only subdomains. Railway requires a CNAME. So the root domain uses one.com's own hosting, while subdomains use Railway via CNAME.

### How to deploy changes

#### Option 1: Using Claude Code (recommended)

Claude can upload files directly to one.com via the File Manager API. The process:

1. Edit files locally (`index.html`, `tools.json`)
2. Push to GitHub: `git push`
3. Ask Claude to upload to one.com using this workflow:

```
Claude opens the one.com File Manager in Chrome (filemanager.one.com)
Then runs JavaScript to fetch from GitHub API and PUT to one.com API:

GitHub API: https://api.github.com/repos/heroicadmin/home-page/contents/<file>
  Headers: Accept: application/vnd.github.v3.raw

one.com API: https://filemanager.one.com/api/webspace/2/drive/limitbreak.no/data/httpd.www/<file>
  Method: PUT
  Query: ?targetCharset=utf-8
  Body: file content
```

> **Important**: The GitHub repo must be public for the API fetch to work without auth tokens. Make it public before uploading, private after if desired.

> **Important**: The one.com API requires an active File Manager session (cookies). Claude must navigate to filemanager.one.com first so the browser has valid session cookies.

> **Limitation**: The one.com API does NOT support creating subdirectories (MKCOL returns 404). All files must be in the root, or CSS must be inlined into HTML.

#### Option 2: Manual via File Manager

1. Go to https://filemanager.one.com
2. Login with one.com credentials
3. Upload/edit files in the web root

#### Option 3: SFTP

```
Host: limitbreak.no
Port: 22
Username: limitbreak.no
Password: (one.com control panel password)
```

## Adding a New Tool to the Landing Page

Edit `tools.json` and add an entry:

```json
{
  "id": "my-tool",
  "name": "My Tool",
  "tagline": "Short description",
  "description": "1-2 sentence explanation of what it does.",
  "url": "https://mytool.limitbreak.no",
  "status": "live",
  "color": "#hex-color",
  "icon": "emoji"
}
```

Status options: `"live"` (clickable, shows URL) or `"coming-soon"` (greyed out).

Available colors not yet used: `#ff7e7e`, `#c4a8ff`, `#ffd27e`, `#ff9ef5`.

Then deploy the updated files (see deployment section above).

## Deploying a New Subdomain Service on Railway

### 1. Create the project

```bash
# Initialize git repo if needed
git init && git add -A && git commit -m "Initial commit"

# Create GitHub repo under heroicadmin org
gh repo create heroicadmin/<repo-name> --private --source=. --push
```

### 2. Create Railway project

1. Go to https://railway.com/dashboard
2. Click "+ New" > "GitHub Repository"
3. Select `heroicadmin/<repo-name>`
4. Railway auto-detects Dockerfile and deploys

### 3. Add custom domain on Railway

1. Open the service in Railway > Settings > Networking
2. Click "+ Custom Domain"
3. Enter: `<subdomain>.limitbreak.no`
4. Set the port your app listens on
5. Railway shows the required DNS records (CNAME + TXT)

### 4. Add DNS records on one.com

1. Go to https://www.one.com/admin/dns.do > DNS records tab
2. Add **CNAME** record:
   - Hostname: `<subdomain>`
   - Is an alias of: `<hash>.up.railway.app` (from Railway)
3. Add **TXT** record:
   - Hostname: `_railway-verify.<subdomain>`
   - Value: `railway-verify=<hash>` (from Railway)

DNS propagation takes up to 90 minutes.

### 5. Add to the landing page

Add the tool to `tools.json` and deploy (see sections above).

## Existing Railway Projects

| Railway Project | Service | Domain | GitHub Repo |
|----------------|---------|--------|-------------|
| successful-adventure | pulse-social-tracker + Postgres | pulse.limitbreak.no | heroicadmin/pulse-social-tracker |
| vibrant-emotion | hemelaga | hemelaga.limitbreak.no | heroicadmin/hemelaga |
| terrific-perfection | production-map | map.limitbreak.no | heroicadmin/production-map |
| natural-sparkle | home-page (unused) | — | heroicadmin/home-page |

## Existing DNS Records (one.com)

| Type | Name | Value |
|------|------|-------|
| CNAME | hemelaga | tvn9b2pz.up.railway.app |
| CNAME | map | 4uz1idl5.up.railway.app |
| CNAME | pulse | dv1zdge5.up.railway.app |
| CNAME | home | opyp9eus.up.railway.app |
| TXT | _railway-verify.hemelaga | railway-verify=542bd2... |
| TXT | _railway-verify.map | railway-verify=12d05d... |
| TXT | _railway-verify.pulse | railway-verify=a62e7b... |
| TXT | _railway-verify.home | railway-verify=7f6125... |

## Quick Reference

| What | Where |
|------|-------|
| Domain DNS | https://www.one.com/admin/dns.do |
| File Manager | https://filemanager.one.com |
| Railway Dashboard | https://railway.com/dashboard |
| GitHub Org | https://github.com/heroicadmin |
| Home page repo | https://github.com/heroicadmin/home-page |
