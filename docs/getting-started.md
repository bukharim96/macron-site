# Getting Started

Macron is a light framework for building cross-platform desktop applications with JavaScript, HTML and CSS using the system's native web engine. It does so by creating a single thread where all static content runs alongside native modules tailored for rapid development. To learn more about Macron, check the <a href="/">homepage</a> or the <a href="/">Under The Hood</a>.

As far as you need to get up and running, you will only need a single configuration file to make things happen. This section will get you covered with all the basic commands to start, serve and build a full Macron application in a few simple steps.

## Prerequisites

To get up and running with Macron, your machine will need to meet some requirements. Macron uses different approaches for creating applications on different platforms. For this reason, you need to install the independent software dependencies for your particular machine.

||Windows|MacOS|Linux
|--|--|--|--|
|NodeJS >=8.11.3|Yes|N/A|N/A|
|Python >=3.31|Yes|N/A|N/A|
|pip >=9.0.1|Yes|N/A|N/A|
|winshell|Yes|N/A|N/A|
|WPF|Yes|N/A|N/A|
|pythonnet|Yes|N/A|N/A|
|pyinstaller >=3.4|Yes|N/A|N/A|
|pygubu|Yes|N/A|N/A|

!!! warning
    We currently support Windows only. A MacOS and Linux version is expected to come soon.
    Previous versions of Node are not battle-tested so the table above is susceptible to changes.

## Installation

No extra requirements are needed for this process since all the requirements are expected to be covered. With NodeJS installed, run:

```
npm install -g macron
```

The following commands show how to get started quickly.

* `macron init [app-name]` - Create a new Macron project. Run this command without <code>app-name</code> to initialize Macron in the current directory.
* `macron start` - Starts the application. Creates windows in a new thread.
* `macron build` - Builds a stable application. <a href="">Learn more</a>.
* `macron deploy` - Creates final build of your app and setup wizard. <a href="">Learn more</a>.
* `macron --help` - Run this command to revisit commands and receive further help with your application.

## Project layout

When you run the `macron init` command, you will get a few files. The most important of these is the `macron.config.js` configuration file which we will cover in a moment. Traditionally, the general project layout is defined in the following order even though it is not opinionated at all:

    public/
        icons/
            icon.ico                  # Windows only (for now)
        index.html                    # The entry point to the application
    src/                              # All application logic and backend
    macron.config.js                  # The configuration file

## Macron configuration endpoint

The `macron.config.js` is a single entry point for handling general application settings. It contains several options for managing how the main window start ups (maximized, fixed width, etc.) and how the build is created.

After a successful initialization, the configurations are passed on to the native end where the main window of your app is created. Check the <a href='under-the-hood'>Under The Hood</a> section for a detailed explanation of the process.

``` javascript
const { Window } = require('macron');

const App = new Window({
  title: 'MyFirstMacronWindow',
  width: 1200,
  height: 960,
  minHeight: 500,
  minWidth: 500,
  maxHeight: 500,
  maxWidth: 500,
  frameless: true,
  startupFromCenter: true,
  // startupState: 'maximized',
  devServerURI: 'http://localhost:3000/',
  menu: require('./src/menubar')
});

module.exports = {
  name: 'MyCoolApp',
  mainWindow: App,
  build: {
    debugMode: false
  }
}
```

## Configuration options

The <code>Window</code> object entails and generates a ready-made API endpoint that is used by the native end to create a platform-specific window. This is the root of your application and can be built at any given moment into a stable application with a setup wizard that could be distributed.

The following options are available for any configuration and only apply to the main window:

