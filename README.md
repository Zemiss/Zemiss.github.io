# zemiss.github.io

A personal academic and growth blog built with Jekyll and deployed on GitHub Pages.

## Content Focus

- Weekly study logs and exam review notes
- Undergraduate research plans and academic roadmaps
- Notes on mathematics, AI, and bioinformatics
- Personal reflections and project ideas

## Tech Stack

- Jekyll 4
- GitHub Pages
- Bootstrap + jQuery theme
- Grunt for LESS and JavaScript asset builds
- Service Worker based offline support

## Project Structure

- `_posts/`: blog posts, grouped by topic
- `_layouts/`: page and post layouts
- `_includes/`: shared partials such as head, nav, footer, and search
- `css/`, `js/`, `img/`: static assets
- `pwa/`: manifest and app icons
- `.github/workflows/`: build and deployment workflow

## Local Development

### Prerequisites

- Ruby 3.1+
- Bundler
- Node.js 24+ for Grunt based asset work

### Run the site

```powershell
cd C:\Users\12445\Desktop\Zemiss.github.io\
bundle install
bundle exec jekyll serve
```

Open [http://127.0.0.1:4000/](http://127.0.0.1:4000/).

### Rebuild front-end assets

```powershell
npm install
npx grunt
```

Use `npx grunt watch` when editing files under `less/` or `js/`.

## Deployment

GitHub Actions builds the site on pushes to `master` and deploys it to GitHub Pages. Pull requests also run a Jekyll build check before merge.

## Notes

- `_site/` is the generated output directory from Jekyll.
- Keep post front matter consistent: `layout`, `title`, `date`, `author`, `header-img`, and `tags`.
- Some older files still contain encoding issues and should be normalized to UTF-8 in a later cleanup pass.

## License

MIT

