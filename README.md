# kylem.art

Source for [kylem.art](https://kylem.art) — a Jekyll site built on the [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) theme.

## Local development

Requires Ruby 3.4+ and Bundler.

```sh
bundle install
bundle exec jekyll serve
```

Then open <http://localhost:4000>.

## Deployment

A GitHub Actions workflow at `.github/workflows/pages-deploy.yml` builds the site with Jekyll and publishes to GitHub Pages on every push to `master`. The Pages source must be set to **GitHub Actions** in repo settings (Settings → Pages → Source).

## Writing posts

Drop a Markdown file into `_posts/` named `YYYY-MM-DD-title.md` with the standard Chirpy front matter. See the [Chirpy wiki](https://github.com/cotes2020/jekyll-theme-chirpy/wiki) for full reference.
