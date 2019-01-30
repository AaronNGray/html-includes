# HTMLIncludes

Based on the polyfill for [HTMLImports](https://www.w3.org/TR/html-imports/) https://github.com/webcomponents/html-imports.

The polyfill hosts the included documents in the include link element. E.g.

```html
<link rel="include" href="my-element.html">

<!-- becomes -->

<link rel="include" href="my-element.html">
  <!-- my-element.html contents -->
</link>
```

The polyfill fires the `HTMLIncludesLoaded` event when imports are loaded, and exposes the `HTMLImports.whenReady` method. This api is necessary because unlike the native implementation, script elements do not force imports to resolve. Instead, users should wrap code in either an `HTMLIncludesLoaded` handler or after load time in an `HTMLIncludes.whenReady(callback)` call.

The polyfill provides the `HTMLIncludes.includeForElement()` method which can be used to retrieve the `<link rel=include>` that included an element.

## Caveats / Limitations

### `<link>.include` is not a `Document`

The polyfill appends the included contents to the `<link>` itself to leverage the native implementation of [Custom Elements](https://www.w3.org/TR/custom-elements), which expects scripts upgrading the `CustomElementRegistry` to be connected to the main document.

As a consequence, `.ownerDocument` will be the main document, while `.parentNode` of the included children will be the `<link rel=include>` itself. Consider using `HTMLIncludes.includeForElement()` in these cases. e.g:

```javascript
const ownerDoc = HTMLIncludes.includeForElement(document.currentScript);
let someElement = ownerDoc.querySelector('some-element');
if (ownerDoc !== HTMLIncludes.includeForElement(someElement)) {
  // This element is contained in another import, skip.
  someElement = null;
}
```

If you require document isolation, use [`html-imports#v0`](https://github.com/webcomponents/html-imports/tree/v0).

### Dynamic imports

The polyfill supports dynamically added imports by observing mutations in `<head>` and within other imports; it won't detect imports appended in `<body>`.

If you require to append includes in `<body>`, notify the polyfill of these additions using the method `HTMLIncludes.loadIncludes(document.body)`.

### Imoported stylesheets in IE/Edge

In IE/Edge, appending `<link rel=stylesheet>` in a node that is not `<head>` breaks the cascading order; the polyfill checks if an import contains a `<link rel=stylesheet>`, and moves all the imported `<link rel=stylesheet>` and `<style>` to `<head>`. It drops a placeholder element in their original place and assigns a reference to the applied element, `placeholder.__appliedElement`. e.g.

`my-element.html` imports a stylesheet and applies a style:

```html
<link rel="stylesheet" href="my-linked-style.css">
<style> .blue { color: blue }; </style>
```

And is imported in index.html:

```html
<head>
  <link rel="import" href="my-element.html">
</head>
```

This is how the resolved import will look like:

```html
<head>
  <link rel="stylesheet" href="my-linked-style.css">
  <style> .blue { color: blue }; </style>
  <link rel="include" href="my-element.html">
    <link type="include-placeholder">
    <style type="include-placeholder"></style>
  </link>
</head>
```

The placeholders contain a reference to the applied element:

```javascript
var myInclude = document.head.querySelector('link[rel=include]').include;
var link = myInclude.querySelector('link');
console.log(link.__appliedElement || link);
var style = myInclude.querySelector('style');
console.log(style.__appliedElement || style);
```

## Building & Running Tests

### Build

```bash
$ git clone https://github.com/AaronNGray/html-includes.git
$ cd html-includes
$ npm i
$ bower i
$ gulp
```

### Run tests

```bash
$ npm i -g web-component-tester
$ wct
```
