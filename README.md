# logbookHelper
<img src="icon.png" width="200px" alt="logbookHelper icon with letter L and H" /> 

**Disclaimer**: DEVELOPMENT IN PROGRESS.

### Internship logbook helper for CPNV students
This is an **opensource web extension** licensed under **[GNU GPLv3 or later](/LICENSE)**. It's dedicated IT CPNV students for their 2 interships on 3rd and 4th year. As they have to fill hours and descriptions about their work everyday, this extension could help to use the app on the Intranet. The extension take load inforation directly from `intranet.cpnv.ch` under the `/stages/Journal.php` route. The data and configuration keep in local.  
It use VueJS, TailwindCSS and Jetbrains Mono. <!-- **Only available for Firefox Desktop 57.0+**.-->

## Features
- Shows how many hours or days late or early there are.
- ...

## Build
If you want to build the extension from the source, you need:
- Windows 10, MacOS, Linux
- NPM v6^

### Process
- Clone the repos, open a shell
- `cd logbookHelper`
- `cd extension`
- `npm install`
- `npm run build`

The build is now available in the `dist` or as `.zip` file under `artifacts` (name like `logbookHelper-v1.0.1-production.zip`). It's not signed by Mozilla so you can only install it temporarly.

## Install temporarly unsigned `.zip` or .`xpi`
1. Go to `about:debugging`
1. Under This Firefox, click `Load a temporary module` and select the file
1. The module is available until you close Firefox.

## Install signed versions
### Install with a `.xpi` or a `.zip` downloaded
1. Go to `about:addons`
1. Click on the settings icon
1. Install Add-on from file
1. Select the extension file
1. Click `Add`

--> The extension is correctly installed if you can open it as popup at the top right of your browser (you should the see the extension icon and click on it).

### Install from `.xpi` under a download link
1. Click on the download link
1. Follow step 5 in the precedent procedure

## Develop
If you want to develop the extension:
- `cd extension`
- `npm install`
- `npm run serve`
- Follow the procedure `Install temporarly unsigned .zip or .xpi` and select the file `extension/dist/manifest.json`. The `dist` folder contains the extension code transpiled by webpack.
- Go to `about:debugging` and click `Inspect` for the extension.
- Now you can start to develop in your IDE (`npm run serve` will build the code at each save) and to see changes just type F5 in the debugging tab. The extension will be reloaded. Look at [this article](https://extensionworkshop.com/documentation/develop/debugging/#debugging-popups).
- When you want to build the extension, change `manifest.json` and `package.json` versions and follow the build process (`npm run build`).

## Versions
WIP

## Credits
- openDevApps - [GNU GPLv3+](https://github.com/samuelroland/openDevApps/blob/develop/LICENSE.txt)  
Copyright (C) 2021 Samuel Roland  
(This is a derivative work of openDevApps, some files have been copied to have a basic file structure. Because of the openDevApps licence, logbookHelper is under the same licence).

### Librairies
- TailwindCSS - [MIT](https://github.com/tailwindlabs/tailwindcss/blob/master/LICENSE)  
Copyright (c) Adam Wathan <adam.wathan@gmail.com>  
Copyright (c) Jonathan Reinink <jonathan@reinink.ca>  
- VueJS - [MIT](https://github.com/vuejs/vue/blob/dev/LICENSE)  
Copyright (c) 2013-present, Yuxi (Evan) You
### Fonts
- Jetbrains Mono - [OFL](https://github.com/JetBrains/JetBrainsMono/blob/master/OFL.txt)  
Copyright 2020 The JetBrains Mono Project Authors (https://github.com/JetBrains/JetBrainsMono)
### Icons
Found with [iconduck.com](https://iconduck.com).
- Fluent UI System Icons  [MIT](https://github.com/microsoft/fluentui-system-icons/blob/master/LICENSE)  
Copyright (c) 2020 Microsoft Corporation
- Material Design Icons - [Apache License 2.0](https://github.com/Templarian/MaterialDesign/blob/master/LICENSE)
- Majesticons Icons Set - [MIT](https://github.com/halfmage/majesticons/blob/main/LICENSE)  
Copyright (c) 2021 Gerrit Halfmann
