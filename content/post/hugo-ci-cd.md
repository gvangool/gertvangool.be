+++
comments = false
date = "2016-01-26T16:44:02+01:00"
image = "/uploads/post/gohugo-io-header.png"
menu = ""
share = false
slug = "hugo-ci-cd"
tags = ["hugo", "ci", "cd"]
title = "Continuous deployment with Hugo"

+++

For better automation (and easier updating), I've added [CircleCI](https://circleci.com/)
for automated building and testing of the website.
It will automatically deploy any pushes to the master into a [separate git
repository](https://github.com/gvangool/gvangool.github.io) (to take
advantage of [Github Pages](https://pages.github.com/)).
<!--more-->

The idea is rather simple:

- Dependencies: we download and install the latest [Hugo](http://gohugo.io)
  version and make it available in `~/bin` which is in `$PATH` (this
  happens in `dependencies > pre`). For syntax highlighting, we need to use
  [Pygments](http://pygments.org/), we'll add a [requirements
  file](https://github.com/gvangool/gertvangool.be/blob/3865bc80d2da9bee08e2dd848a70d5ddfeb2e900/requirements.txt)
  which CircleCI automatically installs in a virtualenv.
- In `checkout > post` we make sure that all submodules (in this case [the
  theme](https://github.com/vjeantet/hugo-theme-casper)) are up to date.
- For actual tests, we just run Hugo in verbose mode. It won't catch all the
  errors, but at least we have a good idea.
- Deployment: Since we just ran our tests, the `public` directory still
  contains the website. So before we clone the Github Pages repository, we
  need to empty that directory. The [deploy
  script](https://github.com/gvangool/gertvangool.be/blob/2402b6baa0fc9ce74916e52a5d8ffe214bc81050/deploy.sh))
  will rebuild the website and commit it to Github.

That comes together in the following `circle.yml` file:

``` yaml
checkout:
  post:
    - git submodule sync
    - git submodule update --init --recursive

machine:
  pre:
    - git config --global user.name "CircleCI"
    - git config --global user.email "circleci@circleci.com"

dependencies:
  pre:
    - wget -O ~/tmp/hugo_0.15_linux_amd64.tar.gz https://github.com/spf13/hugo/releases/download/v0.15/hugo_0.15_linux_amd64.tar.gz
    - cd ~/tmp && tar xzf hugo_0.15_linux_amd64.tar.gz
    - mv ~/tmp/hugo_0.15_linux_amd64/hugo_0.15_linux_amd64 ~/bin/hugo

test:
  override:
    - hugo -v

deployment:
  master:
    branch: master
    commands:
      - rm -rf public
      - git clone git@github.com:gvangool/gvangool.github.io.git public
      - ./deploy.sh
```

Don't forget to add a SSH deployment key into Circle CI's project page
(`Settings > Permissions > SSH Permissions`).
