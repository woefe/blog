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

---
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">Woefe's Blog</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="https://woefe.com" property="cc:attributionName" rel="cc:attributionURL">Wolfgang Popp</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