* **title** <code>string</code> - Required. The statusbar title.
* **width** <code>number</code> - The window width.
* **height** <code>number</code> - The window height.
* **minHeight** <code>number</code> - The minimum window height.
* **minWidth** <code>number</code> - The minimum window width.
* **maxHeight** <code>number</code> - The maximum window height.
* **maxWidth** <code>number</code> - The maximum window width.
* **frameless** <code>boolean</code> - Removes outer frame from window if `true`. Default is `false`.
* **startupFromCenter** <code>boolean</code> - Determines whether window should appear in the center if `true`. If set to `false`, the platform decides where to draw the window. Default is `true`.
* **startupState** <code>string</code> - Default `'normal'`. Could be either:
    * `'maximized'` - Window starts in a maximized state.
    * `'minimized'` - Window starts in a minimized state.
    * `'normal'` - Window starts in a state described by the platform.
* **devServerURI** <code>string</code> - Specifies a local or online URL source to start the app. Overrides `sourcePath` if defined.
* **menu** <code>MacronMenubar</code> - A pointer to the app's menubar.

!!! tip
    1. All the options are inserted into the global window context as `macron.CurrentWindow` and can all be accessed as dynamic setters and getters which means a script could mutate any of these properties.
    2. Setting a single <code>minWidth</code> and <code>maxWidth</code> alongside a single <code>minHeight</code> and <code>maxHeight</code> will result in a window that cannot be resized.
    3. Building your app adds some extra options to the `macron.config.js` for the build process and is covered in the <a href='/build'>Build & Deploy</a> section.

## Taking your app to the next level with native modules

Macron uses the Python runtime to access native functionalities of the platform. These functionalities are then curated into a common form that can be recognized by Macron's `NativeBridge` class. The modules then get bundled with an app and can get accessed at runtime.

### Accessing core modules

Since desktop apps are expected to have a lot of behaviour, Macron makes use of Python's core modules to generate a set of APIs that can be accessed at runtime. Some of those APIs include `FileSystem`, `Process` and more. Check out the <a href='/api'>API Reference</a> section for more on core modules.

#### Example

The example below demonstrates the process of using the `FileSystem` API to set and get the contents of a '*pupp.txt*' file.

```javascript
const { FileSystem } = require('macron');
const originalContents = 'I 💕 🐶';

FileSystem.writeFile('./pupps.txt', originalContents);

alert(FileSystem.readFile('./pupps.txt') === originalContents) // pass

```

<!-- ### Creating custom native modules

HTML5 web apps using JavaScript often find performance to be a matter of concern. As a matter of fact, a Macron app with native modules is way more performant than one without. Macron caters for performance-critical apps by giving the developer all the power to target a user's hardware capabilities.

Advantages include:

* Accessing the WebCam,
* Accessing the shell/kernel,
* Changing the desktop's layout and style,
* Creating complex hashes and cryptos,
* Creating and managing TCP servers,
* Viewing torrents,
* And a myriad more...

#### Example

This example shows how to get the working directory of your app:

```python tab='Native (Python)'
from os import getcwd
from macron import NativeBridge, macronMethod

class OperatingSystem(NativeBridge):
    @macronMethod
    def getWorkingDirectory(self):
        return getcwd() or None

```

```javascript tab="Web Context (JavaScript)"
const { OperatingSystem } = require('macron');

console.log(
    OperatingSystem.getWorkingDirectory() // logs working directory to console
);

```

!!! tip
    1. For extra performance measure, low-level languages could be used to create assemblies that could be accessed. For a C/C++ combination with you app, all you need is to run `make` on your module and use
    Python to '*macronize*' it as illustrated in the demo above. Go to the <a href='/how-tos'>How Tos</a> section to get information on support for other languages.
    2. Native modules can be established as standalone plugins.
    3. Find out more on native modules in the <a href='/api'>API Reference</a> section. -->

## Building your app for deployment

If you went along with this mini-tutorial from the start, you are on the right track. The next step is to build your awesome app into something quite interactable that can be used by your users.

To build your app, run:

```
macron build
```

... in your project directory. Once your app is built, run:

```
macron start build
```

... to execute your build.

For more on building and deploying your app, check the <a href="">Building & Deploying</a> section.
