---
layout: note
title: Deploy Jekyll with Tailwind to Github Pages with Github Actions
tags: ruby, jekyll, github, github actions
permalink: jekyll-tailwind-github-actions-pages
published_at: 2024-01-30 13:19:00
---

There's an [official guide](https://jekyllrb.com/docs/continuous-integration/github-actions/) but it is pretty basic. 

Go to your repo settings on GitHub. Go to _Pages_ in the _Code and automation_ menu. Select _GitHub Actions_ as _Source_ under _Build and deployment_.

Now go to the _Actions_ tab and start a _New workflow_. Search for _Jekyll_. Click _configure_ for _Jekyll_ (not _GitHub Pages Jekyll_). This will open an editor with a bunch of yml code.

TL;DR I did a few changes and here is the result. This is the final result after many attempts.

```
name: Deploy Jekyll site to Pages

on:
  push:
    branches: ["main"]

  workflow_dispatch:

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build dependencies
        run: npm install

      - name: Setup Ruby
        uses: ruby/setup-ruby@v1.170.0
        with:
          bundler-cache: true
          cache-version: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v4

      - name: Build with Jekyll
        run: bundle exec jekyll build --baseurl "${{ steps.pages.outputs.base_path }}"
        env:
          JEKYLL_ENV: production

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3

  deploy:
    needs: build

    permissions:
      pages: write
      id-token: write

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    runs-on: ubuntu-latest

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

I made the mistake of commiting directly to _main_ insted of creating a new _PR_.

The _workflow_ didn't work.

I decided to create a new branch and fix it. I named it `gh-actions`.

First of all I updated the beginning of the workflow, so it runs also on pull requests (just for testing purposes).

```
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
```

I had to add a step inside the build job.

```
- name: Build dependencies
  run: npm install
```

Also updated the `_config.yml` file to include Tailwind styles:

```
include: ["node_modules/tailwindcss"]
```

I moved the _permissions_ section to the _deploy_ job, as they're not needed for the _build_.

I pushed everything and waited. It failed.

![](/assets/images/gh-actions-branch-error.png)

It took me a while to realize what was going on.

After spending some time navigating through the _Settings_ of the repo, I found out what I needed in _Settings > Code and automation > Environments > github-pages_. Deployment branches can be defined and I wasn't allowing my new `gh-actions` to be deployed. For testing purposes I selected _No restriction_ and tried again. It worked.

![](/assets/images/gb-pages-deployment-branches.png)

Once I tested everything was working, I selected only the `main` branch and removed the `pull_request` from the _workflow_.
