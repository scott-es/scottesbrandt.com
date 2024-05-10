---
layout: post
title:  "Enable dark mode on your jekyll site without writing CSS"
date:   2024-04-24 23:31:43 -0400
categories: jekyll
---

### <span style="color: #666666;">Goal & approach</span>

My blog site was originally designed based on default jekyll theme having a light color scheme.  My goal was to add a dark mode without writing additional CSS, something quick and easy. Although it's straightforward to conditionally apply styles with CSS [@media at-rule](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries#targeting_media_features), I've opted to use an existing JavaScript library called `darkreader` to switch the sites color scheme dynamically.  If darkreader does a good enough job of dynamically styling the site for dark mode, then this avoids the need to write any alternative CSS! 

I would like to detect the client browser's color scheme preference on the first visit and enable it automatically, as well as provide a toggle for visitors to switch color modes on-demand. 

<br/>

### <span style="color: #666666;">Customizing jekyll</span>

I will need to include the `darkreader` Javascript as well as implement my own scripting for toggling the color scheme and detecting browser preference, which means I need to modify the existing Jekyll theme defaults.  Overriding theme defaults is described in detail in the [jekyll docs](https://jekyllrb.com/docs/themes/#overriding-theme-defaults). 

For my purposes, I will be customizing 3 files: <br/>
 - `_includes/head.html`: to include external JS and CSS resources<br/>
 - `_includes/header.html`: to place the toggle component in the HTML layout <br/>
 - `_includes/footer.html`: to call functions for auto detection and toggling <br/>

 <br/>

### <span style="color: #666666;">Set up client-side resources</span>

First we must include the necessary scripts and CSS client side.

Resource: [darkreader](https://www.npmjs.com/package/darkreader)
to dynamically change the site to a dark-mode color scheme
<br/>
Resource: [Font Awesome](https://opensource.com/article/22/9/dark-theme-website#:~:text=You%20can%20include%20an%20easy,while%20in%20the%20dark%20theme.)
to use free dark mode button css and icons

I will also use my own JS file to store some functions I will need, located in my sites home directory under `assets/js/color-scheme.js`

Modify `_includes/head.html` and add the following: <br/>

```html
<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.0.7/css/all.css">
<script src="https://cdn.jsdelivr.net/npm/darkreader@4.9.84/darkreader.min.js"></script> 
<script src="/assets/js/color-scheme.js"></script>
```

This will allow me to use the moon and sun buttons from Font Awesome as well as leverage the dynamic dark mode functionality provided by darkreader to toggle on or off.

<br/>

### <span style="color: #666666;">Add toggle to site header</span>

Modify `_includes/header.html` and add the following to your HTML layout

```html
<span>
  <a class="dark-mode-button"  style="padding-left: 25px; text-decoration: none; cursor: pointer;">
    <i id="icon-dark" class="fa fa-moon"></i>
    <i id="icon-light" class="fa fa-sun"></i>
  </a>
</span>

```

This will display the moon and sun icons next to one another.

<br/>

### <span style="color: #666666;">Enable auto-detect and toggle functionality</span>
Now that I have the visual elements in place and the external resources set up, I need to make it functional.

Modify `_includes/footer.html` and add the following at the bottom before the closing `</footer>` tag.

```html
<script>

    const iconDark = document.getElementById('icon-dark');
    const iconLight = document.getElementById('icon-light');
    const darkModeButton = document.getElementsByClassName('dark-mode-button')[0]
    const darkModePreferred = (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches)

    window.onload = function() {
      checkDarkMode()
    }

    darkModeButton.onclick = function() {
      toggleDarkMode()
    }

</script>
```

<em>Note: `checkDarkMode()` and `toggleDarkMode()` are defined in the following step.</em>

Rather than include the code below directly in `_includes/footer.html`, I chose to put them in a separate file
to keep things tidy.  

Place the following code to `assets/js/color-scheme.js`

```js
// Setup darkreader for CORS
DarkReader.setFetchMethod(url => {
  let headers = new Headers()
  headers.append('Access-Control-Allow-Origin', '*')

  return window.fetch(url, {
    headers,
    mode: 'no-cors',
  })
})

function darkModeSelected() {
  let darkModeSelected = localStorage.getItem('darkMode');

  if (darkModeSelected === "false"){
    return false;
  }
  if (darkModeSelected === "true") {
    return true;
  }
  return darkModeSelected;
}

function darkModeEnabled() {
  if (darkModeSelected() === true) {
    return true;
  }
  if (darkModeSelected() === null && darkModePreferred === true) {
    return true;
  }
  return false;
}

// set color mode and icons on page load
function checkDarkMode() {
  if (darkModeEnabled()) {
    DarkReader.enable();
  } else {
    DarkReader.disable();
  }

  if (darkModeSelected() === true) {
    setIconsDarkModeOn();
  } else if (darkModeSelected() === false) {
    setIconsDarkModeOff();
  }
}

function toggleDarkMode() {
  if (darkModeEnabled()) {
    disableDarkMode();
  } else {
    enableDarkMode();
  }
}

function enableDarkMode() {
  DarkReader.enable();
  localStorage.setItem('darkMode', 'true');
  setIconsDarkModeOn();
}

function disableDarkMode() {
  DarkReader.disable();
  localStorage.setItem('darkMode', 'false');
  setIconsDarkModeOff();
}

function setIconsDarkModeOn() {
  iconLight.style.color = "grey";
  iconDark.style.color = "blue";
}

function setIconsDarkModeOff() {
  iconDark.style.color = "grey";
  iconLight.style.color = "blue";
}
```

`localStorage` is used to remember the visitor's preference for the site if they have overrided the 
detected browser preference.

<br/>
