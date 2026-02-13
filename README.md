# Diego's notes

Personal blog

## Build locally

- **Ruby**: 2.7 or newer recommended (e.g. `rbenv install 3.2.0` then `rbenv local 3.2.0`).
- **Install and run**:

  ```bash
  bundle install
  bundle exec jekyll serve
  ```

  Then open <http://localhost:4000>.

## Deploy to GitHub Pages

1. Create a repo (e.g. `dlresende.github.io` for a user site, or any repo with Pages enabled).
2. Push this directory to the repo.
3. In the repo **Settings → Pages**, set source to the default branch (e.g. `main`). GitHub will build the Jekyll site automatically.

## Contents

All articles from the WordPress blog have been migrated to Markdown in `_posts/` with Jekyll front matter (title, date, layout). Permalinks follow the same pattern as before: `/:year/:month/:day/:title/`.
