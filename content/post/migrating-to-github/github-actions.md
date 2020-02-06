+++ 
title = "GitHub Actions for GitHub Pages"
draft = false
comments = true 
slug = "" 
tags = ["github", "github-actions"]
categories = []
date = "2020-02-05T00:00:00-00:00"

showpagemeta = true
showcomments = true
+++

### Setting Up Actions

#### Major Caveat!!!

At the time of this writing there was a... [major inconvenience][inconvenience].

Basically, there are some restrictions that prevent Github actions from being able to publish GitHub Pages. To work
around this I had to monkey around different git configurations. The fact that my site uses submodules made it even
more cumbersome. Nonetheless, it worked and here is how I did it.

### Step 1. Configure a deployment key

There are other [multiple options](https://github.com/JamesIves/github-pages-deploy-action#configuration-) for providing
the GitHub action authorization. I chose this route as the most restrictive given the constraint mentioned above.

To setup a deployment key simply [follow the guide](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys) 
to create and install a deployment key.

### Step 2. Setup a workflow

Go to your project and find the **Actions** tab:

{{< figure src="../img/actions-tab.png" style="width: 200px" >}}

Then, click on **Set up a workflow yourself**:

{{< figure src="../img/setup-workflow.png" style="width: 300px" >}}

The workflow I ended up with that works with submodules is the following:

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - name: Checkout submodules
        shell: bash
        run: |

          # NOTE: the deploy key below is per repo so it wouldn't work for the necessary submodules.
          # NOTE: this works because the submodules are public.
          git config --global url."https://github.com/".insteadOf "git@github.com:"

          git submodule sync --recursive
          git submodule update --init --force --recursive --depth=1

          # NOTE: we need to revert this because the "Deploy" step needs to use ssh
          git config --global --unset-all url."https://github.com/".insteadof
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2.4.1
        with:
          hugo-version: '0.62.2'
          extended: true
      - name: Hugo build
        run: hugo
      - name: Install SSH Client
        uses: webfactory/ssh-agent@v0.2.0
        with:
          ssh-private-key: ${{ secrets.DEPLOY_PRIVATE_KEY }}
      - name: Deploy
        uses: JamesIves/github-pages-deploy-action@releases/v3
        with:
          SSH: true
          BRANCH: gh-pages
          FOLDER: public
```

As you can see, my hopeful build and deploy strategy becomes some fancy gymnastics just to get everything
playing well together. The hope is that soon in the future it will be. :pray:

### Step 3. Commit and Push

After a quick commit and push, we can now see the pipeline in action by going back to the **Actions** tab.

{{< figure src="../img/actions-result.png" style="max-width: 100%; width: 700px;" >}}

[inconvenience]: https://github.community/t5/GitHub-Actions/Github-action-not-triggering-gh-pages-upon-push/m-p/26869/highlight/true#M301
