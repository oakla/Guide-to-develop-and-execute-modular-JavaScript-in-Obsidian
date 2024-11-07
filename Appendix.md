[quote="Milleras, post:2, topic:88339"]
## Appendix

### Script `utility-obsidian.js` with the module `UtilityObsidian`

Before using this code, make sure you update the module search path in the `importModule()` function.

utility-obsidian.js

```
// @ts-check

/* -------------------------------------------------------------------------- */
/*                                Import Section                              */
/* -------------------------------------------------------------------------- */

/**
 * @typedef {import('custom-js')} CustomJS
 * @typedef {import('obsidian')} Obsidian
 * @typedef {import('obsidian').App} App
 * @typedef {import('obsidian').Notice} Notice
 * @typedef {import('obsidian').TAbstractFile} TAbstractFile
 * @typedef {import('obsidian').TFile} TFile
 * @typedef {import('obsidian').TFolder} TFolder
 */

class ModuleLoader {
  /**
   * @param {string} moduleFolderSuffix Suffixe name of one of folder module in `/03-Fichier/scripts_module_<suffixe>/` folders.
   * @param {string} moduleFile Name of the file module in the `/03-Fichier/scripts_module_<suffixe>/` folder.
   * @returns {any} Exported module.
   */
  static importModule(moduleFolderSuffix, moduleFile) {
    const { obsidian, app } = self.customJS || {};
    if (obsidian == null || app == null) throw new Error("customJS is null.");
    let adapter = app.vault.adapter;
    if (adapter instanceof obsidian.FileSystemAdapter) {
      const modulePath =
        adapter.getBasePath() +
        "/03-File/scripts_module_" +
        moduleFolderSuffix +
        "/" +
        moduleFile;
      delete global.require.cache[global.require.resolve(modulePath)];
      return require(modulePath);
    }
    throw new Error("Obsidian adapter is not a FileSystemAdapter.");
  }
}
/* -------------------------------------------------------------------------- */

class UtilityObsidian {
  // @ts-ignore
  static #notice = (msg) => new Notice(msg, 5000);
  static #log = (msg) => console.log(msg);
  static #noticeAndThrowError = (error) => {
    // @ts-ignore
    new Notice(error, 5000);
    throw new Error(error);
  };
  static #logAndThrowError = (error) => {
    console.log(error);
    throw new Error(error);
  };

  /**
   * Get all tags in the vault, with their number of occurrences.
   * @param {App} app A reference to the Obsidian `app`.
   * @returns {Map<string,number>} Map in the form `{ key: <tag-name>, value: <nb-occurrence> }`
   */
  static getAllTags(app) {
    // @ts-ignore
    return new Map(Object.entries(app.metadataCache.getTags()));
  }

  /**
   * Get the name of all tags in the vault.
   * @param {App} app A reference to the Obsidian `app`.
   * @returns {string[]} List of all tags in the vault.
   */
  static getAllTagNames(app) {
    // @ts-ignore
    return Object.keys(app.metadataCache.getTags());
  }

  /**
   * Create a new plaintext file inside the vault. Parent folder is also created if it does not exists.
   * @alias UtilityObsidian:createFileWithInput
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @param {string} filePath Vault absolute path for the new file, with extension.
   * @param {string} fileContent Text content for the new file.
   * @returns {Promise<TFile>}
   */
  static async createFileWithInput(app, obsidian, filePath, fileContent) {
    const dirMatch = filePath.match(/(.*)[/\\]/);
    /** @type {string} */
    let dirName = "";
    if (dirMatch) dirName = dirMatch[1];

    const dir = app.vault.getAbstractFileByPath(dirName);

    if (!dir || !(dir instanceof obsidian.TFolder)) {
      await UtilityObsidian.createFolder(app, dirName);
    }

    return await app.vault.create(filePath, fileContent);
  }

  /**
   * Create a new folder inside the vault.
   * @alias UtilityObsidian:createFolder
   * @param {App} app Obsidian app.
   * @param {string} folderPath Vault absolute path for the new folder.
   * @returns {Promise<TFolder|null>} The created folder or `null` if it already exists.
   */
  static async createFolder(app, folderPath) {
    const folderExists = await app.vault.adapter.exists(folderPath);

    /** @type {(TFolder|null)} */
    let newFolder = null;
    if (!folderExists) {
      newFolder = await app.vault.createFolder(folderPath);
    } else {
      UtilityObsidian.#log(`Folder ${folderPath} already exists.`);
    }

    return newFolder;
  }

  /**
   * Get paths of all folders in the vault.
   * @alias UtilityObsidian:getAllFolderPathsInVault
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @returns {string[]} List of paths of all folders in the vault.
   */
  static getAllFolderPathsInVault(app, obsidian) {
    return app.vault
      .getAllLoadedFiles()
      .filter((f) => f instanceof obsidian.TFolder)
      .map((folder) => folder.path);
  }

  /**
   * Get all folders in the vault.
   * @alias UtilityObsidian:getAllFoldersInVault
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @returns {TFolder[]} List of all folders in the vault.
   */
  static getAllFoldersInVault(app, obsidian) {
    return app.vault
      .getAllLoadedFiles()
      .filter((f) => f instanceof obsidian.TFolder);
  }

  /**
   * Get all markdown files in the vault.
   * @alias UtilityObsidian:getAllMarkdownFiles
   * @param {App} app A reference to the Obsidian `app`.
   * @returns {TFile[]} List of all markdown files in the vault.
   */
  static getAllMarkdownFilesInVault(app) {
    return app.vault.getMarkdownFiles();
  }

  /**
   * Get all files whose path begins with `folderPath`.
   * @param {App} app A reference to the Obsidian `app`.
   * @param {string} folderPath Path folder to explore.
   * @returns {TFile[]} List of all files found.
   */
  static getAllFilesInFolder(app, folderPath) {
    return app.vault
      .getFiles()
      .filter((file) => file.path.startsWith(`${folderPath}`));
  }

  /**
   * Get all markdown files whose path begins with `folderPath`.
   * @param {App} app A reference to the Obsidian `app`.
   * @param {string} folderPath Path folder to explore.
   * @returns {TFile[]} List of all markdown files found.
   */
  static getAllMarkdownFilesInFolder(app, folderPath) {
    return app.vault
      .getMarkdownFiles()
      .filter((file) => file.path.startsWith(`${folderPath}`));
  }

  /**
   * Check if something exists at the given path.
   * @alias UtilityObsidian:fileExists
   * @param {App} app Obsidian app.
   * @param {string} filePath Path to file.
   * @returns {Promise<boolean>}
   */
  static async fileExists(app, filePath) {
    return await app.vault.adapter.exists(filePath);
  }

  /**
   * Tries to move to system trash. If that isn't successful/allowed, use local trash.
   * @alias UtilityObsidian:removeFile
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @param {string} filePath Vault absolute path to the file, with extension, case sensitive.
   */
  static async removeFile(app, obsidian, filePath) {
    const file = app.vault.getAbstractFileByPath(filePath);
    if (file && file instanceof obsidian.TFile)
      await app.vault.trash(file, true);
  }

  /**
   * Get the list of tags inside the frontmatter property `file`.
   * @alias UtilityObsidian:getFrontMatterTags
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @param {TFile} file Vault file to parse.
   * @returns {string[]} Tags inside the frontmatter of `file`.
   */
  static getFrontMatterTags(app, obsidian, file) {
    const fileFrontMatter = app.metadataCache.getFileCache(file)?.frontmatter;
    if (!fileFrontMatter) return [];

    const fileTags = obsidian.parseFrontMatterTags(fileFrontMatter);
    if (!fileTags) return [];

    return fileTags;
  }

  /**
   * Get the list of alias inside the frontmatter property `alias`.
   * @alias UtilityObsidian:getFrontMatterAliases
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @param {TFile} file Vault file to parse.
   * @returns {string[]} Aliases inside the frontmatter of `file`.
   */
  static getFrontMatterAliases(app, obsidian, file) {
    const fileFrontMatter = app.metadataCache.getFileCache(file)?.frontmatter;
    if (!fileFrontMatter) return [];

    const fileAliases = obsidian.parseFrontMatterAliases(fileFrontMatter);
    if (!fileAliases) return [];

    return fileAliases;
  }

  /**
   * Get inside the frontmatter the value of the property `propertyName`.
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @param {TFile} file Vault file to parse.
   * @param {string} propertyName Property name of the value to get.
   * @returns {any} Value inside the frontmatter of `file`.
   * @throws {Error} If the property doesn't exists.
   */
  static getFrontMatterEntry(app, obsidian, file, propertyName) {
    const fileFrontMatter = app.metadataCache.getFileCache(file)?.frontmatter;
    if (!fileFrontMatter) return [];

    const propertyValue = obsidian.parseFrontMatterEntry(
      fileFrontMatter,
      propertyName
    );
    if (propertyValue == null)
      throw new Error(
        `The property ${propertyName} doesn't exists in the file ${file.path}.`
      );

    return propertyValue;
  }

  /**
   * Get a file inside the vault at the given path.
   * @alias UtilityObsidian:getFileByPath
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @param {string} filePath Vault absolute path to the file, with extension, case sensitive.
   * @returns {TFile}
   * @throws {Error} If file not found, file is a folder, or file is not a file.
   */
  static getFileByPath(app, obsidian, filePath) {
    const file = app.vault.getAbstractFileByPath(filePath);

    if (!file) {
      UtilityObsidian.#log(`'${filePath}' not found.`);
      throw new Error(`'${filePath}' not found.`);
    }

    if (file instanceof obsidian.TFolder) {
      UtilityObsidian.#log(`'${filePath}' found but it's a folder.`);
      throw new Error(`'${filePath}' found but it's a folder.`);
    }

    if (!(file instanceof obsidian.TFile)) {
      UtilityObsidian.#log(`${filePath} is not a file.`);
      throw new Error(`${filePath} is not a file.`);
    }

    return file;
  }

  /**
   * Get a folder inside the vault at the given path.
   * @alias UtilityObsidian:getFolderByPath
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @param {string} folderPath Vault absolute path to the folder, with extension, case sensitive.
   * @returns {TFolder}
   * @throws {Error} If folder not found, folder is a file, or folder is not a folder.
   */
  static getFolderByPath(app, obsidian, folderPath) {
    const folder = app.vault.getFolderByPath(folderPath);

    if (!folder) {
      UtilityObsidian.#log(`'${folderPath}' not found`);
      throw new Error(`'${folderPath}' not found.`);
    }

    if (folder instanceof obsidian.TFile) {
      UtilityObsidian.#log(`'${folderPath}' found but it's a file.`);
      throw new Error(`'${folderPath}' found but it's a file.`);
    }

    if (!(folder instanceof obsidian.TFolder)) {
      UtilityObsidian.#log(`'${folderPath}' is not a folder.`);
      throw new Error(`'${folderPath}' is not a folder.`);
    }

    return folder;
  }

  /**
   * Trigger an update of a file in the cache.
   * @alias UtilityObsidian:cacheUpdate
   * @param {App} app Obsidian app.
   * @param {TFile} targetFile Vault file to update.
   * @returns {Promise<void>}
   */
  static cacheUpdate(app, targetFile) {
    return new Promise((resolve, reject) => {
      const timeout = setTimeout(() => reject("Cache update timeout"), 500);
      const resolvePromise = (file) => {
        if (file === targetFile) {
          clearTimeout(timeout);
          app.metadataCache.off("changed", resolvePromise);
          resolve();
        }
      };
      app.metadataCache.on("changed", resolvePromise);
    });
  }

  /**
   * Move a file safely, and update all links to it depending on the user's preferences.
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @param {TFile} fileToMove The file to move.
   * @param {TFolder} newParent The folder to which the file will be moved.
   * @throws {Error} If the new parent doesn't exists.
   */
  static async moveFile(app, obsidian, fileToMove, newParent) {
    const newParentPath = newParent.path;
    const folderExists = await app.vault.adapter.exists(newParentPath);
    if (!folderExists) {
      UtilityObsidian.#log(`The folder '${newParentPath}' doesn't exists.`);
      throw new Error(`The folder '${newParentPath}' doesn't exists.`);
    }
    const newFilePath = obsidian.normalizePath(
      `${newParentPath}/${fileToMove.name}`
    );
    await app.fileManager.renameFile(fileToMove, newFilePath);
  }

  /**
   * Renames the file (keeps the same file extension).
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @param {TFile} fileToRename The file to rename.
   * @param {string} newFileName The new file name without the file extension, which will be deducted from `fileToRename`.
   * @throws {Error} If the new file name contains `\`, `/` or `:`.
   */
  static async renameFile(app, obsidian, fileToRename, newFileName) {
    if (newFileName.match(/[\\/:]+/g)) {
      UtilityObsidian.#log(
        `File name ${newFileName} cannot contain any of these characters: \\ / :`
      );
      throw new Error(
        `File name ${newFileName} cannot contain any of these characters: \\ / :`
      );
    }

    const newFilePath = obsidian.normalizePath(
      `${fileToRename.parent?.path}/${newFileName}.${fileToRename.extension}`
    );
    await app.fileManager.renameFile(fileToRename, newFilePath);
  }

  /**
   * Insert a `text` at the cursor position.
   * @param {App} app A reference to the Obsidian `app`.
   * @param {string} text Text to insert at the cursor position.
   * @throws {Error} If no active editor.
   */
  static insertAtCursorPosition(app, text) {
    const activeEditor = app.workspace.activeEditor;
    if (!activeEditor || !activeEditor.file || !activeEditor.editor)
      UtilityObsidian.#noticeAndThrowError("No active editor.");

    // @ts-ignore
    const editor = activeEditor.editor;
    // @ts-ignore
    editor.replaceRange(text, editor.getCursor());
  }

  /**
   * Replace the selectioned text by `replacement` text.
   * @param {App} app A reference to the Obsidian `app`.
   * @param {string} replacement Text in replacement of selection.
   * @throws {Error} If no active editor.
   */
  static replaceSelection(app, replacement) {
    const activeEditor = app.workspace.activeEditor;
    if (!activeEditor || !activeEditor.file || !activeEditor.editor)
      UtilityObsidian.#noticeAndThrowError("No active editor.");

    // @ts-ignore
    const editor = activeEditor.editor;
    // @ts-ignore
    const doc = editor.getDoc();
    doc.replaceSelection(replacement);
  }

  /**
   * Get the select text in the current page.
   * @param {App} app A reference to the Obsidian `app`.
   * @param {Obsidian} obsidian A reference to the Obsidian API.
   * @returns {string} The selected texte or empty.
   * @throws {Error} If no active view.
   */
  static getSelectedText(app, obsidian) {
    const activeView = app.workspace.getActiveViewOfType(obsidian.MarkdownView);
    if (!activeView)
      UtilityObsidian.#noticeAndThrowError(
        "No active view - could not get selected text."
      );

    return activeView?.editor.getSelection() ?? "";
  }
}

