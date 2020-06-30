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

This is a good checkpoint: the website should look reasonable at this stage.

## Tweaking the navigation bar 

The **breakpoint** of a navigation bar is the window width at which the bar collapses into the familiar "three horizontal bars" symbol for an expanding menu. Bootstrap 4 makes this easy. [Here's how it works](https://stackoverflow.com/a/36289507/4812646):

> Changing the navbar breakpoint is easier in Bootstrap 4 using the navbar-expand-* classes:

```html
<nav class="navbar fixed-top navbar-expand-sm">..</nav>
```

The options are

- `navbar-expand-sm`: mobile menu on xs screens <576px
- `navbar-expand-md`: mobile menu on sm screens <768px
- `navbar-expand-lg`: mobile menu on md screens <992px
- `navbar-expand-xl`: mobile menu on lg screens <1200px
- `navbar-expand`: never use mobile menu
- *(no expand class)* = always use mobile menu

The relevant line shows up at the top of `themes/academic/layouts/partials/navbar.html`. You know what that means: make a copy of that file in `/layouts/partials/` and change 

```html
<nav class="navbar navbar-light fixed-top navbar-expand-lg py-0 compensate-for-scrollbar" id="navbar-main">
```

to

```html
<nav class="navbar navbar-light fixed-top navbar-expand-md py-0 compensate-for-scrollbar" id="navbar-main">
```

You can use `navbar-expand-sm` if your menu is particularly brief. 

By the way, this is also *way* easier than it used to be in Bootstrap 3, remember to be grateful.

## Three Column Footer

To edit the footer, use `themes/academic/layouts/partials/site_footer.html`. Copy it to `layouts/partials/`. For simplicity, I'm assuming no `terms.md` or `privacy.md` so I'll just delete that part of `site_footer.html`.

Here's my modification:

```html
<footer class="site-footer">

<!-- FLIP 2019 -->
<div class="row" id="footer-columns">
  <div class="col-md-4" id="footer-col-1">
    <img src="{{ $.Site.BaseURL }}img/{{ $.Site.Params.mylogo }}" class="center-me">
  </div>
  <div class="col-md-4" id="footer-col-1">
    <img src="{{ $.Site.BaseURL }}img/{{ $.Site.Params.midlogo }}" class="center-me">
  </div>
  <div class="col-md-4" id="footer-col-1">
    <p class="powered-by">
    {{ with site.Copyright }}{{ replace . "{year}" now.Year | markdownify}} &middot; {{ end }}

    Powered by the
    <a href="https://sourcethemes.com/academic/" target="_blank" rel="noopener">Academic theme</a> for
    <a href="https://gohugo.io" target="_blank" rel="noopener">Hugo</a>.

    {{ if ne .Type "docs" }}
    <span class="float-right" aria-hidden="true">
      <a href="#" id="back_to_top">
        <span class="button_icon">
          <i class="fas fa-chevron-up fa-2x"></i>
        </span>
      </a>
    </span>
    {{ end }}
  </p>
  </div>
</div>
<!-- /FLIP 2019 -->

</footer>

```

Then go ahead and define `mylogo` and `midlogo` in `params.toml`. 

```toml
# Footer logos
midlogo = "logo/UCRPAlogo.png"
mylogo = "logo/FlipAmbigram.png"
```

### About Widget 

One fancy thing to add in is a watermark. If you copied the watermark code in `baseof.html` and `params.toml` (as well as the relevant parts of `custom.css`, it should already be up and running. 

In `baseof.html`: the code:

```html
<!-- FLIP: FOR WATERMARK -->
<div id="watermark" style="background-image:url('{{ $.Site.BaseURL }}/img/{{ .Site.Params.watermark }}');"></div>
<!-- /FLIP -->	
  {{ partial "navbar.html" . }}
```

Note that I've tried to annotate my edits compared to the default file. This is helpful since the next time I make edits, the default file may have been upgraded and I need to remember where to put hacks.

Go ahead and define `watermark` in `params.toml`:

```toml
# watermark on the about widget
watermark = "background/Bundle3.jpg"
```

The next thing to tweak is the template for this widget itself. 

Copy `themes/academic/layouts/partials/widgets/about.html` to `layouts/partials/widgets/about.html`. 

Now remove the parts that have to do with the *interests* and *education* information; those will go into the **cv** widget. 

You can just comment out the chunk of code right below `{{ $person_page.Content }}`:

```html
    <div class="row">

      {{ with $person.interests }}
      <div class="col-md-5">
        <h3>{{ i18n "interests" | markdownify }}</h3>
        <ul class="ul-interests">
          {{ range . }}
          <li>{{ . | markdownify | emojify }}</li>
          {{ end }}
        </ul>
      </div>
      {{ end }}

      {{ with $person.education }}
      <div class="col-md-7">
        <h3>{{ i18n "education" | markdownify }}</h3>
        <ul class="ul-edu fa-ul">
          {{ range .courses }}
          <li>
            <i class="fa-li fas fa-graduation-cap"></i>
            <div class="description">
              <p class="course">{{ .course }}{{ with .year }}, {{ . }}{{ end }}</p>
              <p class="institution">{{ .institution }}</p>
            </div>
          </li>
          {{ end }}
        </ul>
      </div>
      {{ end }}

    </div>
```

Next we want to move the list of icons. It defaults to being under the profile photo and title. We'd like to move it to after the mini biography.

This is the piece of code to move:

```html
      <ul class="network-icon" aria-hidden="true">
        {{ range $person.social }}
        {{ $pack := or .icon_pack "fas" }}
        {{ $pack_prefix := $pack }}
        {{ if in (slice "fab" "fas" "far" "fal") $pack }}
          {{ $pack_prefix = "fa" }}
        {{ end }}
        {{ $link := .link }}
        {{ $scheme := (urls.Parse $link).Scheme }}
        {{ $target := "" }}
        {{ if not $scheme }}
          {{ $link = .link | relLangURL }}
        {{ else if in (slice "http" "https") $scheme }}
          {{ $target = "target=\"_blank\" rel=\"noopener\"" }}
        {{ end }}
        <li>
          <a href="{{ $link | safeURL }}" {{ $target | safeHTMLAttr }}>
            <i class="{{ $pack }} {{ $pack_prefix }}-{{ .icon }} big-icon"></i>
          </a>
        </li>
        {{ end }}
      </ul>
```

Move it to the following line in `about.html`:

```html
    {{ $person_page.Content }}
```

Note: by default the `./content/home/about.md` widget doesn't have any content. The biographical blurb draws from `./content/authors/admin/_index.md`. This is an opportunity: any content you put in `about.md` now appears under the icon bar. Let's make that content `<small>` by default so that it appears as a foot note. In `about.html`, add the following code after the icon list:

```html
<small>{{ $page.Content }}</small>
```

One can also make the `<small>` tag of class `comment` to make it gray.

And go ahead an put a blurb in the main content of `about.md` (anything below the `+++`). Note that `$person_page.Content` draws from `_index.md` while `$page.Content` draws from `about.md`.

### CV Widget

The next item on the website should be a mini-CV with links to a full CV. Copy `themes/academic/layouts/partials/widgets/blank.html` to `layouts/partials/widgets/blank.html` and rename it to `CV.html`. 

Now make a copy of `content/home/demo.md` (from `./themes/academic` if you deleted it) and name it `CV.md`. In `CV.md`: 

```markdown
+++
widget = "CV"  
headless = true  # This file represents a page section.
active = true  # Activate this widget? true/false
weight = 25  # Order that this section will appear.

title = "Curriculum Vitae"
subtitle = ""

# CV location
cv_pdf = "./files/Tanedo.pdf"

# Group Logo
group_logo = "./img/logo/UCRHEP_03.png"

[design]
  # Choose how many columns the section has. Valid values: 1 or 2.
  columns = "2"

[advanced]
 # Custom CSS. 
 css_style = ""
 
 # CSS class.
 css_class = ""

[interests]
  interests = [
    "Dark matter",
    "Models of new physics",
    "Particle astro/cosmo",
    "Equity in science"
  ]

[[education.courses]]
  course = "PhD in Physics"
  course_short = "PhD"
  institution = "Cornell University"
  institution_short = "Cornell"
  year = 2013
  logo = "/logo/icon_Co.png"

[[education.courses]]
  course = "MSc in Physics"
  course_short = "MSc"
  institution = "Durham University / IPPP"
  institution_short = "Durham/IPPP"
  year = 2008
  logo = "/logo/icon_D.png"

[[education.courses]]
  course = "MASt in Mathematics"
  course_short = "MASt"
  institution = "Cambridge University"
  institution_short = "Cambridge"
  year = 2007
  logo = "/logo/icon_Ca.png"

[[education.courses]]
  course = "BS in Physics & Mathematics"
  course_short = "BS"
  institution = "Stanford University"
  institution_short = "Stanford"
  year = 2008
  logo = "/logo/icon_S.png"

[service]
  service = [
    "Open House Committee (chair)",
    "Website Committee (chair)",
    "Graduate Diversity Committee",
    "Outreach Committee",
    "Advancing Faculty Diversity Hiring Committee",
    "[Reviewer](https://publons.com/author/637273/)",
    "[Cientifico Latino Mentor](https://www.cientificolatino.com)",
    "[Phy Sci Book Club Moderator](https://www.cellardoorbookstore.com/book-clubs)"
  ]
+++
**Flip Tanedo** is an assistant professor of theoretical physics at the University of California, Riverside. His research seeks to discover how dark matter fits into our fundamental understanding of nature.

- UCI Chancellor's Advance Postdoctoral Fellow, 2014 - 2015  
- Paul & Daisy Soros Fellowship, 2010 - 2012  
- NSF Graduate Research Fellow, 2006 - 2011  
- Marshall Scholarship, 2006 - 2008
```

Now modify `CV.html` to call the additional page data:

```html
{{ $st := .page }}
{{ $columns := $st.Params.design.columns | default "2" }}

<div class="row">
  {{ if ne $columns "1" }}
    <div class="col-12 col-lg-4 section-heading">
      <!-- FLIP: h1 -> h2 -->
      {{ with $st.Title }}<h2>{{ . | markdownify | emojify }}</h2>{{ end }}
      <!-- /FLIP -->
      {{ with $st.Params.subtitle }}<p>{{ . | markdownify | emojify }}</p>{{ end }}
      <!-- FLIP -->
      <p>
        <a class="btn btn-outline-primary btn-xs" href="{{ $st.Params.cv_pdf }}">
          Download Complete CV
        </a>
      </p>
      {{ if $st.Params.group_logo }}
      <p>
        <img class="sidebarpic" src="{{ $st.Params.group_logo }}">
        <meta itemprop="image" content="{{ $st.Params.group_logo }}">
      </p>
      {{ end }}
      <!-- /FLIP -->
    </div>
    <div class="col-12 col-lg-8">
      {{ $st.Content }}
      <!-- FLIP -->
      <div class="row">

      {{ with $st.Params.interests }}
      <div class="col-md-5">
        <h3>{{ i18n "interests" | markdownify }}</h3>
        <ul class="ul-interests">
          {{ range .interests }}
          <li>{{ . | markdownify }}</li>
          {{ end }}
        </ul>
      </div>
      {{ end }}

      {{ with $st.Params.education }}
      <div class="col-md-7">
        <h3>{{ i18n "education" | markdownify }}</h3>
        <ul class="ul-edu fa-ul">
          {{ range .courses }}
            {{ if .logo }}
              <img src="{{ $.Site.BaseURL }}img/{{ .logo }}" style="height:1rem; float: left; padding-right: 10px;">
            {{ else }}
              <i class="fa-li fas fa-graduation-cap"></i>
            {{ end }}
            <div class="description">
              {{ .course_short }}
              {{ with .institution_short }}, {{ . }}{{ end }}
              {{ with .year }}({{ . }}){{ end }}
              <br />
            </div>
          {{ end }}
        </ul>
      </div>
      {{ end }}

      </div>
     {{ with $st.Params.service }}
      <p style="font-size: .8rem;">
        <b>Service</b>:
        {{ range .service }}
        {{ . | markdownify }} &middot;
        {{ end }}
      </p>
      {{ end }}
      <!-- /FLIP -->
    </div>
  {{ else }}
    <div class="col-lg-12">
      {{ with $st.Title }}<h1>{{ . | markdownify | emojify }}</h1>{{ end }}
      {{ with $st.Params.subtitle }}<p>{{ . | markdownify | emojify }}</p>{{ end }}
      {{ $st.Content }}
    </div>
  {{ end }}
</div>
```



Note also the minor tweak of `<h1>` to `<h2>` in the following line:

```html
{{ with $st.Title }}<h2>{{ . | markdownify | emojify }}</h2>{{ end }}
```

This is so that "Curriculum Vitae" fits on one line on large screens. 

## Some additional comments from CV widget

This section is just a discussion of some of the edits above, you can skip it if everything runs well and you don't need to tweak anything.

##### Add a "download pdf" button

In `CV.html` under `<div class="col-xs-12 col-md-4 section-heading">`:

```html
    <p>
    <a class="btn btn-outline-primary btn-xs" href="{{ $st.Params.cv_pdf }}">
      Download Complete CV
    </a>
    </p>
```

Note that in previous versions of *Academic* the templates used `$page` instead of `$st`. Also, previous versions of Bootstrap had a different `btn` class structure (`btn btn-primary btn-outline` instead of `btn btn-outline-primary`). I remark on these to highlight the ways in which template updates show up.

In the header of `CV.md`:

```md
# CV location
cv_pdf = "./files/Tanedo.pdf"
```

Note that this has to go between the `+++` in the header section, and above any list parameters (Defined with brackets: `[list]`). 

In the file structure: `/static/files/Tanedo.pdf`. 

##### Adding the research group logo

A class `sidebarpic` is defined in `flip2018.css` specifically for the CV widget. In `flip2018.css`:  

```css
/*  for CV WIDGET  */

.sidebarpic{
  /* sets max size  */
  width: 250px;
  height: 250px;
  text-align: center;
  margin: 0 auto;
}
```

In `CV.md`:  

```md
# Group Logo
group_logo = "./img/logo/UCRHEP_03.png"
```

In `CV.html`:  

```html
    {{ if $page.Params.group_logo }}
    <p>
      <img class="sidebarpic" src="{{ $page.Params.group_logo }}">
      <meta itemprop="image" content="{{ $page.Params.group_logo }}">
    </p>
    {{ end }}
```

Note that I was a bit more clever with the go `if` statement here. That's probably good practice.

##### Filling in the CV widget 

In `CV.html`:

```html
 <div class="col-12 col-lg-8">
    {{ $st.Content }}

    <!-- FLIP 2019  -->
    <div class="row">

      {{ with $st.Params.interests }}
      <div class="col-md-5">
        <h3>{{ i18n "interests" | markdownify }}</h3>
        <ul class="ul-interests">
          {{ range .interests }}
          <li>{{ . | markdownify }}</li>
          {{ end }}
        </ul>
      </div>
      {{ end }}

      {{ with $st.Params.education }}
      <div class="col-md-7">
        <h3>{{ i18n "education" | markdownify }}</h3>
        <ul class="ul-edu fa-ul">
          {{ range .courses }}
          <!-- <li> -->
            {{ if .logo }}
              <img src="{{ $.Site.BaseURL }}img/{{ .logo }}" style="height:1rem; float: left; padding-right: 10px;">
            {{ else }}
              <i class="fa-li fas fa-graduation-cap"></i>
            {{ end }}
            <div class="description">
              {{ .course_short }}
              {{ with .institution_short }}, {{ . }}{{ end }}
              {{ with .year }}({{ . }}){{ end }}
              <br />
            </div>
          <!-- </li> -->
          {{ end }}
        </ul>
      </div>
      {{ end }}

    </div>
    {{ with $st.Params.service }}
      <p style="font-size: .8rem;">
        <b>Service</b>:
        {{ range .service }}
        {{ . | markdownify }} ,
        {{ end }}
      </p>
    {{ end }}
    <!-- /FLIP 2019 -->
  </div>
```

In `CV.md` header:

```toml
[interests]
  interests = [
    "Dark matter",
    "Models of new physics",
    "Particle astro/cosmo",
    "Equity in science"
  ]

[[education.courses]]
  course = "PhD in Physics"
  course_short = "PhD"
  institution = "Cornell University"
  institution_short = "Cornell"
  year = 2013
  logo = "/logo/icon_Co.png"

[[education.courses]]
  course = "MSc in Physics"
  course_short = "MSc"
  institution = "Durham University / IPPP"
  institution_short = "Durham/IPPP"
  year = 2008
  logo = "/logo/icon_D.png"

[[education.courses]]
  course = "MASt in Mathematics"
  course_short = "MASt"
  institution = "Cambridge University"
  institution_short = "Cambridge"
  year = 2007
  logo = "/logo/icon_Ca.png"

[[education.courses]]
  course = "BS in Physics & Mathematics"
  course_short = "BS"
  institution = "Stanford University"
  institution_short = "Stanford"
  year = 2008
  logo = "/logo/icon_S.png"
  
 [service]
  service = [
    "Open House Committee (chair)",
    "Website Committee (chair)",
    "Graduate Diversity Committee",
    "[Graduate Student Mentor](https://gradmentors.ucr.edu)",
    "[Reviewer](https://publons.com/author/637273/)"
  ]
```

### ## Research Carousel

Rename `./content/home/slider.md` to `research.md`. Copy over the old version. Here's what the top of my file looks like (adding more items is straightforward):

```markdown
+++
# Slider widget.
widget = "slider"  # See https://sourcethemes.com/academic/docs/page-builder/
headless = true  # This file represents a page section.
active = true  # Activate this widget? true/false
weight = 30  # Order that this section will appear.

# Slide interval.
# Use `false` to disable animation or enter a time in ms, e.g. `5000` (5s).
interval = false

# Slide height (optional).
# E.g. `500px` for 500 pixels or `calc(100vh - 70px)` for full screen.
height = "400px"

# Slides.
# Duplicate an `[[item]]` block to add more slides.
[[item]]
  title = "VSIDM"
  content = "Higgsed Yang-Mills Dark Sector"
  align = "right"  # Choose `center`, `left`, or `right`.
  overlay_img = "research/vsidm.png"  # relative to `static/img/`
  overlay_filter = 0.5  # Darken the image. Value in range 0-1.
  cta_label = "1907.10217"
  cta_url = "https://arxiv.org/abs/1907.10217"
  cta_icon_pack = "fas"
  cta_icon = "graduation-cap"
```



### Teaching Widget

I like having a grid of courses that I've taught. Eventually I might need to do something about archiving old courses. The style sheet for this is `flipiconlist.css`. This defines classes `teaching`, `smaller`, and `teachingpic`. Now we modify a clean widget template (use `blank.html`) and rename it to `teaching.html`. By now you know where to put it. Here's what my `teaching.html` looks like:

```html
{{ $st := .page }}
{{ $columns := $st.Params.design.columns | default "2" }}

<div class="row">
    <div class="col-12 col-lg-4 section-heading">
      {{ with $st.Title }}<h1>{{ . | markdownify | emojify }}</h1>{{ end }}
      {{ with $st.Params.subtitle }}<p>{{ . | markdownify | emojify }}</p>{{ end }}
    </div>
    <div class="col-12 col-lg-8">
      <!-- FLIP 2019 -->
      <div class="row">
        {{ with $st.Params.teaching }}
          {{ range .class }}
            <div class="col-3 teaching">
             <a href="{{ .website }}"><img class="teachingpic" src="{{ $.Site.BaseURL }}img/teaching/{{ .photo }}"></a>
             <a href="{{ .website }}">{{ .number }}, {{ .session }}</a> <br />
             <span class="smaller">{{ .name }}</span>
            </div>
          {{ end }}
        {{ end }}
      </div>
      <!-- /FLIP 2019 -->
      {{ $st.Content }}
    </div>
</div>
```

The format for the classes is:

```toml
[[teaching.class]]
  name = "Computational Physics"
  number = "P177"
  session = "Spr 2018"
  photo = "P177-2018.png"
  website = "https://github.com/Physics177-2018"
```

Note: the teaching page links to a "how to request a letter of recommendation" page. Go ahead and copy and pate `./content/recs/` over to the new site.

### Team

This is similar to the teaching.  Styling is now in `custom.css`. In the future, it may be better to make a separate style for *only* this widget, which seems to be something allowed. (I'm not sure where to put the css file, though.)

Student format:

```toml
[[mygroup.oldstudents]]
  name = "Corey Kownacki*"
  position = "Grad"
  start = "2017"
  end = "18"
  photo = "template_corey.jpg"
  website = "http://theory.ucr.edu/group.html"
```

Here's my kludge for the `students.html` style, it goes right below `{{ $st.Content }}`: (Note: I removed the `if/else` for number of columns; this widget has two columns)

```html
      <!-- FLIP 2019 -->
      
      {{ with $st.Params.mygroup }}
        <ul class="ul-students">
          {{ range .students }}
            <li class="ul-students" title="{{ .name }}">
              <a href="{{ .website }}"><img class="studentpic" src="{{ $.Site.BaseURL }}img/students/{{ .photo }}"></a>
            </li>
          {{ end }}
        </ul>
      {{ end }}

    <p>
      <span class="comment">* - co-advised</span>
    </p>

    <div class="row comment">
      <h5>Past Students</h5>
    </div>

    {{ with $st.Params.mygroup }}
      {{ range .oldstudents }}
        <div class="row comment">
          <div class="col-xs-3">
            <a href="{{ .website }}" class=comment>
              {{ .name }}&nbsp;
            </a>
          </div>
          <div class="col-xs-3">
            {{ .start }} - {{ .end }}&nbsp;
          </div>
          <div class="col-xs-6">
            {{ .position }}, {{ .role }}
          </div>
        </div>
      {{ end }}
    {{ end }}


    <!-- /FLIP 2019 -->
```

(I've made a few other tweaks in the html file.)

### Twitter / Public

This uses the blank widget with a twitter **shortcode** (Hugo's way of inserting bits of html). I had to adapt this since last year due to some updates on how Hugo does short codes. 

### Contact Widget; crypted e-mail

#### Google Calendar

Google Calendar links go into the `toml` of the `contact.md` widget. Here's how it works. In the header of `contact.md`:

```toml
# Google Calendar
[[calendar]]
  src = "aXE0b2FzM3YyMmszajYyZjJjaTMzMWhxZDRAZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ"

[[calendar]]
  src = "NTQ1YzF2aWU5a3U4aWRxMnRnaWVtNjEwZ2tAZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ"

[[calendar]]
  src = "cW1kbHB2aGc1cG40bXQzN2VqNHQ4c3NiMGdAZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ"

[[calendar]]
  src = "ajRpMzBlc2E1bTFlNHE2YTYzNHVqMG5qNDRAZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ"
```

In `/layouts/partials/widgets/contact.html` :

```html
    {{ with $page.Content }}{{ . }}{{ end }}

    <!-- FLIP 2019 -->
    {{ with $page.Params.calendar }}
      <iframe src="https://calendar.google.com/calendar/embed?height=200&amp;wkst=1&amp;bgcolor=%23ffffff&amp;ctz=America%2FLos_Angeles&amp;{{ range $index, $item := $page.Params.calendar }}src={{ with $item.src }}{{ . }}{{ end }}&amp;{{ end }}color=%23C0CA33&amp;color=%23F4511E&amp;color=%237986CB&amp;color=%23F6BF26&amp;mode=AGENDA&amp;showNav=0&amp;showPrint=0&amp;showTabs=0&amp;showCalendars=0&amp;showTitle=0" style="border:solid 1px #777" width="98%" height="200" frameborder="0" scrolling="no"></iframe>
    {{ end }}
    <!-- /FLIP 2019 -->

    {{ if $page.Params.email_form }}
```

Note how the Go code is nested:

```go
{{ with $page.Params.calendar }}
     {{ range $index, $item := $page.Params.calendar }}
     src=
     	{{ with $item.src }}
     		{{ . }}
     	{{ end }}
     &amp;
		{{ end }}
	...</iframe>
{{ end }}
```

Unfortuantely it looks like something is going on the server side of Google Calendar and this only works on Chrome. Apparently this has to do with the "[prevent site cross-tracking](https://support.google.com/calendar/thread/49408914?hl=en)" default in Safari.

#### Crypted e-mail

Lifted from [this Stack Overflow thread](https://stackoverflow.com/a/41566570). The idea is that I want my e-mail address to be difficult to parse for spam bots. I now suspect that this is largely antiquated for two reasons:

1. I suspect that spam bots are much more sophisticated than they used to be. Further, I suspect that there are plenty of places (e.g. my university's directory) where my e-mail address is listed without any protections.
2. My university e-mail is routed through a fairly good spam filter.

Since the code is there, though, we might as well use it. The idea is that there should be no line that looks like an easy-to-parse e-mail address. This solution uses style sheets (hence we have to include  `cryptedmail.css` in our list of CSS files)

Here's what goes into the `contact.html` template (which you've copied from the `themes` folder into the `layouts`  folder).

```html
      <!-- FLIP 2019 -->
      <!-- from: https://stackoverflow.com/a/41566570 -->
      {{ with $.Site.Params.email1 }}
        <li>
          <i class="fa-li fas fa-envelope fa-2x" aria-hidden="true"></i>
          <span id="person-email" itemprop="email">
            <a data-name="{{ $.Site.Params.email1 }}" data-domain="{{ $.Site.Params.email2 }}" data-tld="{{ $.Site.Params.email3 }}" href="#" class="cryptedmail" onclick="window.location.href = 'mailto:' + this.dataset.name + '@' + this.dataset.domain + '.' + this.dataset.tld"></a>
          </span>
        </li>
      {{ end }}
      <!-- /FLIP 2019 -->
```

This defines new site parameters: `email1`, `email2` , and  `email3`. You should define those in `params.toml`:

```toml
  email1 = "flip.tanedo" # using cryptedmail/
  email2 = "ucr" # using cryptedmail
  email3 = "edu" # using cryptedmail
```

You can also go ahead and comment out the old e-mail call.

Note that `flip2018.css` includes a little bit of additional styling for e-mails. It uses the `Titillium Web` font (loaded in `flipfont.toml`): 

```css
#person-email{
  font-family: 'Titillium Web', sans-serif;
}

.style-email{
  font-family: 'Titillium Web', sans-serif;
}
```

## Don't forget to update menu.toml

now the site should be up and running. We just needs to update `menu.toml` to make sure we reference each section.

## Additional Pages

The steps above focused on getting the front page set up. All of the widgets appeared in some specified order on `index.html`. What if we'd like to make another static page? 

A good template for this is the `/content/talk/` subdirectory. It contains an `index.md` markdown page with content:

```toml
---
title: Recent & Upcoming Talks

# View.
#   1 = List
#   2 = Compact
#   3 = Card
view: 2

# Optional header image (relative to `static/img/` folder).
header:
  caption: ""
  image: ""
---
```

This content shows up at `/talk/`  relative to the site root. 

#### References

* [Creating static content using partials](https://discourse.gohugo.io/t/creating-static-content-that-uses-partials/265/19), cited in [this answer](https://stackoverflow.com/a/37515023)
* [Academic: managing content](https://sourcethemes.com/academic/docs/managing-content/)