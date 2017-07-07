# electron-peer-connection

This npm module is a wrapper for RTCPeerConnection, suited for the inter-Electron BrowserWindow communication which does not require a STUN server for signalling. Signalling is done throught Electron's built-in ipc feature instead.
<br><br>

## Install
```
$ npm install --save electron-peer-connection
```
<br>

## Example

* main process
```javascript
const p2pChannel = require('electron-peer-connection').main;

p2pChannel.initChannel();
p2pChannel.addClient( { window: jitsiMeetWindow, name: 'jitsiMeetWindow' } );
p2pChannel.addClient( { window: microWindow, name: 'microWindow' } );
```

* window A (Sender)
```javascript
const WindowPeerConnection = require("electron-peer-connection").WindowPeerConnection;

let windowA = new WindowPeerConnection('windowA');
windowA.attachStream(stream);
windowA.sendStream('windowB');
```

* window B (Receiver)
```javascript
const WindowPeerConnection = require("electron-peer-connection").WindowPeerConnection;

let windowB = new WindowPeerConnection('windowB');
windowB.onReceivedStream(function (stream) {
    video.srcObject = stream;
});
```
<br>

## API

### Class: WindowPeerConnection
<hr>

#### Parameter

* <b>windowName</b>: the BrowserWindow’s name in main process

The WindowPeeerConnection wraps around webkitRTCPeerConnection class, which on construction in a renderer process, sets up a IPC listeners that receive message and data from other windows. All the message are relayed through the main process, and data are serialized before transmitted.

#### Method

* <b>attachStream ( streamToSend ) </b>: attaches a MediaStream object on the peer connection channel, which can be transmitted by sendStream method. Currently, it can only attach one stream per connection.
* <b>removeStream ( ) </b>: removes the MediaStream attached to this peer connection channel.
* <b>sendStream ( receiverName ) </b>: sends the attached MediaStream object to the target window specified the string ‘receiverName’. receiverName refers to the name of the variable assigned to the target BrowserWindow in the Electron’s main process.

#### Event Listener

* <b>onReceivedStream ( function(receivedStream) ) </b>: triggered when the WindowPeerConnection receives a MediaStream from a remote window.

<br>

### Main Process: Data Relay Channel
<hr>

A Javascript object that is called in the Electron’s main process. Since Electron does not support direct IPC between BrowserWindows, all messages and data must be relayed through the main process, using ipcMain. Each BrowserWindow is wrapped as a client in the data Channel.

#### Function

* <b>addClient ( client ) </b>: adds a client on the data relay channel. A client is a Javascript object with the following format: <br> client = { window: browerWindowObject, name: "BrowserWindowName" }

* <b>initChannel ( ) </b>: initiates the data relay channel with the current array of clients. Clients can be furthered added after the channel is initiated.

<br>

## License
MIT
