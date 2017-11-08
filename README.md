##### RED Interactive Agency - Ad Technology

Summary of attempts to use Web Components in the RED Ad Tech ecosystem:

## Abstract
Dynamic creative need modular views/functionality that can be instantiated on-demand -- like classes. 

Web Components would seem to be a good fit -- except, packaging Web Components is extremely problematic. This is due to their HTML complexity that may include inline and/or nested html, markup, styles, and javascript.

The spec-proposed way of dealing with these dynamics are:
- HTML Templates
- HTML Imports

## HTML Templates & Imports
These constructs are not well-supported in the Webpack community. https://github.com/webpack-contrib/html-loader/issues/14 This is likely because of the difficulty in packaging it. 

The challenge is that rendering DOM-strings is usually done with `element.innerHTML`. However, remember that `<script>` tags rendered as such will not execute. 

Then consider that `.html` files may include any combination of the following:

`<link rel="import" href="./relative-path-to-html" />`
- requires recursive parsing of content pages, while maintaining modularity
all of the loaders that were tested handled this differently (see below for a full list)

`<script type="text/javascript" src="./relative-path-to-js"></script>`
- easy enough to handle in a global scope, but not when bound to an HTML Template
 
`<script type="text/javascript" src="//absolute-path-to-js"></script>`
- does not execute when DOM-string is added to document via `element.innerHTML`

`<script> console.log('inline script tags'); </script>` 
- does not execute when DOM-string is added to document via `element.innerHTML`

`<template>`
- defers shadow content from executing (including child <script> and <link> tags)

CustomElements, like `<my-element />`
when added via `element.innerHTML` also fail to execute properly, even in Chrome
- https://github.com/webcomponents/webcomponentsjs/issues/459

The likelihood of these combinations is especially true for web-components, since each `.html` essentially is a page.


## Tested Packages
[wc-loader](https://github.com/aruntk/wc-loader)
This loader came the closest. It located all of the dependencies and it repackaged them in a way that worked. The final limitations included:
 - `<script>` tags loading absolute urls were not parsed out of the DOM-string, thus causing them not to function
 - browser limitations when trying to render `<my-component>` from a DOM-string

[html-loader](https://github.com/webpack-contrib/html-loader)
This standard webpack loader does a tremendous job of locating every type of dependencies and modularizing it. However, for the output to function as expected, another loader would be needed. I was not able to find one for this purpose.

[polymer-webpack-loader](https://github.com/webpack-contrib/polymer-webpack-loader)
The repo on this one looks promising, but it does not seem to work at all on nested `<link>` or `<script>` tags.

[polymer-build](https://github.com/Polymer/polymer-build)
If I were forced to continue on this path, I would next test the Html-splitting capabilities of this project. Theoretically one could add the dependencies it to the webpack graph, and then somehow reconstruct it. Gauging by these other attempts, I'd guess it very difficult and time-consuming.

## Workarounds
Web-components can be utilized if they are just CustomElement definitions declared by JS, and:
- their CustomElement definitions have been established before page render, and their markup is hard-coded into the DOM. 
- they are instantiated with `document.createElement` and have been authored to work that way
