# S4RC
Research Interest Group: Sustainable & Secure Scientific Software & Reproducible Computing

## Website

This repository contains the source for the S4RC website, built with Jekyll using the [Ed theme](https://minicomp.github.io/ed/).

The site is automatically built and deployed to GitHub Pages at: https://uvads.github.io/S4RC/

## Local Development

To run the site locally:

1. Install Ruby and Bundler
2. Run `bundle install`
3. Run `bundle exec jekyll serve`
4. Open http://localhost:4000/S4RC/ in your browser

Alternatively run the site locally via Docker image

1. Change into the top level directory of the cloned repo
2. Run `docker run --rm --volume="$PWD:/srv/jekyll" -p 4000:4000 jekyll/jekyll jekyll serve`
3. Open http://localhost:4000/S4RC/ in your browser

## Structure

- `_config.yml` - Site configuration
- `index.md` - Home page
- `_texts/` - Collection of text documents
- `_data/navigation.yml` - Site navigation menu
- `assets/` - Static assets (images, CSS, etc.)
