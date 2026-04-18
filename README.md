# YAKV Blog

Documentation and blog for YAKV (Yet Another Key-Value store).

## Local Development

1. Clone this repository:
```bash
git clone <your-repo-url>
cd yakv-blog
```

2. Install dependencies:
```bash
bundle install
```

3. Serve the site locally:
```bash
bundle exec jekyll serve
```

4. Open your browser to `http://localhost:4000`

## GitHub Pages

1. Push this repository to GitHub
2. Enable GitHub Pages in repository settings
3. Choose the branch to deploy (usually `main`)
4. Your site will be published at `https://<username>.github.io/yakv-blog`

## Creating Content

### Pages

Create markdown files in the root or `docs/` directory:

```markdown
---
title: Your Page Title
layout: default
nav_order: 1
---

# Your Content Here
```

### Navigation

Control navigation order with front matter:

```yaml
nav_order: 1
parent: Parent Page Name  # For nested pages
```

## Configuration

Edit `_config.yml` to update:
- Site title and description
- Navigation settings
- Search configuration
- GitHub repository link

## License

The theme is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