module.exports = UtilityObsidian;
```

### Script `macro-tags-of-pages.js` with the macro `MacroTagsOfPage`

Before using this code, make sure you update the module search path in the `importModule()` function.

macro-tags-of-pages.js

```
// @ts-check

class MacroTagsOfPage {
  /* -------------------------------------------------------------------------- */
  /*                                Import Section                              */
  /* -------------------------------------------------------------------------- */

  /**
   * @typedef {import('custom-js')} CustomJS
   * @typedef {import('obsidian')} Obsidian
   * @typedef {import('obsidian').App} App
   * @typedef {import('obsidian').Notice} Notice
   * @typedef {import('obsidian').TAbstractFile} TAbstractFile
   * @typedef {import('obsidian').TFile} TFile
   * @typedef {import('obsidian').TFolder} TFolder
   */

  /**
   * @param {string} moduleFolderSuffix Suffixe name of one of folder module in `/03-Fichier/scripts_module_<suffixe>/` folders.
   * @param {string} moduleFile Name of the file module in the `/03-Fichier/scripts_module_<suffixe>/` folder.
   * @returns {any} Exported module.
   */
  static importModule(moduleFolderSuffix, moduleFile) {
    const { obsidian, app } = self.customJS || {};
    if (obsidian == null || app == null) throw new Error("customJS is null.");
    let adapter = app.vault.adapter;
    if (adapter instanceof obsidian.FileSystemAdapter) {
      const modulePath =
        adapter.getBasePath() +
        "/03-File/scripts_module_" +
        moduleFolderSuffix +
        "/" +
        moduleFile;
      delete global.require.cache[global.require.resolve(modulePath)];
      return require(modulePath);
    }
    throw new Error("Obsidian adapter is not a FileSystemAdapter.");
  }

