# alnah.io

[![Hugo](https://img.shields.io/badge/Hugo-0.140+-ff4088?logo=hugo&logoColor=white)](https://gohugo.io/)
[![Deploy](https://img.shields.io/github/actions/workflow/status/alnah/alnah.github.io/hugo.yaml?branch=main&label=deploy)](https://github.com/alnah/alnah.github.io/actions)
[![License](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

> Technical blog about building reliable tools, and systems in Go.

## Languages

Content is available in three languages:

- English (`/en/`)
- Français (`/fr/`)
- Português Brasil (`/pt-br/`)

## Local Development

```bash
# Clone with submodules (theme)
git clone --recurse-submodules https://github.com/alnah/alnah.github.io.git
cd alnah.github.io/blog

# Run local server
hugo server

# Run with future-dated posts
hugo server --buildFuture
```

Requires [Hugo Extended](https://gohugo.io/installation/) v0.140+.

## Structure

```text
alnah.github.io/
├── .github/workflows/    # GitHub Actions deployment
├── blog/
│   ├── content/
│   │   ├── en/           # English content
│   │   ├── fr/           # French content
│   │   └── pt-br/        # Portuguese content
│   ├── assets/           # SCSS, images
│   ├── layouts/          # Template overrides
│   └── hugo.yaml         # Site configuration
└── docs/                 # Additional documentation
```

## Deployment

Automatic deployment to GitHub Pages on push to `main`. The workflow:

1. Builds the Hugo site with `--gc --minify`
2. Deploys to [alnah.io](https://alnah.io)

## Theme

Uses [Hugo Stack](https://github.com/CaiJimmy/hugo-theme-stack) as a git submodule.

```bash
# Update theme
git submodule update --remote themes/stack
```

## License

Content is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
