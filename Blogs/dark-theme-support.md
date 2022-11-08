---
blurb: How to add dark theme support to your website.
title: Dark Theme Support
path: dark-theme-support
tags:
 - Theme
 - Dark
 - Browser
 - Web Development
published: 2021-09-26
length: 4
---

Adding Dark theme to your website has three straightforward requirements:

1. Script to update a DOM class.
2. Clickable element to toggle themes.
3. Dark/Light theme CSS styles.

If our styles are dependent upon a top level class, we can update this class and the browser will automatically start running with the new theme.

## Script

The following function inverts the current theme and sets the theme into local storage.

```javascript
// swap the theme and add selection to local storage.
function setTheme() {
    let newTheme = document.body.className == "dark" ? "light" : "dark";
    window.localStorage.setItem("theme", newTheme);
    document.body.className = newTheme;
}
```

To keep this theme consistent between site visits and page navigation we need to retrieve the theme from local storage (if present) or default to 'light' theme when the dom is received.

```javascript
    // set theme prioritising local storage before preferred theme.
    window.addEventListener('DOMContentLoaded', (event) => {
        const localStorageTheme = window.localStorage.getItem("theme");
        let siteTheme  = localStorageTheme ?? "light";
        document.body.className = siteTheme;
    });
```

## Clickable Element

You need a clickable html element which calls the `setTheme` function we defined earlier.

```html
<div id="theme-wrapper">
    <span id="theme" onClick="setTheme()"></span>
</div>
```

## Styling

The following CSS updates my colours, icons and even content. I had already created the colours so I can just import and assign them to variables.

```scss
@import "scss/Colors.scss";

:root {
    --color-page-background: #{$Grey-White};
    --color-text: #{$Black};
    --color-page-background-contrast: #{$White};
    --color-heading: #{$Blue};
    --color-tags: #{$White-Grey};
    --color-zebra: #{$White-Grey};
    --color-link: #{$LightBlue};
    --color-icon: #{$Black};
    --color-border: #{$Light-Grey};
    --color-highlight: #{$Black-Alternate};
    --site-owner: "Kaelan Reece";
    --theme-icon: url("/Images/Icons/sun.svg")
}

.dark {
    --color-page-background: #{$Grey-Black};
    --color-page-background-contrast: #{$Grey-Black-Darker};
    --color-text: #{$White};
    --color-heading: #{$Teal};
    --color-tags: #{$Black-Grey};
    --color-zebra: #{$Black-Grey};
    --color-link: #{$LightBlue};
    --color-icon: #{$White};
    --color-border: #{$Dark-Grey};
    --color-highlight: #{$Black-Highlight};
    --site-owner: "DarkWolf_V";
    --theme-icon: url("/Images/Icons/moon.svg")
}
```

When styling elements you need to evaluate the variables defined above.

```scss
background-color: var(--color-page-background);
```

A small easter egg I added was to use my alias 'DarkWolf_V' when using the dark theme, this is done by using 'content' as a variable.

```css
content: var(--site-owner);
```

SVG icons look great however, they require special CSS properties to adjust the colour. Apply a 'mask' and background colour does the trick.

```css
-webkit-mask-image: url("/Images/Icons/chevron.svg");
mask-image: url("/Images/Icons/chevron.svg");
background-color: var(--color-icon);
```

## User Preference

As a bonus feature we can set the optimal user theme based on their preferences. We can update our logic from before to redefine the `DOMContentLoaded` event listener. We follow an order of precedence here starting with the default (light theme), the local storage and finally the user preference.

```javascript
    // set theme prioritising local storage before preferred theme.
    window.addEventListener('DOMContentLoaded', (event) => {
        const localStorageTheme = window.localStorage.getItem("theme");
        let siteTheme  = "light";

        if (localStorageTheme) {
            siteTheme = localStorageTheme;
        } else if(window.matchMedia) {
            siteTheme = window.matchMedia('(prefers-color-scheme: dark)').matches ? "dark" : "light";
        }

        document.body.className = siteTheme;
    });
```

We would also want to detect theme changes. Personally, my setup alternates between dark and light depending on the time of day. We therefore need to consider both their preferred scheme changing and their local storage settings. Despite their preferences, if they are visiting the site and would like to switch themes, it should be allowed.

```javascript
    // update theme if preference changes.
    window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', e => {
        // ignore theme updates when theme set in local storage.
        if (window.localStorage.getItem("theme")) {
            return;
        }
        const newColorScheme = e.matches ? "dark" : "light";
        document.body.className = newColorScheme;
    });
```

This is everything you need to support a dark/light theme for your website. I created this article for my [website](https://kaelanreece.com){target=__blank} when I realised there were no obvious ways for me to implement this using pure razor pages on the server side, too achieve this I needed to add some JavaScript. Similar logic to what I have described in this article will be applicable regardless of whether you are using an SPA, statically or server side rendered application. I hope you found this useful.
