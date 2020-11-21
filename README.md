# Screensy
#### This is my first Desktop app build with ElectronJS. This application  includes Per Window Screen Recording.


## Quick Start Guide 

**Electron** is a framework that enables you to create desktop applications with JavaScript, HTML, and CSS. These applications can then be packaged to run directly on macOS, Windows, or Linux, or distributed via the Mac App Store or the Microsoft Store.

### Prerequisites

Before proceeding with Electron you need to install [Node.js](https://nodejs.org/en/download/). 

To check that Node.js was installed correctly, type the following commands in your terminal client:

```
node -v
npm -v
```
The commands should print the versions of Node.js and npm accordingly. If both commands succeeded, you are ready to install Electron.

### Electron Forge

Create a new app with [Electron Forge](https://www.electronforge.io/) - it provides a solid starting point for building and distributing the app.

Open [VS Code Editor](https://code.visualstudio.com/) and create a project folder. Then type the following in the **Terminal**.

```text

npx create-electron-app my-app

cd my-app
npm start

```
> **ProTip:** Enter `rs` into the terminal to restart the app after making code changes.

####  HTML Markup
The HTML contains a `<video>` element to preview the output from a screen and provides buttons to start/stop recording.

#### Include Node in Electron’s Render Process

In order to use Node in Electron’s frontend render process, we need to need to add the following config option:

```javascript

const  createWindow  =  ()  =>  {
	const  mainWindow  =  new BrowserWindow({
	width:  1080,
	height:  720,
	webPreferences:  {     /// <-- Update this option.
		nodeIntegration:  true,			
		enableRemoteModule:  true,
		}
});

```

### Screen Recorder

Create a file named `src/app.js`. All code in this section runs in this file.

#### Get Available Screens

How do we access the available windows or screens to record? Electron has a built-in [desktopCapturer](https://www.electronjs.org/docs/api/desktop-capturer) that returns a list of the user’s screens.
```javascript

const videoSelectBtn = document.getElementById('videoSelectBtn');
videoSelectBtn.onclick = getVideoSources;

const { desktopCapturer, remote } = require('electron');
const { Menu } = remote;

// Get the available video sources
async function getVideoSources() {
  const inputSources = await desktopCapturer.getSources({
    types: ['window', 'screen']
  });
}

```
#### Display a Popup Menu

At this point we have a list of screens, but need a UI element for the user to select one. This is a good use-case for a popup menu.

```javascript

const { desktopCapturer, remote } = require('electron');
const { Menu } = remote;

async function getVideoSources() {
  const inputSources = await desktopCapturer.getSources({
    types: ['window', 'screen']
  });

  const videoOptionsMenu = Menu.buildFromTemplate(
    inputSources.map(source => {
      return {
        label: source.name,
        click: () => selectSource(source)
      };
    })
  );


  videoOptionsMenu.popup();
}

```

#### Preview Video Stream

Once a screen is selected it should be previewed in the video element. The code below uses `navigator.mediaDevices.getUserMedia` to turn the screen into a raw video feed.

A MediaRecorder instance is created to record the stream as a `webm` video file that can be played back.

```javascript

let mediaRecorder; // MediaRecorder instance to capture footage
const recordedChunks = [];

// Change the videoSource window to record
async function selectSource(source) {

  videoSelectBtn.innerText = source.name;

  const constraints = {
    audio: false,
    video: {
      mandatory: {
        chromeMediaSource: 'desktop',
        chromeMediaSourceId: source.id
      }
    }
  };

  // Create a Stream
  const stream = await navigator.mediaDevices
    .getUserMedia(constraints);

  // Preview the source in a video element
  videoElement.srcObject = stream;
  videoElement.play();

  // Create the Media Recorder
  const options = { mimeType: 'video/webm; codecs=vp9' };
  mediaRecorder = new MediaRecorder(stream, options);

  // Register Event Handlers
  mediaRecorder.ondataavailable = handleDataAvailable;
  mediaRecorder.onstop = handleStop;
}

```

#### Record and Save a Video File

The final step is to give the user control over the recording and saving of a video file.
```javascript

const { writeFile } = require('fs');
const { dialog, Menu } = remote;

const startBtn = document.getElementById('startBtn');
startBtn.onclick = e => {
  mediaRecorder.start();
  startBtn.classList.add('is-danger');
  startBtn.innerText = 'Recording';
};

const stopBtn = document.getElementById('stopBtn');

stopBtn.onclick = e => {
  mediaRecorder.stop();
  startBtn.classList.remove('is-danger');
  startBtn.innerText = 'Start';
};


// Captures all recorded chunks
function handleDataAvailable(e) {
  console.log('video data available');
  recordedChunks.push(e.data);
}

// Saves the video file on stop
async function handleStop(e) {
  const blob = new Blob(recordedChunks, {
    type: 'video/webm; codecs=vp9'
  });

  const buffer = Buffer.from(await blob.arrayBuffer());

  const { filePath } = await dialog.showSaveDialog({

    buttonLabel: 'Save video',
    defaultPath: `vid-${Date.now()}.webm`
  });

  console.log(filePath);

  writeFile(filePath, buffer, () => console.log('video saved successfully!'));
}

```




## Total Downloads
![GitHub All Releases](https://img.shields.io/github/downloads/AmitDeka/ScreenRecorder-ElectronJS/total?color=green)

### Downloads : 


[![](https://img.shields.io/github/v/release/AmitDeka/ScreenRecorder-ElectronJS?color=Green&label=Release%20For%20Windows)](https://github.com/AmitDeka/ScreenRecorder-ElectronJS/releases/download/v1.0.0/Screen.Recorder-1.0.0.Setup.exe)


### Latest Info

![GitHub last commit](https://img.shields.io/github/last-commit/AmitDeka/ScreenRecorder-ElectronJS)
![GitHub Release Date](https://img.shields.io/github/release-date/AmitDeka/ScreenRecorder-ElectronJS)


### Licensing
![GitHub](https://img.shields.io/github/license/AmitDeka/ScreenRecorder-ElectronJS)
