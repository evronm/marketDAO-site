# MarketDAO Website

Website for MarketDAO project, built with Jekyll and hosted on GitHub Pages.

## Security Status

MarketDAO has been **audited by Hashlock Pty Ltd** (January 2026). All identified issues have been resolved.

### Key Security Features

- ✅ **Governance token locking**: Tokens locked during proposal support and voting (H-03/H-04 fix)
- ✅ **Distribution token locking**: Tokens locked during distribution registration (H-02 fix)
- ✅ **Operator voting restrictions**: All transfers checked against election end (H-05 fix)
- ✅ **Pro-rata distributions**: Proportional calculations prevent pool exhaustion (M-01 fix)
- ✅ **Reentrancy protection**: ReentrancyGuard on all transfer functions
- ✅ **O(1) scalability**: Supports 10,000+ holders with constant gas costs

## Local Development

1. **Install Dependencies**
   ```
   bundle install
   ```

2. **Run Development Server**
   ```
   bundle exec jekyll serve
   ```
   
3. **View Site**
   Open your browser to http://localhost:4000/marketDAO-site/

## Deployment

This site is automatically deployed to GitHub Pages when changes are pushed to the main branch.

## Project Structure

- `_layouts/` - HTML templates
- `_includes/` - Reusable components
- `assets/` - CSS, JavaScript, and images
- `docs/` - Documentation pages
  - `user-guide.md` - User-facing documentation
  - `technical-reference.md` - Developer documentation
- `index.md` - Homepage
- `_config.yml` - Jekyll configuration

## About MarketDAO

MarketDAO introduces tradable voting tokens for each election, creating a dynamic governance system where voting power can be bought and sold based on the strength of preferences.

### Main Repository

The smart contracts and frontend code are in the main repository: [https://github.com/evronm/marketDAO](https://github.com/evronm/marketDAO)
