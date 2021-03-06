![minified size](http://img.badgesize.io/iconfu/svg-inject/master/dist/svg-inject.min.js?label=minified%20size&v=2) ![gzip size](http://img.badgesize.io/iconfu/svg-inject/master/dist/svg-inject.min.js?compression=gzip&v=2) [![npm version](https://badge.fury.io/js/%40iconfu%2Fsvg-inject.svg?v=2)](https://badge.fury.io/js/%40iconfu%2Fsvg-inject)



# SVGInject

A tiny, intuitive, robust, caching solution for injecting SVG files inline into the DOM.

Developed and maintained by [INCORS](http://www.incors.com), the creators of [iconfu.com](https://www.iconfu.com).


## What does it do?

SVGInject replaces an `<img>` element with an inline SVG. The SVG is loaded from the URL specified in the `src` attribute of the `<img>` element.

Element b​efore injection:

```html
<img src="image.svg" onload="SVGInject(this)" />

```

Element after injection (SVG loaded from image.svg):

```html
<svg version="1.1" ...> ... </svg>
```


## Why should I use it?

An SVG can only be properly styled with CSS and accessed with Javascript on element level if the SVG is inline in the DOM. With SVGInject you can keep your SVGs as individual files, but still access them with CSS and Javascript.


## How to install SVGInject?

### Manually 

Include the SVGInject Javascript file in the `<head>` element of your HTML document

```html
<head>
  ...
  <script src="svg-inject.min.js"></script>
  ...
</head>
```

Download plain version (v1.0.5): [svg-inject.js](https://raw.githubusercontent.com/iconfu/svg-inject/v1.0.3/dist/svg-inject.js)

Download minified version (v1.0.5): [svg-inject.min.js](https://raw.githubusercontent.com/iconfu/svg-inject/v1.0.3/dist/svg-inject.min.js)

### npm

If you are using [npm](https://www.npmjs.com)

```console
$ npm install @iconfu/svg-inject
```

### Yarn

If you are using [Yarn](https://yarnpkg.com)

```console
$ yarn add @iconfu/svg-inject
```


## Basic usage

Add `onload="SVGInject(this)"` to any `<img>` element where you want the SVG to be injected

**Example:**

```html
<html>
<head>
  <script src="svg-inject.min.js"></script>
</head>
<body>
  <img src="image.svg" onload="SVGInject(this)" />
</body>
</html>
```

**That's all: The SVG gets injected and is styleable!!!**

:sparkles: :sparkles: :sparkles:


<hr>


## What are the advantages?

* **Wide browser support**: Works on all browsers supporting SVG. Yes, this includes Internet Explorer 9 and higher! ([full list](https://caniuse.com/#feat=svg))

* **Fallback without Javascript**: If Javascript is not available the SVG will still show. It's just not styleable with CSS. 

* **Fallback if image source is not available**: Behaves like a normal `<img>` element if file not found or not available.

* **Prevention of ID conflicts**: IDs used internally in the SVG are made unique before injection to prevent ID conflits in the DOM


## What are additional advantages when using `onload`?

The recommended way to trigger injection is to call `SVGInject(this)` inside the `onload` attribute:

```html
<img ... onload="SVGInject(this)" /> 
```

This provides additional advantages:

* **Intuitive usage**: Insert SVG images into HTML code just as PNG images, with only one additional instruction. It's very clear to understand what it does looking at the pure HTML.

* **Works with dynamic content**: If `<img>` elements are added dynamically, injection still works.

* **Built-in prevention of unstyled image flash**: SVGInject hides `<img>` elements until injection is complete, thus preventing a brief flicker of the unstyled image (called [unstyled image flash](#how-does-svginject-prevent-unstyled-image-flash)).

* **Early injection**: The injection can already start before the DOM content is fully loaded.

* **Standard-conform**: The `onload` event handler on `<img>` elements has long been supported by all browsers and is officially part of the W3C specification since [HTML5.0](https://www.w3.org/TR/html50/webappapis.html#event-handler-attributes).


If you do not want to use the `onload` attribute but prefer to inject SVGs directly from Javascript, you can find more information [here](#how-to-use-svginject-directly-from-javascript).


## What are the limitations?

SVGInject is intended to work in production environments however it has a few limitations you should be aware of:

* The image source must conform to the [same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy), which basically means the image origin must be were the website is running. This may be bypassed using the [Cross-Origin Resource Sharing (CORS) mechanism](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
* Due to the same-origin policy SVGInject does not work when run from the local file system in many browsers (Chrome, Safari), yet in Firefox it works.


## How are attributes handled?

All attributes are copied from the `<img>` element to the injected `<svg>` element before injection with the following exceptions:

* The `src`, `title`, `alt`, `onerror` and `onload` attributes are not copied
* The `title` attribute is transformed to a `<title>` element and inserted as the first child element of the injeted SVG. If there is an existing `<title>` element as first child element it gets replaced.

You can disable the previously described attribute handling by setting the `copyAttributes` option to `false`. You may also implement your own attribute handling in the `beforeInject` options hook.

Additionaly after loading the SVG, the value of the `src` attribute of the `<img>` element is transformed to an absolute URL and inserted as a `data-inject-url` attribute. 

## API

| Function | Description |
|----------|-------------|
| SVGInject(img, options) | Injects the SVG specified in the `src` attribute of the specified `img` element or array of elements. The optional second parameter sets the [options](#options) for this injection. |
| SVGInject.setOptions(options) | Sets the default [options](#options) for SVGInject. |
| SVGInject.err(img, fallbackSrc) | Used in `onerror` event of an `<img>` element to handle cases when loading of the original source fails (for example if the file is corrupt or not found or if the browser does not support SVG). This triggers a call to the option's `onFail` hook if available. The optional second parameter will be set as the new `src` attribute for the `img` element. |


### Options

| Property name | Type | Default | Description |
| ------------- | ---- | ------- | ----------- |
| cache | boolean | `true` | If set to `true` the SVG will be cached using the absolute URL. The cache only persists for the lifetime of the page. Without caching images with the same absolute URL will trigger a new XMLHttpRequest but browser caching will still apply. |
| copyAttributes | boolean | `true` | If set to `true` [attributes will be copied](#how-are-attributes-handled) from the `img` to the injected `svg` element. You may implement your own method for copying attributes in the `beforeInject` options hook. |
| makeIdsUnique | boolean | `true` | If set to `true` the id of elements in the `<defs>` element that can be references by property values (for example 'clipPath') are made unique by appending the string "--inject-X", where X is a running number which increases with each injection. This is done to avoid duplicate ids in the DOM. |
| afterLoad | function(svg) | `undefined` | Hook after SVG is loaded. The loaded `svg` element is passed as a parameter. If caching is active this hook will only get called once for injected SVGs with the same absolute path. Changes to the `svg` element in this hook will be applied to all injected SVGs with the same absolute path. |
| beforeInject | function(img,&nbsp;svg) | `undefined` | Hook directly before the SVG is injected. The `img` and `svg` elements are passed as parameters. The hook is called for every injected SVG. If an [Element](https://developer.mozilla.org/de/docs/Web/API/Element) is returned it gets injected instead of applying the default SVG injection. |
| afterInject | function(img,&nbsp;svg) | `undefined` | Hook after SVG is injected. The `img` and `svg` elements are passed as parameters. |
| onFail | function(img,&nbsp;status) | `undefined` | Hook after injection fails. The `img` element and a `status` string are passed as an parameter. The `status` has one of the values: `'SVG_NOT_SUPPORTED'` - the browser does not support SVG, `'SVG_INVALID'` - the SVG is not in a valid format or `'LOAD_FAIL'` -loading of the SVG failed. <br> <br> If SVGInject is used with the `onload` attribute, `onerror="SVGinject.err(this);"` must be added to the `<img>` element to make sure `onFail` is called. |


## How does SVGInject prevent "unstyled image flash"

Before an SVG is injected the original unstyled SVG may be displayed for a brief moment by the browser. If a style is already applied to the SVG at runtime, the styled SVG will look different from the unstyled SVG, causing a brief “flashing” of the unstyled SVG before injection occurs. We call this effect unstyled image flash.

If SVGInject is used with the `onload` attribute, SVGInject has a built-in functionality to prevent unstyled image flash. A `<style>` element with one CSS rule is added to the document to hide all injectable `<img>` elements until injection is complete.

When [using Javascript directly](#how-to-use-svginject-directly-from-javascript) SVGInject has no build in functionality to prevent unstyled image flash. You can find a custom solution for this in the [example for using SVGInject without the `onload` attribute](#example-without-using-the-onload-attribute).


## How to use SVGInject directly from Javascript?

Instead of using the `onload` attribute on the `<img>` element you can also call SVGInject directly from Javascript.

**Examples:**
```javascript
// inject by class name
SVGInject(document.getElementsByClassName('myClassName'));

// inject by id
SVGInject(document.getElementById('myId'));
```

You need to make sure the images are already inside the DOM before injection like this:

```javascript
document.addEventListener('DOMContentLoaded', function() {
  // call SVGInject() from here
});
```

If you dynamically insert new `<img>` elements you need to call `SVGInject()` on these elements after their insertion.


## Fallback for no SVG support (IE <= 8)

If the browser does not support SVG, this simple fallback solution replaces the `src` attribute with an alternative image URI.

```html
<​img​ ​src​=​"image.svg"​ ​onload​=​"SVGInject(this)"​ ​onerror​=​"SVGInject.err(this, 'image.png')" /​>
```

A more generic method that will attempt to load a file with the same path and name but a different file extension can be found in [this example](#example-with-fallback-for-ie8--ie7).

Note that the `onerror="SVGInject.err(this)"` is necessary if SVGInject is used with the `onload` attribute,​ because the `onFail` callback will only get triggered this way.


## What about some examples?

Here are some examples which cover the most common use cases.

### Basic Example

This is the standard usage of SVGInject which works on all modern browsers, and IE9+.

```html
<html>
<head>
  <script src="svg-inject.min.js"></script>
</head>
<body>
  <img src="image.svg" onload="SVGInject(this)" />
</body>
</html>
```

### Example with fallback for IE8 & IE7

This example shows how to add a fallback for browsers not supporting SVGs. For these browsers an alternative PNG source is used to display the image.

```html
<html>
<head>
  <script src="svg-inject.min.js"></script>
</head>
<body>
  <img src="image.svg" onload="SVGInject(this)" onerror="SVGInject.err(this, 'image.png')" />
</body>
</html>
```

Another, more generic way of providing a fallback image source is using the `onFail` hook. If loading the SVG fails, this will try to load a file with the same name except “png” instead of “svg” for the file ending.

```html
<html>
<head>
  <script src="svg-inject.min.js"></script>

  <!-- optional PNG fallback if SVG is not supported (IE <= 8) -->
  <script>
    SVGInject.setOptions({
      onFail: function(img, status) {
        if (status == 'SVG_NOT_SUPPORTED') {
          img.src = img.src.slice(0, -4) + ".png";
        }
      }
    });
  </script>
</head>
<body>
  <!-- the onerror="SVGInject.err(this)" is needed to trigger the onFail callback -->
  <img src="image.svg" onload="SVGInject(this)" onerror="SVGInject.err(this)" />
</body>
</html>
```

### Example using `options`

This example shows how to use SVGInject with all available options.

```html
<html>
<head>
  <script src="svg-inject.min.js"></script>

  <script>
    SVGInject.setOptions({
      cache: false, // no caching
      copyAttributes: false, // do not copy attributes from `<img>` to `<svg>`
      makeIdsUnique: false, // do not make ids used within the SVG unique
      afterLoad: function(svg) {
        // add a class to the svg
        svg.classList.add('my-class');
      },
      beforeInject: function(img, svg) {
        // wrap SVG in a div element
        var div = document.createElement('div');
        div.appendChild(svg);
        return div;
      }, 
      afterInject: function(img, svg) {
        // set opacity
        svg.style.opacity = 1;
      },
      onFail: function(img) {
        // set the image background red
        img.style.background = 'red';
      }
    });
  </script>
</head>
<body>
  <img src="image.svg" onload="SVGInject(this)" onerror="SVGInject.err(this)" />
</body>
</html>
```

### Example without using the `onload` attribute

This example shows how to use SVGInject directly from Javascript without the `onload` attribute. After the DOM content has loaded, SVGInject is called on all elements with class `prevent-unstyled-image-flash`. It also implements a method to prevent [unstyled image flash](#how-does-svginject-prevent-unstyled-image-flash).


```html
<html>
<head>
  <script src="svg-inject.min.js"></script>

  <!-- hide images until injection has completed or failed -->
  <style>
    /* hide all img elements until the svg is injected to prevent "unstyled image flash" */
    img.prevent-unstyled-image-flash {
      visibility: hidden;
    }
  </style>

  <script>
    SVGInject.setOptions({
      onFail: function(img, svg) {
        // if injection fails show the img element
        img.classList.remove('prevent-unstyled-image-flash');
      }
    });

    document.addEventListener('DOMContentLoaded', function() {
      // inject images with an .svg file ending
      SVGInject(document.querySelectorAll('img[src$=".svg"]'));
    });
  </script>
</head>
<body>
  <img src="image_1.svg" class="prevent-unstyled-image-flash" />
  <img src="image_2.svg" class="prevent-unstyled-image-flash" />
</body>
</html>
```


## Browser support

Full support for all modern browsers, and IE9+ ([full list](https://caniuse.com/#feat=svg))

Support for IE8,IE7 with optional [PNG fallback method](#fallback-for-no-svg-support-ie--8)


## License

[MIT License](https://github.com/iconfu/svg-inject/blob/master/LICENSE)
