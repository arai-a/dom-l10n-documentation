# Using Fluent

This tutorial will show how to use fluent to localize the message in our `<h1>` element.

If you don't have a Parcel project set up with fluent files, please refer to the previous sections.

---

First, we need to add and import our Fluent dependencies.

For this tutorial we will install `fluent/bundle` and `fluent/dom` via the following command:

> **_terminal_**
> ```
> npm install @fluent/bundle @fluent/dom
> ```

And verify that the dependencies are added to your `package.json`.

> **_package.json_**
> ```json
> "dependencies": {
>   "@fluent/bundle": "^0.17.0",
>   "@fluent/dom": "^0.8.0",
>   ...
> }
> ```

Then we need to import some items from these dependencies in `index.js`.

> **_src/index.js_**
> ```JavaScript
> import { DOMLocalization } from "@fluent/dom";
> import { FluentBundle, FluentResource } from "@fluent/bundle";
> ```

---

Next, we need to add the metadata to the HTML regarding which languages we have available.

Add the following lines to the `<head>` section of your `index.html` file.

> **_src/index.html_**
> ```html
>     <meta name='defaultLanguage' content='en-US' />
>     <meta name='availableLanguages' content='en-US, es-MX' />
> ```

---

Now we are going to add a JavaScript function to `index.js` to retrieve this meta data.

> **_src/index.js_**
> ```JavaScript
> function getMeta(elem) {
>   return {
>     available: elem.querySelector('meta[name="availableLanguages"]')
>       .getAttribute("content")
>       .split(",").map(s => s.trim()),
>     default: elem.querySelector('meta[name="defaultLanguage"]')
>       .getAttribute("content"),
>   };
> }
> ```

---

Now that we are able to retrieve the metadata for which locales are supported, we need a way to resolve the paths to the Fluent resources that we created for those locales.

We are going to add the following line to the `<head>` section of the `index.html` file:

> **_src/index.html_**
> ```HTML
> <link name='localization' content='./localization/{locale}/main.ftl' />
> ```
> > **_src/index.html_**
> > ```HTML
> > <!DOCTYPE html>
> > <html>
> >   <head>
> >     <meta charset='UTF-8' />
> >     <meta name='defaultLanguage' content='es-MX' />
> >     <meta name='availableLanguages' content='en-US, es-MX' />
> >     <link name='localization' content='./localization/{locale}/main.ftl' />
> >     <script type='module' src='index.js'></script>
> >   </head>
> >   <body>
> >     <h1 id='welcome'>Localize me!</h1>
> >   </body>
> > </html>
> > ```

---

We will now need an asynchronous JavaScript function to fetch a `FluentResource` from a given path, replacing the `{locale}` template with an actual given locale.

> **_src/index.js_**
> ```JavaScript
> async function fetchResource(locale, resourceId) {
>   const url = resourceId.replace("{locale}", locale);
>   const response = await fetch(url);
>   const text = await response.text();
>   return new FluentResource(text);
> }
> ```

Once we are able to fetch a resource, we will want to be able to turn that resource into a `FluentBundle`.

> **_src/index.js_**
> ```JavaScript
> async function createBundle(locale, resourceId) {
>     let resource = await fetchResource(locale, resourceId);
>     let bundle = new FluentBundle(locale);
>     let errors = bundle.addResource(resource);
>     if (errors.length) {
>       // Syntax errors are per-message and don't break the whole resource
>     }
>     return bundle;
> }
> ```

Finally, we will want a top-level `generateBundles` function (remember, this is what `DOMLocalization` is initialized with) to provide a generator over the available fluent bundles for our supported locales.

> **_src/index.js_**
> ```JavaScript
> async function* generateBundles(resourceIds) {
>   const defaultLanguage = getMeta(document.head).default;
>   yield await createBundle(defaultLanguage, resourceIds[0]);
> 
>   const availableLanguages = getMeta(document.head).available;
>   for (const locale of availableLanguages) {
>     yield await createBundle(locale, resourceIds[0]);
>   }
> }
> ```

This function will first grab the bundle for the default language, and then go through the list of available languages. Yes, there is a redundancy here where the default language is also in the list of available languages. For simplicity, we'll leave it this way.

---

Out final steps are to grab the translatable `<h1>` element, our template path for the `resourceIDs`, and construct our `DOMLocalization` object to perform the translations.

> **_src/index.js_**
> ```JavaScript
> const welcome = document.getElementById("welcome");
> console.log(welcome);
> 
> const resources = document.querySelector("link[name=localization]");
> const resourcePathTemplate = resources.attributes.content.nodeValue;
> console.log(resourcePathTemplate);
> 
> const l10n = new DOMLocalization([resourcePathTemplate], generateBundles);
> l10n.connectRoot(document.documentElement);
> l10n.setAttributes(welcome, "hello");
> l10n.translateRoots();
> ```

With this final step, you should be able to invoke

> **_terminal_**
> ```
> npm start
> ```

and now see **Hello!**, as specified in our Fluent file for `en-US`. You should also be able to change the default language in `index.html` to `es-MX` and see **¡Hola!**.

---

Congratulations! You've just set up a project from scratch that uses Fluent DOM to localize HTML elements.

---

In the [next section](./using-l10n-id.md) we will cover how to use `data-l10n-id` directly to translate DOM elements without doing it manually from JavaScript.
