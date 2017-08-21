+++
tags = [ "angular","electron","desktop","electron-builder"]
categories = [ "angular"
]
date = "2017-08-20T20:33:51+02:00"
title = "Angular on Electron, part 2"
draft = false
thumbnail = "/img/blog/electron2/back.jpg"

+++

In the previous post the bootstrap of Angular project on Electron platform was described. In this one, the process of application packaging will be presented.

One of the benefits of Electron is that it runs on all major platforms. Each platform has naturally its needs regarding to creating the distribution package. Fortunately, the packaging and all that stuff do not need to be done manually. There are few tools which will help you. But first things first.

## Get ready

Before we begin this tutorial there will be one small change to an existing project from part 1. Because the folder "dist" will be used for another purpose, the Angular default build folder will be changed to "build". This is done by modification of "angular-cli.json" config file. The property "outDir" of "apps" section will change the default output directory.

${PROJECT_FOLDER}/angular-cli.json:

```
    "apps": [
        {
          ...
          "root": "src",
          "outDir": "build"
          ...}
          ],
      ...
```

After this change, test the configuration by building the project. The folder "build" should be created with all the content. If everything is OK, we are good to go.

```
    > ng build
    > tree build/
    build/
        ├── favicon.ico
        ├── index.html
        ├── inline.bundle.js
        ├── inline.bundle.js.map
        ├── main.bundle.js
        ├── main.bundle.js.map
        ├── polyfills.bundle.js
        ├── polyfills.bundle.js.map
        ├── styles.bundle.js
        ├── styles.bundle.js.map
        ├── vendor.bundle.js
        └── vendor.bundle.js.map
```

## Assets packaging

All the assets of application need to be bundled in the distribution somehow. Naturally there are several ways how to accomplish that. The first option is to bundle the files ( js, css bundles) as they are. Your install folder that will contain the build folder with all the files.  Sometimes it is better to hide the raw files from user. So the next option is to use archive. As this si preferred way, the asar archive were designed.


*Asar archive*

The asar archive (https://github.com/electron/asar) is tar-like format very easy to use. Following example demonstrates the basic usage.

```
    > npm install -g asar
    > asar pack app app.asar
```

The benefit of asar archive is that it could be read as simple as a folder in the filesystem. One way how to achieve it is via Node API. (https://electron.atom.io/docs/tutorial/application-packaging/#using-asar-archives)

```
    const fs = require('fs')
    fs.readdirSync('/path/to/example.asar')
```
There is more. The Electron BrowserWindow class provides the ability to access the archive too. So it can load the "index.html" file in the application the common way via the BrowserWindow#loadURL method. In this case, the "__dirname" would be the path to the .asar file.

```
        var path = require('path');
        var url = require('url');
        win = new BrowserWindow({ width: 1280, height: 960 });
        // __dirname is a current working directory
        win.loadURL(url.format({
                pathname: path.join(__dirname, 'build', 'index.html'),
                protocol: 'file:',
                slashes: true
        }));
```

Now after the packaging and the ways how to use the asar were described, let's move to an installer. Don't worry you would not need to create the archives and config files manually. The tools for this purpose are there for you.


## Electron builder

For this tutorial, the [electron-builder] (https://github.com/electron-userland/electron-builder) tool will be used.  The utility simplifies the creation of distribution package for all major platforms, even the cross platform build is possible.

```
    npm install electron-builder --save-dev
```

The electron-builder tool needs a little bit of configuration, so the final changes in "package.json" file looks like this (This is a minimal configuration)
${PROJECT_FOLDER}/package.json

```
      "author": {
        "name": "Radomir Sohlich",
        "email": "sohlich@example.com"
      },
      "description": "Example application.",
      "devDependencies": {
         "electron-builder": "^19.22.0",
      },
      "build": {
        "appId": "com.example.app",
        "files": [
          "main.js",
          "build"
        ],
        "mac": {
          "category": "productivity"
        }
      }
```

The build->files section defines which files/folders will be included in asar archive. The archive works as the application folder. The electron executes the main.js directly from the archive. The build folder contains application build with index.html and bundles. The "author" and "description" properties are not mandatory, but it is always better to provide this info. (The electron-builder will produce less warning messages :) )

The final step is to add a npm command to run the electron-builder build.

${PROJECT_FOLDER}/package.json

```
    "scripts": {
        "dist": "build --mac"
      },
```

The "dist" command will build the application and subsequently build a distribution package. In this case, the MacOS is configured. The platform is defined by the switch. The available values are in the following figure. (stolen from project documentation https://github.com/electron-userland/electron-builder#cli-usage)

```
      --mac, -m, -o, --macos   Build for macOS, accepts target list (see
                               https://goo.gl/HAnnq8).                       [array]
      --linux, -l              Build for Linux, accepts target list (see
                               https://goo.gl/O80IL2)                        [array]
      --win, -w, --windows     Build for Windows, accepts target list (see
                               https://goo.gl/dL4i8i)                        [array]
```

To execute the build, call the command in terminal and the output should look like this. 

```
    > npm run dist
    > electron_app@0.0.0 distmac /Users/radek/Junk/electron_app
    > ng build && build --mac
    
    Hash: ee6c31ccdab8735b740c
    Time: 10199ms
    chunk    {0} polyfills.bundle.js, polyfills.bundle.js.map (polyfills) 177 kB {4} [initial] [rendered]
    chunk    {1} main.bundle.js, main.bundle.js.map (main) 5.27 kB {3} [initial] [rendered]
    chunk    {2} styles.bundle.js, styles.bundle.js.map (styles) 10.5 kB {4} [initial] [rendered]
    chunk    {3} vendor.bundle.js, vendor.bundle.js.map (vendor) 1.89 MB [initial] [rendered]
    chunk    {4} inline.bundle.js, inline.bundle.js.map (inline) 0 bytes [entry] [rendered]
    electron-builder 19.22.1
    No native production dependencies
    Packaging for darwin x64 using electron 1.7.5 to dist/mac
    Building macOS zip
    Building DMG
    Application icon is not set, default Electron icon will be used
```

After the successful command run, the dist folder should be created in the working directory and the installer should be there.

```
    dist/
    --- electron_app-0.0.0-mac.zip
    --- electron_app-0.0.0.dmg
    --- mac
        --- electron_app.app
```

The "electron_app.app/Resources" folder contains the asar archive with "app.asar" file. That is our bundled application. If you need to debug, what is packed into "app.asar" file and what is included in the installer. The "build --dir" command could be used. It will create only the expanded distribution folder (same as dist/mac). 

Note:
If you encounter the error by executing the "build --dir" command.

```
    Error: Cannot find electron dependency to get electron version
```

The electron dependency need to be add to project.

```
    npm install electron --save-dev
```

The repository with example project [](https://github.com/sohlich/angular_on_electron)



### Series:
- [Angular on Electron, part 1](/post/angular_electron/)
- [Angular on Electron, part 2](/post/angular_electron_2/)
