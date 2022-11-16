# Rodrigo Machuca Personal Site

[![Netlify Status](https://api.netlify.com/api/v1/badges/f09f53df-1845-4497-be07-776ea925a57d/deploy-status)](https://app.netlify.com/sites/rmachuca-me/deploys)

This is my personal site built with Hugo and deployed in Netlify:
https://rmachuca.me

## Common Operations

### Update Theme

[hugo-coder](https://github.com/luizdepra/hugo-coder) theme is installed as a `git submodule`; hence, to update it use the following command:

```sh
git submodule foreach git pull origin main
```

### Run Local Hugo Server

``` sh
hugo server
```
