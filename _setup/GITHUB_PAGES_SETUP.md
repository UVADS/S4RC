# GitHub Pages Setup Instructions

This repository is configured to work with GitHub Pages using the Ed Jekyll theme.

## Enabling GitHub Pages

To enable GitHub Pages for this repository:

1. Go to the repository settings on GitHub
2. Navigate to the "Pages" section (usually under Settings > Pages)
3. Under "Source", select the branch you want to deploy (typically `main` or `master`)
4. Click "Save"

GitHub will automatically build and deploy your Jekyll site.

## Configuration

The site is configured in `_config.yml` with:
- **Site URL**: https://uvads.github.io
- **Base URL**: /S4RC
- **Theme**: Ed theme (via `remote_theme: minicomp/ed`)

## Theme Information

The [Ed theme](https://minicomp.github.io/ed/) is a Jekyll theme designed for minimal editions. It's perfect for:
- Scholarly publications
- Academic projects
- Text-focused websites
- Documentation sites

### Key Features:
- Clean, readable typography
- Minimal design focused on content
- Responsive layout
- Support for scholarly annotations
- Optimized for performance

## Site Structure

- **Home page**: `index.md`
- **Texts collection**: Files in `_texts/` directory
- **Texts index**: `texts.md` 
- **Navigation**: Configured in `_data/navigation.yml`
- **Configuration**: `_config.yml`

## Adding Content

### Adding a New Text

1. Create a new Markdown file in the `_texts/` directory
2. Add front matter at the top:
   ```yaml
   ---
   layout: narrative
   title: Your Title
   author: Your Name
   ---
   ```
3. Write your content in Markdown below the front matter

### Updating Navigation

Edit `_data/navigation.yml` to add or remove menu items.

## Local Development

If you want to preview the site locally:

```bash
# Install dependencies
bundle install

# Serve the site locally
bundle exec jekyll serve

# Open http://localhost:4000/S4RC/ in your browser
```

## Troubleshooting

If the site doesn't build:
1. Check the "Actions" tab in GitHub to see build logs
2. Verify that GitHub Pages is enabled in repository settings
3. Ensure the `_config.yml` file is valid YAML
4. Check that all required files are present and properly formatted

## Additional Resources

- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [Ed Theme Documentation](https://minicomp.github.io/ed/documentation/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
