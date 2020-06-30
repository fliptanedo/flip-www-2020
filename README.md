# Flip's Personal Website

29 June 2020

Instructions for generating this site. See `README_Academic.md` for George Cushen's README file for the Academic theme.

## Preferred Tools

* Git and GitHub
* Sublime Text 3 

## Download Hugo Theme Academic 

See instructions for [Hugo Theme Academic](https://sourcethemes.com/academic/docs/install-locally/#install-with-git). This guide assumes that you have Hugo installed. The web deploy is the easiest way to do it.

Don't forget to initialize the theme:

```
git submodule update --init --recursive
```

(This is especially true if you get an error when running Hugo along hte lines of `failed to extract shortcode: template for shortcode "alert" not found`.)

## Run Hugo

`hugo server -d` 

Then navigate to the url.

## Set up Sublime

Open the folder in Sublime and save it as a project.

## Set up user

Navigate to `./config/_default/` and modify the toml files there as appropriate.



