# Guide-to-develop-and-execute-modular-JavaScript-in-Obsidian

Taken from [an article on Obsidian forum by Milleras](https://forum.obsidian.md/t/guide-to-develop-and-execute-modular-javascript-in-obsidian/88339)

Whereas Markdown can only be used to produce static content, JavaScript **expands the possibilities of Obsidian** by enabling dynamic rendering. Its impact on performance is low and it reduces the need for additional plugins to add features. Furthermore, if the code is properly structured into JavaScript functions and externalized into external scripts, it **preserves the interoperability of markdown files** in case that you wish to migrate away from Obsidian. Indeed, it become unnecessary to go through the markdown files to modify and adapt them, as all you have to do is re-implements the few JavaScript functions located in the script files. It is these aspects that we are going to explore in this guide.

Here are a few examples of renderings created using JavaScript scripts with their call code from Markdown pages, which would be tedious to produce in pure Markdown:

- `Categories of the page` (inspired by Wikipedia, I use it as a footer on all my pages)

```javascript
await customJS.MacroCategoriesOfPage.listInBox(<dataview-plugin>, "<path-to-page>");
```

![example-1](https://forum.obsidian.md/uploads/default/original/3X/4/e/4e277e63fcb4d6c7ba55568d4337b67942b02b42.png)

- `Navigation bar` ([inspired by Wikipedia 1](https://fr.wikipedia.org/wiki/Projet:Portail_et_projet/Onglets), I use it as a page header on some pages)

```javascript
await customJS.MacroNavigationBar.forWelcomePage(<dataview-plugin>);
```

![example-2](https://forum.obsidian.md/uploads/default/original/3X/d/4/d4d9dfd20d7257cab638cc44f8dccff0665e7b34.png)

- `Dynamic link` (the link target a local file located outside Obsidian and its root varies depending on the OS, which is impossible to do in markdown)

```javascript
customJS.MacroSysinfoLink.onTextToFile("<link-text>", "partial-path-to-file", "<file-format-indicator>")
```

![example-3](https://forum.obsidian.md/uploads/default/original/3X/1/6/1619a058c65e0517a441f02cdea4e93a556e03f9.png)

- `Main article` ([inspired by Wikipédia](https://en.wikipedia.org/wiki/Template:Category_main_article))

```javascript
await customJS.MacroMainArticle.displayInBox(<plugin-dataview>, "<path-to-page>", "<link-text>");
```

![example-4](https://forum.obsidian.md/uploads/default/original/3X/b/1/b16b951d3fe616fc89eb236b5807187ef9113546.png)



