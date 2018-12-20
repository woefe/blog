# Sources for woefe.com

Sources for my blog at [woefe.com](https://woefe.com).

## First time building the web site

```bash
stack init
stack build
stack exec site build
```

## Developing

```bash
# Necessary after changes to site.hs
stack build

# Rebuild the site
stack exec site rebuild

# Start local development server
stack exec site watch
```
