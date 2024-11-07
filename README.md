# Guide-to-develop-and-execute-modular-JavaScript-in-Obsidian
Taken from https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339


Whereas Markdown can only be used to produce static content, JavaScript **expands the possibilities of Obsidian** by enabling dynamic rendering. Its impact on performance is low and it reduces the need for additional plugins to add features. Furthermore, if the code is properly structured into JavaScript functions and externalized into external scripts, it **preserves the interoperability of markdown files** in case that you wish to migrate away from Obsidian. Indeed, it become unnecessary to go through the markdown files to modify and adapt them, as all you have to do is re-implements the few JavaScript functions located in the script files. It is these aspects that we are going to explore in this guide.

Here are a few examples of renderings created using JavaScript scripts with their call code from Markdown pages, which would be tedious to produce in pure Markdown:

-   `Categories of the page` (inspired by Wikipedia, I use it as a footer on all my pages)

```javascript
await customJS.MacroCategoriesOfPage.listInBox(<dataview-plugin>, "<path-to-page>");
```

![example-1](https://forum.obsidian.md/uploads/default/original/3X/4/e/4e277e63fcb4d6c7ba55568d4337b67942b02b42.png)

-   `Navigation bar` ([inspired by Wikipedia 1](https://fr.wikipedia.org/wiki/Projet:Portail_et_projet/Onglets), I use it as a page header on some pages)

```javascript
await customJS.MacroNavigationBar.forWelcomePage(<dataview-plugin>);
```

![example-2](https://forum.obsidian.md/uploads/default/original/3X/d/4/d4d9dfd20d7257cab638cc44f8dccff0665e7b34.png)

-   `Dynamic link` (the link target a local file located outside Obsidian and its root varies depending on the OS, which is impossible to do in markdown)

```javascript
customJS.MacroSysinfoLink.onTextToFile("<link-text>", "partial-path-to-file", "<file-format-indicator>")
```

![example-3](https://forum.obsidian.md/uploads/default/original/3X/1/6/1619a058c65e0517a441f02cdea4e93a556e03f9.png)

-   `Main article` ([inspired by Wikipédia](https://en.wikipedia.org/wiki/Template:Category_main_article))

```javascript
await customJS.MacroMainArticle.displayInBox(<plugin-dataview>, "<path-to-page>", "<link-text>");
```

![example-4](https://forum.obsidian.md/uploads/default/original/3X/b/1/b16b951d3fe616fc89eb236b5807187ef9113546.png)

## [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-introduction-1)Introduction

To structure JavaScript code in Obsidian and avoid polluting markdown files, a good practice is to externalise it in interconnected script files, and to form object classes representing conceptual entities of which only certain functions will be called in the pages. These classes will be of two types:

-   **modules** : not intended to be used directly in markdown files, these classes encapsulate the business logic and make their service functions available to macros.
-   **macros** : these classes act as an interface between the modules and the markdown files, and their main role is to render in HTML. Their functions are invoked in the pages using the [CustomJS plugin 47](https://github.com/saml-dev/obsidian-custom-js).

In this guide, we will see how to run JavaScript code inside Obsidian Markdown files, and how to set up an Obsidian macro system to externalise code into dedicated script files and manage their dependencies. In addition, to make it easier to write this code, we will see how to set up a development environment.

The proposed **macro system** is not technically revolutionary. It is essentially a method of organisation for Obsidian that aims to standardise and externalise your JavaScript functions, in order to reduce code redundancy and minimise maintenance time. It is directly inspired by the [Wikipedia module and model system 3](https://en.wikipedia.org/wiki/Wikipedia:Templates).

### [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-prerequisites-2)Prerequisites

This guide is written for people who are already somewhat familiar with Obsidian and who have some knowledge of JavaScript and software development. You don’t need to be a software engineer, but at least have already developed a little in JavaScript and optionally know what an [IDE](https://en.wikipedia.org/wiki/Integrated_development_environment) is and in particular [Visual Studio Code 1](https://code.visualstudio.com/).

For this guide you **need to install** :

-   the Obsidian [_Dataview_ 7](https://blacksmithgu.github.io/obsidian-dataview/) plugin for executing JavaScript in markdown pages,
-   the Obsidian [_CustomJS_ 47](https://github.com/saml-dev/obsidian-custom-js) plugin for externalizing JavaScript code and calling it.

If you want to configure a development environment, you need to **optionally install** :

-   the IDE [Visual Studio Code 1](https://code.visualstudio.com/) IDE.
-   the environment [NodeJS 5](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) (optional but recommended to take advantage of the auto-completion of the Obsidian API in the IDE).

## [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-executing-javascript-code-in-obsidian-3)Executing JavaScript code in Obsidian

The Dataview plugin provides [two syntaxes 2](https://blacksmithgu.github.io/obsidian-dataview/queries/dql-js-inline/#dataview-js) for embedding JavaScript in a markdown file.

The **first syntax** produces a rendering in a dedicated block (a `div`) that is separated from the text before and after it:

````markdown
```dataviewjs // here my code ```
````

For example, the following markdown code :

```markdown
Text before. ```dataviewjs dv.span("You read the page: " + dv.fileLink(dv.current().file.path, false, "Guide for Obsidian")) + "."; ``` Text after.
```

Display:

![dataview-block](https://forum.obsidian.md/uploads/default/original/3X/7/c/7cae7d897d7bf279b86d67acc90d65faa8c8063a.png)

The **second syntax** is called _inline_ and produces a rendering within the text before and after :

```markdown
`$= // here my code`
```

For example, the following markdown code :

```markdown
Text before. `$= "You read the page: " + dv.fileLink(dv.current().file.path, false, "Guide for Obsidian") + ".";` Text after.
```

Display:

![dataview-inline](https://forum.obsidian.md/uploads/default/original/3X/a/8/a884b92453bd95042049ca0e60d7d0146513d416.png)

The **code is executed when you switch to _Live preview_ or _Reading_** view. It is therefore recommended to write it in _Source mode_.

For performance reasons, the code is not reinterpreted each time the view is changed, but at regular intervals or when modifications are made.

> **Tip**  
> To force reinterpretation of the code without modifying it, add the `;` character to the end of a line. This character is optional in JavaScript and can be found in multiple copies in a row without causing any problems. Then, when your tests are complete, delete the excess characters.

## [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-externalise-javascript-code-in-script-files-4)Externalise JavaScript code in script files

Writing all the JavaScript code only in markdown files will quickly make your files unreadable, and above all, make code maintenance difficult and lead to a lot of redundancy. That’s why it’s recommended that you move it to script files located in a subfolder of the Obsidian vault, then invoke them in the markdown pages using the [CustomJS plugin 47](https://github.com/saml-dev/obsidian-custom-js).

The **approach to outsourcing code** is as follows:

1.  In Obsidian, create a sub-folder for the JavaScript files. For example, in my case it’s `03-Files/scripts_customjs`, next to my other assets.

![folder_hierarchy](https://forum.obsidian.md/uploads/default/original/3X/b/d/bd7c6985d670433c46871f84919ef22deff2f393.png)

2.  In the Obsidian options, go to the CustomJS configuration parameters.
    
3.  Initialise the _Folder_ parameter with the path to the folder created earlier (for example `03-Files/scripts_customjs`). Close the options window.
    
4.  Using File Explorer, open this folder, then create a new file suffixed with the `.js` extension. For example `my-script.js`. The filename _must not_ contain any spaces or special characters.
    
5.  Using a text editor or code-editing software, open this file and paste the following demonstration code:
    

```javascript
class MacroGuide { /** * @param {Object} dv DataView object of Obisidian extension. */ helloWorld(dv) { const { obsidian, app } = self.customJS || {}; if (obsidian == null || app == null) throw new Error("customJS is null."); dv.span( "You read the page: " + dv.fileLink(dv.current().file.path, false, "Guide for Obsidian") ) + "."; } }
```

6.  Save and close the text editor.
    
7.  In an Obsidian page, insert the following script invocation code:
    

````javascript
```dataviewjs customJS.MacroGuide.helloWorld(dv); ```
````

8.  Switch to _Live Preview_ or _Reading_ to view the rendering. A sentence containing a link to the current page should be displayed. The rendering is identical to the first method in the previous part, but this time the business logic is externalized.

**A few comments:**

At step 6, you can see that the `helloWorld()` function is **encapsulated in a class**. This is a constraint imposed by CustomJS, which will trigger an error if it is not respected. In any way, even if this were not the case, this practice is strongly recommended to take advantage of the readability benefits offered by the Object paradigm.

Secondly, the CustomJS plugin gives access to [two objects 47](https://github.com/saml-dev/obsidian-custom-js) which are essential for interacting with Obsidian:

-   `customJS.obsidian`: provides access to the [Obsidian API 5](https://github.com/obsidianmd/obsidian-api/blob/master/obsidian.d.ts).
-   `customJS.app`: provides access to the [App object 3](https://docs.obsidian.md/Reference/TypeScript+API/App).

Finally, to **access to the Dataview object from a function**, you will need to pass it the global variable `dv` as a parameter.

## [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-configuring-the-development-environment-5)Configuring the development environment

**Note:** this part is optional. If you don’t want to work with an IDE, you can skip this part.

Writing JavaScript code in a simple text editor will quickly become tedious and be the source of many syntax errors without a checking tool. The advantage of an IDE such as [Visual Studio Code for JavaScript 2](https://code.visualstudio.com/docs/languages/javascript) is that it offers syntax highlighting, but above all **[auto-completion 1](https://en.wikipedia.org/wiki/Autocomplete) mechanisms** that will save you a huge amount of time.

The procedure for **configuring the development environment** is as follows:

1.  From a terminal, install the NodeJS package of [`Obsidian API` 11](https://www.npmjs.com/package/obsidian).
    
2.  Launch Visual Studio Code, then open your Obsidian vault folder.
    
3.  In the folder where the CustomJS scripts folder is already located, create a file named `jsconfig.json`. This allows you to [configure a JavaScript project in VS Code](https://code.visualstudio.com/docs/languages/jsconfig) to take advantage of auto-completion with the Obsidian API and JavaScript.
    
4.  Paste the following content into this file:
    

```json
{ "compilerOptions": { "module": "ESNext", "moduleResolution": "Bundler", "target": "ES2022", "jsx": "react", "allowImportingTsExtensions": true, "checkJs": true, "allowJs": true, "experimentalDecorators": true, "strictNullChecks": true, "strictFunctionTypes": true, "baseUrl": ".", "paths": { "obsidian": ["../../../../../.nvm/versions/node/v20.13.1/lib/node_modules/obsidian/obsidian.d.ts"], "custom-js": ["../.obsidian/plugins/customjs/types.d.ts"] } }, "exclude": [ "node_modules", "**/node_modules/*" ] }
```

5.  In the `obsidian` variable, check that the path points to the type definition in the Obsidian NodeJS package.

You should now have auto-completion when developing scripts. In the rest of the guide, we’ll be coming back to modify this file to add our scripts in order to have auto-completion on our own classes.

> **Suggestion**  
> Put the `// @ts-check` instruction at the beginning of each JavaScript file to detect syntax errors.

## [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-structuring-its-scripts-and-managing-dependencies-6)Structuring its scripts and managing dependencies

The [Decomposition](https://en.wikipedia.org/wiki/Decomposition_(computer_science)) is a fundamental principle in software engineering. When applied with an object language like modern JavaScript, it involves subdividing business domains into conceptual entities called classes. A class groups together and provides all the services expected of an entity via its functions (or methods). This is what this section proposes to put in place. This paradigm greatly improves the maintainability and readability of the code by avoiding a lot of code redundancy. On the other hand, it will mean having to manage dependencies between scripts.

A good engineering practice is to have **one object class per file**. For the record, there are two types of class in this macro system.

**Module classes**, which are not intended to be used directly in markdown files, and which encapsulate the business logic and provide macros with their service functions. And **Macro classes**, which act as an interface between the modules and the markdown files using the [CustomJS plugin 47](https://github.com/saml-dev/obsidian-custom-js), and whose main role is to render HTML files.

The **macros** are all located in the single folder configured in customJS:

![customjs-folder](https://forum.obsidian.md/uploads/default/original/3X/4/6/460241161d1dd47f996bef08c86e14074f669d6e.png)

**Modules** are grouped by business area in folders next to the macros folder. For example:

![folder_modules](https://forum.obsidian.md/uploads/default/original/3X/0/9/097cd0f68bca45e74049572e35e4fac08e67b034.png)

The **script import and export** is done using the [import/export mechanism 6](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) provided by the JavaScript API. Except that where the simple keywords `export/import` or `require` are sufficient in pure JavaScript, the permissiveness of the customJS plugin means that you have to use a roundabout method that we will see later.

### [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-naming-module-folders-7)Naming module folders

Module folders can be named anything as long as they **share a common prefix** for the import/export mechanism, and have no spaces or special characters.

I have chosen the `scripts_module_<domain>` convention.

### [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-centralizing-its-configurations-8)Centralizing its configurations

It may be a good idea to create a configuration script for scripts, which centralises the main information about the Obsidian vault in order to reduce the impact that any changes to the vault would have on the code. I recommend putting at least the paths to the main folders.

Its path could be `scripts_module_common/config.js` and its minimal structure :

```javascript
/** * @description Module for all config variables. * @module Config */ // @ts-check module.exports = { myVar: "my-value" };
```

Example of configuration file

```javascript
/** * @description Module for all config variables. * @module Config */ // @ts-check module.exports = { obsidianFolderPaths: { article: "01-Article", project: "02-Project", file: "03-Files", tags: "04-Tags", categories: "04-Tags/Categorie", portals: "04-Tags/Portal", template: "05-Template", templateSource: "05-Template/Source", special: "06-Special", user: "07-User", help: "08-Help", }, fsRootPaths: { linux: "/home/name/", // Root path on Linux File System windows: "%userprofile%/", // Root path on Windows File System android: "", // Root path on Android File System mac: "", // Root path on Mac File System ios: "", // Root path on iOS File System }, };
```

### [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-exporting-a-script-9)Exporting a script

To export, use the `module.exports = ...` syntax from [NodeJS 1](https://nodejs.org/api/modules.html#exports-shortcut), preferably at the end of the :

```javascript
class MyClass { // my code } module.exports = MyClass;
```

You can export several values by creating a JavaScript literal object:

```javascript
class MyClass { // my code } const MY_CONST = "value"; module.exports = { MyClass, MY_CONST, myVar: "value", }
```

Despite repeated attempts, the ES6 import/export syntax does not seem to be supported by the CustomJS plugin. Only the NodeJS syntax works.

### [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-importing-a-script-10)Importing a script

While in pure JavaScript import is easily done using the `import` or `require` keywords, CustomJS raises an error when using them. So you need to be a bit tricky and encapsulate the `require` function in another static function.

```javascript
/** * @param {string} moduleFolderSuffix Suffixe name of one of folder module in `/03-Files/scripts_module_<suffixe>/` folders. * @param {string} moduleFile Name of the file module in the `/03-Files/scripts_module_<suffixe>/` folder. * @returns {any} Exported module. */ static importModule(moduleFolderSuffix, moduleFile) { const { obsidian, app } = self.customJS || {}; if (obsidian == null || app == null) throw new Error("customJS is null."); let adapter = app.vault.adapter; if (adapter instanceof obsidian.FileSystemAdapter) { const modulePath = adapter.getBasePath() + "/03-Files/scripts_module_" + moduleFolderSuffix + "/" + moduleFile; delete global.require.cache[global.require.resolve(modulePath)]; return require(modulePath); } throw new Error("Obsidian adapter is not a FileSystemAdapter."); }
```

The `importModule()` function will automatically look for modules in the sub-folders of `03-Files` prefixed with `scripts_module_`. As this prefix was determined in the previous section, you need to remember to modify the code accordingly.

There are two ways to **integrate `importModule` into a file**:

-   Directly as a function of the main class.

```javascript
class MacroGuide { /* -------------------------------------------------------------------------- */ /* Import Section */ /* -------------------------------------------------------------------------- */ static importModule(moduleFolderSuffix, moduleFile) { // Code of function } /* -------------------------------------------------------------------------- */ /** * @param {Object} dv DataView object of Obisidian extension. */ helloWorld(dv) { // My code } }
```

-   Or in a specific class located before the main class, bearing in mind that this option **only works for classes located outside the CustomJS folder** (`scripts_customjs` in this article).

```javascript
/* -------------------------------------------------------------------------- */ /* Import Section */ /* -------------------------------------------------------------------------- */ class ModuleLoader { static importModule(moduleFolderSuffix, moduleFile) { // Code of function } } /* -------------------------------------------------------------------------- */ class ClassOfModule { /** * @param {Object} dv DataView object of Obisidian extension. */ helloWorld(dv) { // My code } }
```

**Using the `importModule()` function** to import a class is done in almost the same way as in pure JavaScript : by initializing a variable with a call to the function in the import section. This variable is then used in the same way as in pure JavaScript.

-   If you had chosen the first option :

```javascript
class MacroGuide { /* -------------------------------------------------------------------------- */ /* Import Section */ /* -------------------------------------------------------------------------- */ static importModule(moduleFolderSuffix, moduleFile) { // Code of function } Obsidian = MacroGuide.importModule( "common", "utility-obsidian.js" ); // Here I import the class UtilityObsidian presented in the appendix of this article. /* -------------------------------------------------------------------------- */ /** * @param {Object} dv DataView object of Obisidian extension. */ helloWorld(dv) { // To call a static function const page = this.Obsidian.getFileByPath(); // To instanciate a class const myObsidianObject = new this.Obsidian(); } }
```

-   If you had chosen the second option :

```javascript
/* -------------------------------------------------------------------------- */ /* Import Section */ /* -------------------------------------------------------------------------- */ class ModuleLoader { static importModule(moduleFolderSuffix, moduleFile) { // Code of function } } Obsidian = ModuleLoader.importModule( "common", "utility-obsidian.js" ); // Here I import the class UtilityObsidian presented in the appendix of this article. /* -------------------------------------------------------------------------- */ class ClassOfModule { /** * @param {Object} dv DataView object of Obisidian extension. */ helloWorld(dv) { // To call a static function const page = Obsidian.getFileByPath(); // To instanciate a class const myObsidianObject = new Obsidian(); } }
```

> **External packages**  
> It is possible to import any NodeJS module or package (`node:path`, `luxon`, etc.), provided it has already been imported by an Obsidian plugin. For that, use the NodeJS syntax directly. For example: `const path = require(‘node:path’);`.

### [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-configuring-the-ide-for-auto-completion-11)Configuring the IDE for auto-completion

**Note:** this part is optional and should only be followed if you have completed the previous part on configuring the IDE.

Because we cannot use the standard way of importing JavaScript objects, the IDE is unable to offer auto-completion. Alos, you should see red highlighting all over your code. To activate auto-completion, you need to perform a few operations, in particular with [JSDoc 1](https://jsdoc.app/).

For each file with an export instruction :

1.  Open the `jsconig.json` file, and indicate in the `path` variable an import alias and the path to the file to be imported. For example:

```json
{ "compilerOptions": { "paths": { "obsidian": ["../../../../../.nvm/versions/node/v20.13.1/lib/node_modules/obsidian/obsidian.d.ts"], "moment": ["../../../../../.nvm/versions/node/v20.13.1/lib/node_modules/moment/moment.d.ts"], "custom-js": ["../.obsidian/plugins/customjs/types.d.ts"], "module-category": ["scripts_module_tag/category.js"], "module-utility-obsidian": ["scripts_module_common/utility-obsidian.js"] } } }
```

2.  In the importing JavaScript file, add a [declaration `@typedef`](https://jsdoc.app/tags-typedef) at the top of the import section with the import alias specified in the previous step, in the form `/** @typedef {import('alias')} Type */`. For example :

```javascript
/* -------------------------------------------------------------------------- */ /* Import Section */ /* -------------------------------------------------------------------------- */ /** * @typedef {import('custom-js')} CustomJS * @typedef {import('obsidian')} Obsidian * @typedef {import('obsidian').App} App * @typedef {import('obsidian').Notice} Notice * @typedef {import('obsidian').TAbstractFile} TAbstractFile * @typedef {import('obsidian').TFile} TFile * @typedef {import('obsidian').TFolder} TFolder * @typedef {import('module-category')} Category */ // Declaration of `static importModule(moduleFolderSuffix, moduleFile){}` /* -------------------------------------------------------------------------- */
```

3.  In the same file, but this time at the end of the import section, add a [declaration `@type`](https://jsdoc.app/tags-type) of the form `/** @type {typeof import('alias')} */` above the variable initialisations by a call to the `importModule()` function. For example:

```javascript
/* -------------------------------------------------------------------------- */ /* Import Section */ /* -------------------------------------------------------------------------- */ /** * @typedef {import('custom-js')} CustomJS * @typedef {import('obsidian')} Obsidian * @typedef {import('obsidian').App} App * @typedef {import('obsidian').Notice} Notice * @typedef {import('obsidian').TAbstractFile} TAbstractFile * @typedef {import('obsidian').TFile} TFile * @typedef {import('obsidian').TFolder} TFolder * @typedef {import('module-category')} Category */ // Declaration of `static importModule(moduleFolderSuffix, moduleFile){}` /** @type {typeof import('module-utility-obsidian')} */ const Obsidian = ModuleLoader.importModule("common", "utility-obsidian.js"); /** @type {typeof import('module-category')} */ const Category = ModuleLoader.importModule("tag", "category.js"); /* -------------------------------------------------------------------------- */
```

After these steps you should no longer have red highlights and you should have auto-completion for your classes.

**Note:** it is not necessary to follow steps 2 and 3 for each import. Depending on whether the functions called are static or not, and whether the imported classes are instantiated or not, you should carry out one of the two steps (as for my `utility-obsidian.js` module) or sometimes both (as for my `module-category` module) depending on what the IDE indicates.

## [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-generating-html-with-javascript-12)Generating HTML with JavaScript

One of the main advantages of using JavaScript in Obsidian is the ability to dynamically generate HTML content. The easiest way to do this (and perhaps the only way without a plugin) is to use the [Dataview JavaScript API 1](https://blacksmithgu.github.io/obsidian-dataview/api/code-reference/).

> **Impact on performance**  
> While executing JavaScript for data processing has very little impact on performance, this is not the case for generating HTML rendering. It can very easily add a few seconds to the loading time of a page, depending on your hardware configuration. A drop in performance is particularly noticeable when you have to generate a list containing several hundred links.

### [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-the-fundamental-rendering-functions-13)The fundamental rendering functions

-   `dv.el(element, text) : HTMLElement` : render arbitrary text in the given html element (“p”, “ul”, “li”, etc.). This function return the rendered [HTMLElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement).

```javascript
dv.el("b", "This is some bold text");
```

-   `dv.el(element, text, { container: HTMLElement, cls: "first-css-class second-css-class", attr: { alt: "my value" } }) : HTMLElement;` : it is possible to specify a parent [HTMLElement container](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement) via `container`, a custom CSS classes to add to the element via `cls`, and additional attributes via `attr`. The parent container is the Dataview container `dv.container`.

```javascript
async renderList(dv) { // Add CSS class to Dataview div block. dv.container.className += " my-css-class"; // Render a list. const div = dv.el("div", "Here is my list: ", { container: dv.container, cls: "my-class-for-list", }); const ul = dv.el("ul", "", { container: div, }); ul.innerText = ""; // a "bug" into Dataview add an extra span everywhere when there is an empty string, here we remove it. for (let index = 0; index < 10; index++) dv.el("li", index, { container: ul, cls: "my-class-for-item" }); }
```

-   `dv.header(level, text)` : render a header of level 1 to 6 with the given text.

```javascript
dv.header(1, "Big!"); // alias of dv.el(h1, "Big!"); dv.header(6, "Tiny");// alias of dv.el(h6, "Tiny!");
```

-   `dv.paragraph(text)` : render arbitrary text in a paragraph.

```javascript
dv.paragraph("This is some text"); // alias of dv.el(p, "This is some text");
```

-   `dv.span(text)` : render arbitrary text in a span (no padding above/below, unlike a paragraph).

```javascript
dv.span("This is some text"); // alias of dv.el(span, "This is some text");
```

> **Recommendation**  
> Since the last three functions are simply aliases of the first and offer fewer options than the first, I recommend using only the `dv.el()` syntax.

It is **not necessary to use these Dataview functions** to render HTML. Instead, you can go directly to the [Obsidian API](https://docs.obsidian.md/Plugins/User+interface/HTML+elements) and its `createEl()` function, accessible from a [HTMLElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement), as soon as you have access to the main `dv.container` container:

```javascript
const book = dv.container.createEl("div"); book.createEl("div", { text: "How to Take Smart Notes" }); book.createEl("small", { text: "Sönke Ahrens" });
```

Dataview also uses this syntax for its `dv.el()` rendering function.

### [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-the-case-of-collections-and-lists-14)The case of collections and lists

Dataview provides several [functions for displaying data sets](https://blacksmithgu.github.io/obsidian-dataview/api/code-reference/#dataviews): `dv.list()`, `dv.table()` etc. These are very useful when you want to write JavaScript directly to Markdown files while limiting the number of lines of code, and its [Proxy system](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) also makes it very easy to manipulate the data contained in collections. However, there are at least two limitations.

The **first limitation** is that they have **a cost in terms of performance**, which is admittedly low in terms of rendering times, but which exists despite the fact that there is a data cache. It is therefore important to understand how this system works and to identify in which cases it is relevant or not.

To handle data sets, Dataview provides a [data structure called `DataArray`](https://blacksmithgu.github.io/obsidian-dataview/api/data-array/). Each set or subset is a `DataArray` object. You can see what this object looks like by opening the console with `CTRL` + `SHIFT` + `I`, then inserting and executing the following code in an Obsidian page:

````javascript
```dataviewjs console.log(dv.pages()); ```
````

This structure can be quite overkill in simple cases where, for example, you only need to list the pages present in a folder. Although Dataview’s caching system makes it easy to use, there are some cases where using the [Obsidian API 1](https://docs.obsidian.md/Reference/TypeScript+API) directly is more interesting. This is why you will find **in the appendix a utility class** which provides high-level functions **to interact with the Obsidian API**.

The **second limitation** of Dataview’s dataset handling features is the impossibility of customising the rendering with a CSS class or parent container other than `dv.container`. This greatly limits the possibilities of what can be done in HTML. The only alternative is to implement the lists yourself, as we did in the previous section.

## [](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339#p-287454-a-full-example-of-a-module-and-macro-15)A full example of a module and macro

The proposed example is a macro called `MacroTagsOfPages`, which displays in a frame the list of tags in the `tags` property of the page. It is inspired by Wikipedia footers (as can be seen, for example, on the [Obsidian page 1](https://en.wikipedia.org/wiki/Obsidian_(software))). This macro is based on the `UtilityObsidian` module, and its style is defined by the `.macro-tags-of-page` CSS class. **The full source code is provided in the appendix**.

![full-example](https://forum.obsidian.md/uploads/default/original/3X/1/1/11a40af71c9e18dcb63b008c449c246928015d16.png)

To **test this example in Obsidian** :

1.  In the customJS scripts folder (for example `scripts_customjs`), create a file called `macro-tags-of-pages.js`, which will contain the macro `MacroTagsOfPage`, then copy the corresponding source code provided in the appendix.
    
2.  Next to the CustomJS scripts folder, create a new folder called `scripts_module_common`.
    
3.  In this new folder, create a file called `utility-obsidian.js`, which will contain the `UtilityObsidian` module, then copy the corresponding source code provided in the appendix.
    
4.  In the Obsidian CSS snippets folder, create a new file called `macro.css` and copy the source code of the style provided in the appendix.
    
5.  In an Obsidian page containing some tags, add the following code to test the macro:
    

````javascript
```dataviewjs await customJS.MacroTagsOfPage.listInBox(dv, dv.current().file.path);; ```
````

Switching to _Live Preview_ or _Reading_ view should display the rendering.
