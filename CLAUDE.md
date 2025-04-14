# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands
- Validate HTML: `html-validate *.html`
- Format code: `prettier --write "**/*.{html,css,js}"`
- Test: Open in browser and verify mobile responsiveness

## Code Style Guidelines
- **Structure**: Plain HTML/CSS with minimal vanilla JavaScript
- **Mobile-First**: Use responsive design with flexible layouts and media queries
- **Text-Forward**: Prioritize typography, readability, and high contrast
- **CSS**: Use CSS custom properties for theming
- **Naming**: 
  - HTML IDs: camelCase
  - CSS classes: kebab-case
  - JS functions: camelCase
- **JavaScript**: Only when necessary, no frameworks
- **Accessibility**: Follow WCAG guidelines for text contrast and keyboard navigation
- **Performance**: Optimize images, minimize dependencies
- **Cross-Browser**: Test in Chrome, Firefox, Safari, Edge