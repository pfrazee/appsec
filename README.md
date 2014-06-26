Appsec
======

A collection of Web security tools. **Not ready for people to use, seriously: don't use this.**

 - HTML sanitation with Google Caja (todo)
 - CSS sanitation with Rework (todo)
 - Web Worker script API nullification using closures and injection (todo)

Worker sandboxing:

```js
appsec.workerPolicy('import', ['my-worker-lib.js']);
appsec.workerPolicy('whitelist', [
  // defined by my-worker-lib.js
  'doThingA', 'doThingB',

  // allowed default APIs
	'null', 'self', 'console', 'atob', 'btoa',
	'setTimeout', 'clearTimeout', 'setInterval', 'clearInterval',
	'postMessage', 'addEventListener', 'removeEventListener',
	'onmessage', 'onerror', 'onclose'
]);

myWorkerScript = appsec.sandboxWorkerScript(myWorkerScript);
var workerObjUrl = window.URL.createObjectURL(new Blob([myWorkerScript], { type: "text/javascript" }));
var worker = new Worker(workerObjUrl);
// ^ will only be able to make calls in the above whitelist
```

HTML/CSS sanitization:

```js
appsec.htmlPolicy('tags', function(tagName) {
	return (tagName == 'p' || tagName == 'span');
});
appsec.htmlPolicy('urls', function(url) {
	return (url.indexOf('https://myhost.com') === 0) ? url : false; // only URLs on my host
});
appsec.htmlPolicy('classes', function(class) {
	return (class == 'float-left' || class == 'float-right');
});
appsec.cssPolicy('properties', function(property) {
	return (property == 'text-decoration' || property == 'font-weight' || property == 'font-style');
});

html = appsec.sanitizeHTML(html);
// ^ will only include tags, urls, classes, names, and styles in the above policies
```


### API

**Sandboxing**

 - `new appsec.sandboxWorkerScript(src, opts)` Creates a Worker with APIs nullified
   - `src` (string): JS script string (not the URL)
   - `opts.imports` ([string]): JS files to auto-import before loading
   - `opts.whitelist` ([string]): APIs to allow

**Sanitation**

 - `appsec.sanitizeCSS(styles, opts)` CSS sanitation with Rework.
   - `opts.prefix` (string): Sets an id prefix to CSS selectors
   - `opts.properties` (function): Style properties filter
   - `opts.values` (function): Style values filter
 - `appsec.sanitizeHTML(html, opts)` HTML sanitation with Caja
   - Calls `sanitizeCSS` on any styles in the HTML
   - `opts.prefix` (string): Sets an id prefix to CSS selectors
   - `opts.tags` (function): Element-types filter
   - `opts.urls` (function): URLs filter (for images, scripts, iframes, etc). Return false to disallow, or return an alternative URL.
   - `opts.classes` (function): Classes filter
   - `opts.names` (function): Names filter
   - `opts.cssProperties` (function): Style properties filter
   - `opts.cssValues` (function): Style values filter
 - `appsec.sanitizeText(text)`: Make text safe to put in the page (strips any HTML effects)

**Config**

 - `appsec.workerPolicy(name, value)` Sets a global worker policy
   - Policies:
     - `import` ([string]): JS files to auto-import before loading all workers
     - `whitelist` ([string]): APIs to allow in all workers
 - `appsec.htmlPolicy(name, value)` Sets a global html-sanitation policy
   - Policies:
     - `tags` (function): Element-types filter
     - `urls` (function): URLs filter (for images, scripts, iframes, etc). Return false to disallow, or return an alternative URL.
     - `classes` (function): Classes filter
     - `names` (function): Names filter
 - `appsec.cssPolicy(name, value)` Sets a global css-sanitation policy
     - `properties` (function): Style properties filter
     - `values` (function): Style values filter


### What's being done

**HTML Sanitation**

This is a wrapper around Google's Caja library, which parses the HTML and applies policy functions.

**CSS Sanitation**

Rework is used to parse the CSS. The parsed structure is filtered using the policy functions, then reformed as CSS.

**Worker API Nullification**

The worker source is wrapped in a closure. Before the closure, a script is injected which imports any specified JS files and nullifies non-whitelisted APIs with (roughly):

```js
var nulleds=[];
for (var k in self) {
	if (whitelist.indexOf(k) === -1) {
		Object.defineProperty(self, k, { value: null, configurable: false, writable: false });
		nulleds.push(k);
	}
}
```