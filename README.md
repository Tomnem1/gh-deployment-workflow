# GitHub Pages Deployment Workflow

A GitHub Actions workflow that automatically deploys a static site to GitHub Pages.
Every push to `main` branch that changes `site/index.html` triggers a deploy — nothing else
does. Built for the roadmap.sh ["GitHub Pages Deployment"](https://roadmap.sh/projects/github-actions-deployment-workflow)
project.

## Live site

https://tomnem1.github.io/gh-deployment-workflow/

## Repository structure

```
site/index.html                 # the page that gets published
.github/workflows/deploy.yaml   # the deployment workflow
```

Only the contents of `site/` are published to GitHub Pages — the rest of the repository
(workflow files, etc.) is not served publicly.

## GitHub Pages setup (one-time)

Before this workflow can publish anything, GitHub Pages needs to be told where to
get its content from:

1. Go to the repository's **Settings → Pages**.
2. Under **Build and deployment → Source**, select **GitHub Actions** (not "Deploy
   from a branch").

This is a repository setting, not something the workflow configures itself — it has
to be set once before the first successful deploy.

## How the workflow works

The workflow (`.github/workflows/deploy.yaml`) listens for `push` events on the
`main` branch, and is filtered to run only when `site/index.html` changes.

The workflow is split into two jobs:

**`build`** — packages the site for Pages:
1. [`actions/checkout`](https://github.com/actions/checkout) — checks out the repository onto the runner.
2. [`actions/configure-pages`](https://github.com/actions/configure-pages) — prepares the Pages build configuration for the repository.
3. [`actions/upload-pages-artifact`](https://github.com/actions/upload-pages-artifact) — packages the `site/` folder into a GitHub Pages-compatible artifact and uploads it.

**`deploy`** — publishes the packaged site, and only runs after the `build` job succeeds
(`needs: build`):
- Declares the `pages: write` and `id-token: write` permissions the deploy step needs
  to publish to Pages and authenticate the run.
- Runs in the `github-pages` environment, which surfaces the live URL in the run
  summary and deployment history.
- [`actions/deploy-pages`](https://github.com/actions/deploy-pages) — takes
  the uploaded artifact and publishes it to GitHub Pages.

