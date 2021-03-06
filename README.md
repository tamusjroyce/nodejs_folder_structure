# Small utility to build out your file structure

**July 15, 2019 - Recently refactored and greatly simplified.**

So when looking at package.json, you get a general idea of your project.
Including building out directories, blank files, downloading files, syncing npm modules to local directories (feature in progress implemented), and running javascript scripts (OS-agnostic) and commands.

> npm install --save-dev folder_layout

The only command that works right now is `build-fs`. Fortunately, it is the most useful. It builds your directory out and runs each script.

Just add it to your setup script within package.json

```
"scripts": [
  "setup": "npx build-fs",
  "run": "node app.js"
]
```

Beta example:

```
{
  "name": "folder_layout",
  "version": "0.1.0",
  "description": "Server-side module for folder structure building.",
  "main": "app.js",
  "scripts": {
    "setup":  "npx build-fs",
    "bundle": "./node_modules/.bin/browserify index.js > bundle.js",
    "test":   "./node_modules/.bin/mocha --reporter spec"
  },
  "keywords": [
    "folder",
    "folder-layout",
    "sync-directory"
  ],
  "author": "TamusJRoyce",
  "license": "SEE LICENSE IN license.txt",
  "devDependencies": {
    "chai": "^4.2.0",
    "mocha": "^5.2.0"
  },
  "folderLayout_note": "Your paths can be windows or posix based. They will be automatically detected converted to relative paths for processing.",
  "folderLayout": [
    "./README.md",
    "./server/app/.",
    "./server/scripts/setup/.",
    "./server/scripts/prerun/.",
    "./server/scripts/postrun/.",
    "./server/scripts/database/.",
    "./client/site/.",
    "./client/site/assets/.",
    "./client/site/scripts/.",
    "./client/site/styles/.",
    "./src/.",
    "./src/test/.",
    "./src/."
  ]
}
```

Notice above:  "folderLayout" shows a desired file structure.

> Note: Each folder path ends in a period. If a path ends in a slash, it actually represents all files within that path.
>       Per the Posix standard. **./server/app/ does not represent a directory!** ./server/app/. represents the directory.
>
>       https://en.wikipedia.org/wiki/Dirname#Misconceptions

Here is a more advanced one. This one initializes blank files and executes scripts to build out file content (only if the file did not exist before). This comes from package.json in this repository!

```
{
  "...": "...",
  "bin": {
    "folder-structure": "./bin/cli.js",
    "build-folderstruct": "./bin/cli-build_directory_structure.js",
    "init-folderstruct": "./bin/cli-init_directory_structure.js"
  },
  "folderLayout_note": "Your paths can be windows or posix based. They will be automatically detected converted to relative paths for processing.",
  "folderLayout": [
    "./test/.",
    "./test/tests.js",
    "./bin/.",
    [
      "./test/mocha.opts",
      "echo server-tests >> ./test/mocha.opts",
      "echo --recursive >> ./test/mocha.opts"
    ],
    [
      "./license.txt",
      "wget https://api.github.com/licenses/mit -O license.txt",
      "curl -o license.txt https://api.github.com/licenses/mit",
      "node --eval \"const https = require('https'); fs = require('fs'); https.get('https://api.github.com/licenses/mit', {json:true}, function(error,response,body) {if (error) {return console.log(error)$
    ],
    [
      "./.gitignore",
      "echo package.lock >> ./.gitignore",
      "echo node_modules/ >> ./.gitignore"
    ],
    {
      "chain": [
        "./README.md",
        "node --eval \"const package = require('./package.json'); console.log(package.description);\" >> ./README.md"
      ]
    }
  ],
  "repository": {
    "type": "git",
    "url": "https://github.com/tamusjroyce/nodejs_folder_structure.git"
  }
}
```

The reason for the arrays, such as "./.gitignore" above, is that if .gitignore already exists, the command below it will not be ran. Only when .gitignore is initially created will package.lock and node_modules/ be added to the .gitignore.

## Auto Discovery

So you might ask, "How does the script know to create a directory, a blank file, or execute a command?" It does it by finding a pattern in the text. Unless you specify it manually, per the Advanced section below (which you are more than welcome to do).

If the first word is an Action in the Advanced Commands below
* It will run the javascript function associated with the Action with the parameter specified

If it starts with a "./" and ends with a "/."
* Then it will guarentee the necessary folders so that the path exists (or fail while trying)

If it starts with a "./" and does end with a period, such as "./test/readme.txt"
* Then it will guarentee the necessary folders and create a blank file (unless the file already exists)

If it does not start with a period and does not match the rules above
* Then it will be executed at the command prompt

## Global Install

If you add the below to your package.json script section
```
"preinstall": "node -e \"const {execSync} = require('child_process'); JSON.parse(fs.readFileSync('package.json')).globalDependencies.foreach(globaldep => execSync('npm i -g ' + globaldep));\"",
```
and
```
  "globalDependencies": [
    "folder_layout",
    "potato"
  ],
```
then
> npm run preinstall

The above command will install folder_structure and a potato (because who doesn't want to install a potato? be sure to fork it) globally.

## RoadMap
* Add Unit Tests, now that the code has been refactored
* Setup a subfolder with a variety of folderLayout templates and examples similar to existing folder layout standards
* Link node_module subfolders into project path (thus removing the need for bower or requirement of webpack)
** Support folder sync fallback if filesystem linking is not possible - using Atom's cross-platform FileSystem Watcher.
* Support globalDependencies (de-clutter scripts...though inline scripts like above can be quite helpful)
* Allow more than one argument or specify arguments for a specified Action
* Find a way to inject the output of one action as the input of another Action

## Summery

It nice to document your desired folder layout. But it is better when your package manager handles it for you.

Hopefully Helpful!

-TamusJRoyce

## Advanced - tl;dr

### Supported Commands
Note: Replace ```[ ..., ... ]```
      with    ```{ "action": [ ..., ... ] }```
      for extra features. Action specified below.

Runs through all items, similar to the root node
* { "all": [ "...", "..." ] }

Runs through all items, but tries to do so in parallel, ignoring the result and continue processing
* { "parallel": [ "...", "..." ] }

Runs the same command across an array of items
* { "a command specified below": [ "...", "..." ] }

Runs a command for a specific action
* { "a command specified below": "..." }

Where Action can be:
* mkdir     - Creates a directory and subdirectories (auto-detected)
* touch     - Creates a file if not found, and any directories to get to the file (auto-detected)
* copy      - Copies a file or directory (not fully implemented yet)
* mv        - Moves a file (not fully implemented yet)
* rm        - Removes a file
* exec      - Runs a script (auto-detected)
* link      - Either symlinks a file or keeps a file in syncs to a different location (neither implemented yet)
* logerror  - Outputs to the console, useful for debugging
