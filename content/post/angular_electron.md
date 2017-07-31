+++
date = "2017-07-29T08:48:41+02:00"
draft = true
categories = [ "golang"
]
title = "Nats Proxy Framework"
thumbnail="/img/blog/electron/natsproxy_rest.png"
+++

# Angular + Electron = Desktop fun again
The desktop applications are some kind of old habit that I’m used to from back time. Today you are able to fullfill almost all your needs by the cloud services and online applications. But for me it is still more comfortable to have a dedicated application, than to have it in a tab of the browser.

If I look to the problem as a developer of web applications, and more concrete java developer. It is painful to create application desktop application with the GUI focused libraries of java. 

So if I can choose to create such app as a web application with use of basic html and javascript/typescript, I’m totally in.

Here comes the Electron platform (https://electron.atom.io/) based on chromium browser it provides presentation and runtime for your application.
The Electron platform accompanied by Angular framework is a very solid foundation for desktop application development.


# Bootstrap your project

Using an Angular CLI generate a new application and test the first run. After following command you should be able to see the result in the browser on URL http://localhost:4200.

    > ng new electron_app
    > ng serve

Add main file for the electron called main.js

${PROJECT_FOLDER}/main.js:
```
    const { app, BrowserWindow } = require('electron');
    
    // Executes when the application 
    // is initialized.
    app.on('ready', function() {
        console.log('Starting application!');
        // Create browser window 
        // with given parameters
        mainWindow = new BrowserWindow({ width: 1280, height: 960 });
        mainWindow.loadURL("http://localhost:4200");
    
        // It is useful to open dev tools
        // for debug.
        mainWindow.webContents.openDevTools();
        mainWindow.on('closed', function() {
            mainWindow = null;
        });
    });
    
    // Defines the behaviour on close.
    app.on('window-all-closed', function() {
        app.quit();
    });
```

And update the package.json file by few additional properties:
${PROJECT_FOLDER}/package.json:

```
    {
        "name": "electron_app",
        "version": "0.0.0",
        "main": "main.js",
        "license": "MIT",
        "scripts": {
            "ng": "ng",
            "start": "ng serve",
            "build": "ng build",
            "test": "ng test",
            "lint": "ng lint",
            "e2e": "ng e2e"
        },
        "private": true,
        "dependencies": {
            "@angular/animations": "^4.0.0",
            "@angular/common": "^4.0.0",
            "@angular/compiler": "^4.0.0",
            "@angular/core": "^4.0.0",
            "@angular/forms": "^4.0.0",
            "@angular/http": "^4.0.0",
            "@angular/platform-browser": "^4.0.0",
            "@angular/platform-browser-dynamic": "^4.0.0",
            "@angular/router": "^4.0.0",
            "core-js": "^2.4.1",
            "rxjs": "^5.4.1",
            "zone.js": "^0.8.14"
        },
        "devDependencies": {
            "@angular/cli": "1.2.4",
            "@angular/compiler-cli": "^4.0.0",
            "@angular/language-service": "^4.0.0",
            "@types/jasmine": "~2.5.53",
            "@types/jasminewd2": "~2.0.2",
            "@types/node": "~6.0.60",
            "codelyzer": "~3.0.1",
            "jasmine-core": "~2.6.2",
            "jasmine-spec-reporter": "~4.1.0",
            "karma": "~1.7.0",
            "karma-chrome-launcher": "~2.1.1",
            "karma-cli": "~1.0.1",
            "karma-coverage-istanbul-reporter": "^1.2.1",
            "karma-jasmine": "~1.1.0",
            "karma-jasmine-html-reporter": "^0.2.2",
            "protractor": "~5.1.2",
            "ts-node": "~3.0.4",
            "tslint": "~5.3.2",
            "typescript": "~2.3.3"
        }
    }
```

I recommend to install the electron platform globally by

```
    > npm install -g electron
```

The first simple manual run could be done by calling following commands (these must be run in separate terminal as the “ng serve” is blocking )

```
    > ng serve
    > electron .
```

Command “ng server” will start development server on http://localhost:4200, you can notice the this url is used in main.js file. 

# Serving static files

Serving angular application via development server is a development friendly solution. Better solution is to serve a static files from the distribution folder. The main.js file need to be changed to open the index.html file.

${PROJECT_FOLDER}/main.js
```
    const { app, BrowserWindow } = require('electron');
    path = require('path');
    url = require('url');
    app.on('ready', function() {
        console.log('Starting application!');
        mainWindow = new BrowserWindow({ width: 1280, height: 960 });
        mainWindow.loadURL(url.format({
            pathname: path.join(__dirname, 'dist', 'index.html'),
            protocol: 'file:',
            slashes: true
        }));
    
        // Opens dev tools
        mainWindow.webContents.openDevTools();
        mainWindow.on('closed', function() {
            mainWindow = null;
        });
    });
    app.on('window-all-closed', function() {
        app.quit();
    });
```
This might work, but after you open electron you will receive “Failed to load resource” error.

<img src ="https://d2mxuefqeaa7sj.cloudfront.net/s_F68920300BC018EB3E1E88614B689BAD376E939D730AA4BF30888F46A4DF2F8A_1501387336733_image.png"/>


So there is one small change that need to be done in your index.html file. We need to define the base for the file as absolute but from the actual working directory like “./”. This is done in <base> element in head tag of index.html . After the modification you can try “electron .” in your ${PROJECT_FOLDER}

${PROJECT_FOLDER}/index.html :

```
    <!doctype html>
    <html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Electron</title>
        <base href="./">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="icon" type="image/x-icon" href="favicon.ico">
    </head>
    <body>
        <app-root></app-root>
    </body>
    </html>
```