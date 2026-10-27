# Working in this repo (Base44 sandbox)

Static documentation site built with **mkdocs-material**, served in the sandbox by
`docker compose -f docker-compose.base44.yml up -d` from `.base44/Dockerfile`.
The site is live on host port 3000 (container port 8000).

## Non-obvious setup facts

- The dependency list in the image mirrors the repo's `make prepare` target and must keep
  the **hiddify forks** of `mkdocs`, `mkdocs-static-i18n` and `lunr.py`: `mkdocs.yml` uses
  `materialx.emoji`, and the search index uses `languages="fa"`. Installing upstream
  `mkdocs`/`mkdocs-static-i18n` makes the config fail to load.
- The Persian lunr language file is committed at
  `docs/assets/javascripts/lunr/min/lunr.fa.min.js`; the search worker loads it from the
  site's own assets, so don't delete it.
- `mkdocs-rss-plugin` is newer than the mkdocs 1.5.3 from the fork and logs a dependency
  conflict; the site still builds. Its "Unable to read git logs" warnings are expected —
  the build writes to a temp dir outside the git repo.
- `git-authors`, `git-revision-date-localized` and `rss` need git metadata, which is why
  the whole repo (including `.git`) is bind-mounted at `/app`.
- Tailwind output is committed at `docs/assets/css/site.css`. Only re-run
  `make tw` (`npx tailwindcss -i ./input.css -o ./docs/assets/css/site.css`) when
  `input.css` or Tailwind classes in the docs change.
- The `exclude` plugin is toggled by the `DEV` env var (`enabled: !ENV [DEV,false]` in
  `mkdocs.yml`). The compose service leaves `DEV` unset, so `docs/app/**` and
  `docs/manager/**` are still rendered — same as the repo's `make docs`/`make live`.
- No credentials or environment variables are required. Google Analytics, giscus comments
  and the Gurubase "Ask AI" widget all use public client-side IDs committed in
  `mkdocs.yml` / `overrides/main.html`.

## Verifying it works

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:3000/        # 200, English home
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:3000/fa/     # 200, Persian home
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:3000/blog/   # 200
docker compose -f docker-compose.base44.yml ps                          # docs = healthy
```

A full build takes ~35 s and `mkdocs serve` needs ~40 s before it answers, hence the
healthcheck's 120 s start period.

## Editing loop

`mkdocs serve` watches `docs/`, `includes/`, `overrides/` and `mkdocs.yml` over the bind
mount and rebuilds + reloads the browser automatically (verified). `mkdocs.yml` changes are
picked up by the watcher too, but a full `docker compose -f docker-compose.base44.yml
restart docs` is the reliable path after editing it.
