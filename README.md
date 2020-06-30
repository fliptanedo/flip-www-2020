# Flip's Personal Website (2020)

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

Open the folder in Sublime and save it as a project. Go ahead and check that Git Saavy works by saving a new `README.md` file and pushing it back to the GitHub repository.

## Set up user

Navigate to `./config/_default/` and modify the toml files there as appropriate. Check to make sure everything compiles. 

## Transfer Assets from Previous Version

All paths are relative to the project root. 

1. Create a `./layouts/` folder. Hugo will look here for templates and shortcodes first. This over-rides any files from the theme (located in `.themes/academic`).

2. Transfer over the `./layouts/shortcodes/` directory. These include a shortcode for twitter and a shortcode for e-mail hyperlinks that are robust against bots looking for e-mail addresses to spam.

3. Create a `./layouts/partials/` directory for templates for the home page.

4. Unlike earlier versions of Hugo Academic, CSS and javascript now go into the `./assets` folder and are called in `params.toml`. See [this discussion](https://github.com/gcushen/hugo-academic/issues/867). 

   The `plugins_css` approach has been depreciated as of [Academic Version 4.6](https://sourcethemes.com/academic/updates/v4.6.0/). Custom JavaScript in `params.toml` is still specified as `plugins_js`; this is the way custom CSS used to be input; now it all goes into a single css file. 

   1. Create a new folder `./assets/scss/` with a file `custom.scss`. 
   2. Copy and paste your custom CSS into this file. Note that SCSS is a superset of CSS. 
   3. Put all CSS into this `custom.scss` file.

5. Copy over my custom font selection and custom theme. These are `toml` files in `./data/fonts` and `./data/themes`. Update the `params.toml` file to point to these files:

   ```
   theme = "fliptheme"
   font = "flipfont"
   ```

6. Copy the contents of the `./static/img/` folder and the entirety of the `./static/files/` folder. 

7. New in this version: favicon now go to `./assets/images` as a 512x512 image titled `icon.png`. If there is also a file names `logo.png` then this will be uesd in place of the site name on the menubar.

## Initial Set Up

#### config.toml 

In `config/_default/config.toml`, edit the site URL:

```toml
baseurl = "https://theory.ucr.edu/flip/"
```

Update the site title and copyright blurb

#### menus.toml

**Tip**: *edit this file as you write new sections in `content/home/`. If this page refers to sections that don't exist yet then the Hugo server will give you an error and you won't be able to preview your work.*

In `config/_default/menus.toml` edit the menu bar items. Each item looks like this:

```toml
[[main]]
  name = "Home"
  url = "#about"
  weight = 10
```

The **name** is what shows up on the top menu bar.  A **url** of `#about` means it links to the section `content/home/about.md` (make sure that file exists). The order of items in the menu bar and on the home page is given by the **weight**. A section with weight 10 will show up ahead of a section with weight 20. 

#### params.toml

In `config/_default/params.toml` :

* `color_theme = "fliptheme"` assuming the existence of `data/themes/fliptheme.toml`
* `day_night = false`, no light/dark mode toggling.
* `font = "flipfont" `assuming the existence of `data/fonts/flipfont.toml`
* Edit other details as appropriate; e.g. contact info, turn off search

#### ./content/authors/admin

Update `avatar.jpg` with your photo, update `_index.md`. This is where the `about` widget draws its information from.

In `content/authors/admin`:

- replace `avatar.jpg` with a profile photo
- Update the personal data in `_index.md`
  - We won't be using the `interests` or  `education` lists in `_index.md`
- Transfer over the long list of **social** icons. We'll eventually edit the template files so that they show up under the bio blurb, not the profile picture.

## Widget Cleaning

Delete excess widgets. The default *Academic* kickstart comes with a bunch of demo widgets. Let's clear them out. In `content\home\` you can remove everything except `about.md`, `contact.md`,`index.md`, `slider.md`.

We'll fill in the content later.

**Remark**: *at this stage, the site probably looks a bit funky... colors and spacing will be a little off. Feel free to not call the css files yet to return to a slightly more conventional appearance.*

## Background and Footer

There are two major design edits here. They're a bit of a pain to implement, but that's my hubris for you. 

The superstructure of an *Academic* page is in `themes/academic/layouts/_default/baseof.html`. Go ahead and copy this file to `/layouts/_default/baseof.html` so that we may edit it. Create the subdirectories as necessary and make sure you're not editing anything in the `themes` directory.

Here's what the default `baseof.html` looks like.

```html
<!DOCTYPE html>
{{- $language_code := site.LanguageCode | default "en-us" -}}
<html lang="{{$language_code}}" {{ if in site.Data.i18n.rtl.rtl $language_code }}dir="rtl"{{end}}>

{{ partial "site_head" . }}
<body id="top" data-spy="scroll" data-offset="70" data-target="{{ if or .IsHome (eq .Type "widget_page") }}#navbar-main{{else}}#TableOfContents{{end}}" {{ if not (.Scratch.Get "light") }}class="dark"{{end}}>

  {{ partial "search" . }}

  {{ partial "navbar" . }}

  {{ block "main" . }}{{ end }}

  {{ partial "site_js" . }}

  {{/* Docs and Updates layouts include the site footer in a different location. */}}
  {{ if not (in (slice "docs" "updates") .Type) }}
  <div class="container">
    {{ partial "site_footer" . }}
  </div>
  {{ end }}

  {{ partial "citation" . }}

</body>
</html>

```

We can see where the navigation bar is called and where the footer is called. The stuff in between calls the sections of the home page. Here's what we want to do:

1. **We want the background of the `<body>` to be dark gray.** Aesthetically we want the background to be white. However, the navigation bar and footer will be dark gray. What this means is that if one over-scrolls (pulls above or below the main page by a little) then you get a bit of white pulling from under the footer or from above the navigation bar. This looks distracting, so we're going to jump through some hoops. This involves:

   a) setting the `<body>` background to be dark gray. If you've copied over the style files, this should already be active.

   b) creating a new `<div id="THECONTENT">` that has a white background. All of the sections will be inside this division. Over scrolling, however, will pull more space from *outside* this division, which will be dark gray.

