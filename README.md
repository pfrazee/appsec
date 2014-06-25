Appsec
======

A collection of Web security tools

 - `new appsec.Worker(src, {imports:, whitelist:})` WebWorker API nullifying with script-injection (todo)
   - `imports ([string])`: JS files to auto-import before loading
   - `whitelist ([string])`: APIs to allow
 - `appsec.sanitizeCSS(styles, {prefix:})` CSS sanitation with Rework. (todo)
   - `prefix (string)`: sets an id prefix to CSS selectors
 - `appsec.sanitizeHTML(html, {prefix:})` HTML sanitation with Caja (todo)
   - `prefix (string)`: sets an id prefix to CSS selectors
 - `appsec.policy(name, value)` Sets a global security policy (todo)
   - Policies:
     - `worker-import ([string])`: JS files to auto-import before loading all workers
     - `worker-whitelist ([string])`: APIs to allow in all workers
     - `html-tags (function)`: Element-types filter
     - `html-urls (function)`: URLs filter (for images, scripts, iframes, etc)
     - `html-classes (function)`: classes filter
     - `html-names (function)`: names filter
     - `css-properties (function)`: style properties filter
     - `css-values (function)`: style values filter
