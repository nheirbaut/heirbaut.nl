# heirbaut.nl

Hugo blog met het [Beautiful Hugo theme](https://github.com/halogenica/beautifulhugo), geïntegreerd als Hugo module en **zonder NodeJS** (zowel lokaal als in CI).

## Prerequisites

- VS Code
- VS Code extension: **Dev Containers**
- Rancher Desktop (Docker-compatible)

## Starten (devcontainer)

In de repo root:

1. Open de folder in VS Code
2. `Dev Containers: Reopen in Container`
3. In de container terminal:

- Start dev server: `hugo server --bind 0.0.0.0 --port 1313 --buildDrafts`
- Open: `http://localhost:1313`

## Theme integratie (Beautiful Hugo)

We gebruiken **Hugo Modules** (Go modules) om Beautiful Hugo te integreren:

- Geen theme fork
- De versie staat vast in `go.mod`; `go.sum` bevat de checksums
- Overrides gebeuren via Hugo’s standaard lookup order in `layouts/`, `assets/`, `static/`

Pinnen:

- Beautiful Hugo is gepind in `go.mod` (`github.com/halogenica/beautifulhugo/v5 v5.2.1`).
- De module wordt geïmporteerd in `config/_default/hugo.toml`.
- Thema-instellingen en auteur/sociale links staan in `config/_default/params.toml`.

De site gebruikt de standaardopmaak en meegeleverde assets van Beautiful Hugo; een npm-build is niet nodig.

## Content

- Posts: `content/posts/`
- Tags: front matter `tags: [...]` (taxonomie `tags`)

Nieuwe post:

- `hugo new content posts/mijn-post.md`

## Build

In de repo root:

- Productie-build: `hugo --minify`
- Output: `public/`

## Checks (HTML/link)

We gebruiken `htmltest` (Go binary, geen Node):

- Run: `htmltest`
- Config: `.htmltest.yml`

## CI/CD (GitHub Actions)

- CI: `.github/workflows/ci.yml`
  - build: `hugo --minify`
  - checks: `htmltest`
- Deploy: `.github/workflows/deploy.yml`
  - triggert na succesvolle CI op `main`
  - deployt `public/` naar GitHub Pages via de officiële Pages Actions

## Update-procedure

### Hugo (devcontainer + CI)

1. Pas de versie aan in:
   - `.devcontainer/Dockerfile` (`HUGO_VERSION`)
   - `.github/workflows/ci.yml` en `.github/workflows/deploy.yml` (`HUGO_VERSION`)
2. Build + checks:
   - `hugo --minify`
   - `htmltest`

### Beautiful Hugo

In de devcontainer, met de gewenste versie in plaats van `v5.2.1`:

- `hugo mod get github.com/halogenica/beautifulhugo/v5@v5.2.1`
- `hugo mod tidy`
- `hugo --minify`
- `htmltest`

## Rollback

- Config/content fout: revert de commit(s) en push naar `main`.
- Deploy stoppen: disable de workflow “Deploy (GitHub Pages)” in GitHub Actions.
- Terug naar vorige site: revert naar een commit waarvan CI groen was (Pages deployt dan opnieuw).

## Troubleshooting

- Module download issues: `hugo mod clean` en daarna opnieuw `hugo --minify`.
- Verkeerde baseURL op Pages: check `baseURL` in `config/_default/hugo.toml` en de deploy build stap (die zet baseURL automatisch naar de Pages URL).