2. **We want the fancy footer**. I have a neat Feynman diagram footer graphic that I like.  Observe that the `<div class="container">` that encloses the footer does not spread across the entire navigator width.  The `container` class is something inherited from Bootstrap, the responsive design grid system that *Academic* is built upon.

   We have to place additional `<div>`s just around and above the footer. 

Bonus: at this stage you can also put in the `baseof.html` code for a watermark.

Here's what it looks like now:

```html
<!DOCTYPE html>
{{- $language_code := site.LanguageCode | default "en-us" -}}
<html lang="{{$language_code}}" {{ if in site.Data.i18n.rtl.rtl $language_code }}dir="rtl"{{end}}>

{{ partial "site_head" . }}
<body id="top" data-spy="scroll" data-offset="70" data-target="{{ if or .IsHome (eq .Type "widget_page") }}#navbar-main{{else}}#TableOfContents{{end}}" {{ if not (.Scratch.Get "light") }}class="dark"{{end}}>

  {{ partial "search" . }}


<!-- FLIP: FOR WATERMARK -->
<div id="watermark" style="background-image:url('{{ $.Site.BaseURL }}/img/{{ .Site.Params.watermark }}');"></div>
<!-- /FLIP --> 

  {{ partial "navbar" . }}

<!-- FLIP 2019 -->
<div id="THECONTENT"> <!-- closed below; see flip2019.css -->
<!-- /FLIP 2019 -->

  {{ block "main" . }}{{ end }}

  {{ partial "site_js" . }}

  {{/* Docs and Updates layouts include the site footer in a different location. */}}
  {{ if not (in (slice "docs" "updates") .Type) }}
  <!-- FLIP -->
  <div style="position: relative; width: 0; height: 0">
  <div id="feynmanfoot" style="background-image:url('{{ $.Site.BaseURL }}/img/{{ .Site.Params.footmark }}');"></div>
  </div>
  <div id="botbar1"></div>
  <!-- -- -->
  <div id="FOOTERBAR">
  <!-- /FLIP -->
  <div class="container">
    {{ partial "site_footer" . }}
  </div>
  <!-- FLIP -->
  </div> <!-- closes div FOOTERBAR -->
  <!-- /FLIP -->
  {{ end }}

<!-- FLIP -->
</div> <!-- closes id="THECONTENT" -->
<!-- /FLIP -->

  {{ partial "citation" . }}

</body>
</html>

```

Then go ahead and define `mylogo` and `midlogo` in `params.toml`.  We might as well add in a few extra parameters that we'll need.

```toml
## ADDED BY FLIP
# Feynman Diagram Footer
footmark = "layout/feynmanfooter.png"

# Footer logos
midlogo = "logo/UCRPAlogo.png"
mylogo = "logo/FlipAmbigram.png"

# watermark on the about widget
watermark = "background/Bundle3.jpg"
## /FLIP
```