  /** @type {typeof import('module-utility-obsidian')} */
  Obsidian = MacroTagsOfPage.importModule("common", "utility-obsidian.js");
  /** @type {typeof import('module-config')} */
  Config = MacroTagsOfPage.importModule("common", "config.js");
  /* -------------------------------------------------------------------------- */

  /**
   * Print a box in HTML with the list of tags of a page `pagePath`.
   * @param {Object} dv @param {Object} dv DataView object of Obisidian extension.
   * @param {string} pagePath Vault absolute path to the page with tags, with extension.
   * @returns {Promise<void>} A box in HTML with a list of tags.
   */
  async listInBox(dv, pagePath) {
    // Add CSS class to global div.
    dv.container.className += " macro-tags-of-page";

    // Render the list.
    const div = dv.el("div", "Tags: ", {
      container: dv.container,
      cls: "tags",
    });
    await this.#renderListOfTags(dv, div, pagePath);
  }

  /**
   * Print the list of the tags of a page `pagePath` in using its metadata `tags`.
   * @param {Object} dv DataView object of Obisidian extension.
   * @param {Object} container Parent HTML container.
   * @param {string} pathToPage Vault absolute path to the page with tags (or not), with extension.
   */
  async #renderListOfTags(dv, container, pathToPage) {
    const { obsidian, app } = self.customJS || {};
    if (obsidian == null || app == null) throw new Error("customJS is null.");

    // Get data
    const page = this.Obsidian.getFileByPath(app, obsidian, pathToPage);
    const pageTagNames = this.Obsidian.getFrontMatterTags(
      app,
      obsidian,
      page
    ).sort();

    // Render the list.
    const ul = dv.el("ul", "", {
      container: container,
    });
    ul.innerText = "";
    for (const pageTagName of pageTagNames) {
      dv.el("li", pageTagName, {
        container: ul,
      });
    }
  }
}
```

### Style CSS `macro.css` containing the class `.macro-tags-of-page`

macro.css

```
div.macro-tags-of-page {
  --line-height-medium: 1.5714285;
  --background-color-neutral-subtle: #f8f9fa;
  --color-base: #202122;
  --border-color-base: #a2a9b1;
  --color-progressive: #36c;
  --color-progressive--hover: #447ff5;
  --color-visited: #6b4ba1;

  /* Remove theme style */
  box-sizing: unset;

  /* New style */
  line-height: var(--line-height-medium);
  background: var(--background-color-neutral-subtle);
  font-family: sans-serif;
  color: var(--color-base);
  caret-color: var(--color-base);
  border: 1px solid var(--border-color-base);
  margin-top: 1em;
  padding: 5px;
  clear: both;
```
[/quote]

