+++
date = '2025-03-30T16:09:56+05:30'
draft = false
title = 'Free Website'
+++
## Hosting Free Website With Github And Hugo

- Create a github repository with your username as `psukalka.github.io`. Github only allows one website per username (it can have multiple pages though). So, you need to user your username in the repository.

- Clone the repository on local system

- Install hugo on your system

- Mark this repository as a hugo website using: `hugo new site . --force`

- Add some theme. Ex: `git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke`

- Add these theme in `hugo.toml` file too. `theme = 'ananke'`

- Create some post. `hugo new content content/posts/free-website.md`

- Add some content in the post

- Run the server (with draft files): `hugo server -D`