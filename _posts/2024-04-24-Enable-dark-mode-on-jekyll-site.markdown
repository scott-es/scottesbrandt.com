---
layout: post
title:  "Enable dark mode on your jekyll site"
date:   2024-04-24 23:31:43 -0400
categories: jekyll
---

### <span style="color: #666666;">Approach</span>

Although it's straightforward to conditionally apply styles with CSS [@media at-rule](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries#targeting_media_features), I've opted to use an existing javascript library called `darkreader` to switch the sites color scheme dynamically.  This avoids the need to write my own alternative CSS for a "dark mode". 

The site already has a light default color scheme. Hopefully darkreader will do a decent job of converting it into a visually appealling dark mode on demand. Let's see how it goes!

### <span style="color: #666666;">Customizing jekyll</span>

I will need to include the `darkreader` Javascript as well as implement our own scripting, which means we need to modify the existing Jekyll theme defaults.  This is described in detail in the [jekyll docs](https://jekyllrb.com/docs/themes/#overriding-theme-defaults).

### <span style="color: #666666;">Include darkreader script</span>

Resources: [darkreader](https://www.npmjs.com/package/darkreader)

First we must set up `darkreader`, to include the necessary scripts client side, rather than including them via npm package, since we are running this on a static site.

#### Download the script

To avoid CORS issues, I've just [downloaded](https://unpkg.com/browse/darkreader@4.9.83/darkreader.js) and included the `darkreader` JS as a static asset in my jekyll site, and then include the following script in the `head` section of the HTML page.  In my case, I renamed `darkreader.js` to `darkreader-4.9.83.js`.

1. If not already present, create a folder called `assets` in the root of your jekyll project. Create another folder called `js` underneath assets, and place the darkreader script there.  

#### Load the script from HTML head section

1. Modify `_includes/head.html` and add the following <br/><br/>
```html
<script src="assets/js/darkreader-4.9.83.js"></script>
```
2. If your local jekyll server is running, please restart it


### <span style="color: #666666;">Detect if browser prefers dark mode</span>

Resources: [prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme)

Using strategy of light color scheme by default, then using detection of `prefers-color-scheme: dark` to switch to darker color scheme.

Rather than creating new CSS for dark mode, I decide to use a library that can do this for me.  
Let's see how [darkreader](https://www.npmjs.com/package/darkreader) works in this scenaro.

I can detect if the client browser's preferred color scheme is dark using following JavaScript

```js
const prefersDarkColorScheme = () =>
  window &&
  window.matchMedia &&
  window.matchMedia('(prefers-color-scheme: dark)').matches;
```

where `prefersDarkColorScheme()` will be evaluated to either `true` or `false`

I chose to put this in the footer of the page, by overriding jekyll's footer via _includes/footer.html as described here.

<br/>

### <span style="color: #666666;">Activate dark mode when preference is detected</span>



Still within in the `head` section, let's include the following bit of code then

```js
if (prefersDarkColorScheme()) {
   DarkReader.enable()
}
```

<br/>

### <span style="color: #666666;">Improvement: Allow user to manually select dark or light mode</span>

Resource: [Font Awesome](https://opensource.com/article/22/9/dark-theme-website#:~:text=You%20can%20include%20an%20easy,while%20in%20the%20dark%20theme.)

Let's make a toggle available on the site to switch between dark or light mode.  We will use Font Awesome CSS for moon and sun icons for our switcher. 

Include Font Awesome, in the `head` section of HTML pages

```html
<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.0.7/css/all.css">
```

