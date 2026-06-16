<p align="center">
  <img src="banner.svg" alt="Hickory Hill is getting a Little Free Library — a community-funded, community-maintained take a book, leave a book box at the Manor Garden Lane footpath entrance" width="100%">
</p>

# hickoryhilllfl.com

Community fundraising and info site for the Hickory Hill neighborhood Little Free Library
at the Manor Garden Lane footpath entrance.

Live at https://hickoryhilllfl.com.

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire site — one self-contained page (HTML, CSS, and a tiny inline script) |
| `favicon.svg` | Browser-tab icon (SVG, scales from 16×16 up) |
| `banner.svg` | README header banner (SVG, mirrors the site's hero) |
| `.github/workflows/deploy.yml` | Push-to-deploy pipeline |

## Updating the site

Edit `index.html`, commit, push to `main`. The GitHub Actions workflow handles the rest.

### Update donation totals

In `index.html`, find the progress tracker and change the `data-raised` value:

```html
<aside class="progress-tracker" data-raised="0" data-goal="580" data-mid="150">
```

- `data-raised` — current dollars in. Change as donations come in.
- `data-goal` — top of the bar ($580, kit option).
- `data-mid` — milestone marker for the community-built option ($150).

The bar fill height, milestone line position, label placements, and the
percentage-based meta message all derive from those numbers via the inline
script at the bottom of the file.

## Deploy pipeline

`main` → GitHub Actions → S3 sync → CloudFront invalidation. Typical end-to-end
time: under a minute.

Auth uses GitHub OIDC — no long-lived AWS keys live in the repo. The workflow
assumes an IAM role scoped to this repo + `main` branch only.

## AWS infrastructure

All in account `205074708100`, region `us-east-1`.

| Resource | Identifier |
|---|---|
| S3 bucket (private, OAC-only) | `hickoryhilllfl.com` |
| CloudFront distribution | `E2S44AR4ROLO4H` (`dwhcubn60hsct.cloudfront.net`) |
| Origin Access Control | `E4L772NVW2NP9` |
| ACM cert (apex + www) | `arn:aws:acm:us-east-1:205074708100:certificate/ffc8dd1d-4053-4193-a945-518a1b7c9a46` |
| Route 53 hosted zone | `Z062250032CJ4XC268APT` |
| GitHub Actions deploy role | `arn:aws:iam::205074708100:role/hickoryhilllfl-github-deploy` |
| GitHub repo variable | `CLOUDFRONT_DISTRIBUTION_ID = E2S44AR4ROLO4H` |

DNS is on Route 53 (Namecheap nameservers point at AWS). Apex and `www` both
resolve to the CloudFront distribution via A/AAAA aliases.

## Local preview

No build step. Just open `index.html` in a browser, or run a static server:

```sh
python3 -m http.server 8000
# then visit http://localhost:8000/
```

## Workflow conventions

- All work happens on a branch in a git worktree under `.claude/worktrees/`;
  never commit directly to `main`.
- The repo is set to auto-delete branches when their PR merges, so `main` should
  stay as the only branch.
- Commits are signed off (`git commit -s`).
