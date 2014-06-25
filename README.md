Appsec
======

A collection of Web security tools. **Not ready for people to use, seriously: don't use this.**

 - HTML sanitation with Google Caja (todo)
 - CSS sanitation with Rework (todo)
 - Web Worker API nullification using script-injection (todo)

Worker sandboxing:

```js
appsec.policy('worker-import', ['my-worker-lib.js']);
appsec.policy('worker-whitelist', [
    // defined by my-worker-lib.js
    'doThingA', 'doThingB',

    // allowed default APIs
	'null', 'self', 'console', 'atob', 'btoa',
	'setTimeout', 'clearTimeout', 'setInterval', 'clearInterval',
	'postMessage', 'addEventListener', 'removeEventListener',
	'onmessage', 'onerror', 'onclose'
]);

var myWorker = new appsec.SandboxedWorker('foo.js');
// ^ will only be able to make calls in the above whitelist
```

HTML/CSS sanitization:

```js
appsec.policy('html-tags', function(tagName) {
	return (tagName == 'p' || tagName == 'span');
});
appsec.policy('html-urls', function(url) {
	return (url.indexOf('https://myhost.com') === 0) ? url : false; // only URLs on my host
});
appsec.policy('html-classes', function(class) {
	return (class == 'float-left' || class == 'float-right');
});
appsec.policy('css-properties', function(property) {
	return (property == 'text-decoration' || property == 'font-weight' || property == 'font-style');
});

var html = appsec.sanitizeHTML(html);
// ^ will only include tags, urls, classes, names, and styles in the above policies
```


### API

 - `new appsec.SandboxedWorker(src, opts)` Creates a Worker with APIs nullified
   - `src` (string): JS script string (not the URL)
   - `opts.imports` ([string]): JS files to auto-import before loading
   - `opts.whitelist` ([string]): APIs to allow
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
 - `appsec.policy(name, value)` Sets a global security policy
   - Policies:
     - `worker-import` ([string]): JS files to auto-import before loading all workers
     - `worker-whitelist` ([string]): APIs to allow in all workers
     - `html-tags` (function): Element-types filter
     - `html-urls` (function): URLs filter (for images, scripts, iframes, etc). Return false to disallow, or return an alternative URL.
     - `html-classes` (function): Classes filter
     - `html-names` (function): Names filter
     - `css-properties` (function): Style properties filter
     - `css-values` (function): Style values filter


### What's being done

**HTML Sanitation**

This is a wrapper around Google's Caja library, which parses the HTML and applies policy functions.

**CSS Sanitation**

Rework is used to parse the CSS. The parsed structure is filtered using the policy functions, then reformed as CSS.

**Worker API Nullification**

The worker source is wrapped in a closure. Before the closure, a script is injected which imports any specified JS files and nullifies non-whitelisted APIs with:

```js
(function() {
	var nulleds=[];
	for (var k in self) {
		if (whitelist.indexOf(k) === -1) {
			Object.defineProperty(self, k, { value: null, configurable: false, writable: false });
			nulleds.push(k);
		}
	}
	if (typeof console != "undefined") { console.log("Nullified: "+nulleds.join(", ")); }
})();
```

The source is then converted to a blob URI and loaded into a worker.