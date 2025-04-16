# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands
- Install dependencies: `bundle install`
- Start local dev server: `bundle exec jekyll serve`
- Build site: `bundle exec jekyll build`
- Check for errors: `bundle exec jekyll doctor`

## Code Style Guidelines
- HTML: Use semantic elements, correctly nested tags
- CSS: Use CSS variables (defined in :root), mobile-first design
- JavaScript: ES6+, keep scripts minimal and focused
- Indentation: 2 spaces for all languages
- File Organization: Keep CSS in scss files, JS in separate files
- Naming: Use kebab-case for CSS classes, camelCase for JS variables/functions
- Dark Mode: Support light/dark mode toggle with localStorage persistence

## Jekyll Structure
- Use _layouts for templates
- Use _includes for reusable components
- Use front matter (---) in markdown files for page metadata
- Use {{ site.baseurl }} for internal links
- Use relative_url filter for asset paths