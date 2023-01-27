# node-ffmpeg-stream
To convert rtsp stream to websocket stream for multi view purpose
1. [RTSP Stream to Websocket](#rtsp-stream-to-websocket)
2. [RTSP Stream to Recording(Download & Storing into the file location)](#rtsp-stream-to-recording)
3. [RTSP Stream to Screenshot(Take picture from Stream)](#rtsp-stream-to-screenshot)
4. [RTSP Stream to Picturestream(Get picture by picture)](#rtsp-stream-to-picturestream)
5. [RTSP Stream to MQTT(Publish stream to MQTT Broker)](#rtsp-stream-to-mqtt)

## Usage
```
npm i @xdcobra/node-ffmpeg-stream
```

## RTSP Stream to Websocket

### On server
```
Stream = require('@xdcobra/node-ffmpeg-stream').WebsocketStream;
stream = new Stream({
  name: 'name',//name that can be used in future  
  url: 'rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4',  //Stream URL
  wsPort: 7070,  //Streaming will be transferred via this port
  options: { // ffmpeg options 
    '-stats': '', // print progress report during encoding, can be no value ''
    '-r': 30 // frame rate (Hz value, fraction or abbreviation) - By default it set to 30 if no value specified
  }
})
  
   
```
#### To stop running stream 

This method should be called when you have `0 socket connection` or to disconnect to all the viewers(sockets).
```
stream.stop();	
```
##### Socket Connection Image
![CMD Running Snapshot](/assets/screenshot/stream.running.PNG)



### On Client
```
<html>
<body>
	<canvas id="stream"></canvas>
</body>

<script type="text/javascript" src="jsmpeg.min.js"></script>
<script type="text/javascript">
	player = new JSMpeg.Player('ws://localhost:7070', {
	  canvas: document.getElementById('stream') // stream should be a canvas DOM element
	})	
</script>
</html>
```

## RTSP Stream to Recording
### On server
```
RecordNSnap = require('@xdcobra/node-ffmpeg-stream').RecordNSnap;  

var input ={
    "url": "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4",
    "second": "60",                 // by default 5 sec
    "filePath": "D:\\Records\\",    // by default ""
    "fileName": "test5.mp4",        // your output filename with extension like mp4,avi & et.,
    "fileMimeType":"video/mp4",     // Mime type for download file via javascript & advanced javascript
    "recordType":"download"         // can be download or store
}
objRecordNSnap=new RecordNSnap();
 objRecordNSnap.recordVideo(input,function(resp){	
	res.send(resp);
 })
  
   
```
### File download output
```
input = {
    "url": "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4",
    "second": "1",
    "filePath": "F:\\",
    "fileName": "test5.mp4",
    "fileMimeType":"video/mp4",
    "recordType":"download"
}

```

![CMD File Download Output](/assets/screenshot/record.download.PNG)

### File store output
```
input = {
    "url": "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4",
    "second": "1",
    "filePath": "F:\\",
    "fileName": "test5.mp4",
    "fileMimeType":"video/mp4",
    "recordType":"store"
}

```
![CMD File Store Output](/assets/screenshot/record.store.PNG)

## RTSP Stream to Screenshot
### On server
```
RecordNSnap = require('@xdcobra/node-ffmpeg-stream').RecordNSnap; 


var input = {
    "url": "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4",    
    "imagePath": "F:\\",              // by default ""
    "imageName": "test2.jpeg",        // your output filename with extension like mpeg,png & etc.,
    "imageMimeType":"image/jpeg",     // Mime type for download file via javascript & advanced javascript
    "snapType":"download"             // can be download or store
}

snap = new RecordNSnap();
snap.takeSnap(input,function(resp){	
	res.send(resp);
 });


```
### Image download output
```
input = {
    "url": "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4",    
    "imagePath": "F:\\",
    "imageName": "test2.jpeg",
    "imageMimeType":"image/jpeg",
    "snapType":"download"
}

```

![CMD File Download Output](/assets/screenshot/snap.download.PNG)

### Image store output
```
input = {
    "url": "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4",    
    "imagePath": "F:\\",
    "imageName": "test2.jpeg",
    "imageMimeType":"image/jpeg",
    "snapType":"store"
}

```
![CMD File Store Output](/assets/screenshot/snap.store.PNG)

## RTSP Stream to Picturestream
First, define the options for creating a picture stream.
```
PictureStream = require("@xdcobra/node-ffmpeg-stream").PictureStream;

var input = {
    "name": "BigBunny",
    "url": "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4",
    "fps": 30,
    "ffmpegOptions": {
        "-vf": "scale=200:160",
    }
}
```

Then create the callbacks for the picture stream object. On the data callback, you receive the .png image for every frame.
```
// create stream
pictureStream = new PictureStream();

// callbacks
pstream = pictureStream.startStream(input)
pstream
    .on("data", data => {
        console.log(data)
    })
    .on("error", error => {
        console.log(error)
    })
```


#### To stop a running pictureStream 

This method should be called when you don't want to capture pictures from the rtsp stream anymore.
```
pstream.stopStream();
```

### To restart a stopped pictureStream
Once a pictureStream was stopped with stopStream(), you can simply restart it again by using the startStream(input) function again. See this example, where a stream was started, stopped and then restarted.
You don't even need to set the callbacks again, as they are still set.
```
// create stream options
var input = {
    "name": "BigBunny",
    "url": "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4",
    "fps": 30,
    "ffmpegOptions": {
        "-vf": "scale=200:160",
    }
}

// create stream
pictureStream = new PictureStream();

// callbacks
pstream = pictureStream.startStream(input)
pstream
    .on("data", data => {
        // encode to base64
        base64 = Buffer.from(data).toString('base64');

        // publish
        mqttClient.publish("televersuch/image", base64)

        console.log("Published!")
    })
    .on("error", error => {
        console.log(error)
    })

// stop stream
pstream.stopStream();

// restart stream
pstream.startStream(input)
```

## RTSP Stream to MQTT
You are able to publish an RTSP Stream via MQTT. In this example, I stream a RTSP Stream to a Browser via MQTT.

### On server
First, define the options for creating a picture stream.
```
PictureStream = require("@xdcobra/node-ffmpeg-stream").PictureStream;
var mqtt = require("mqtt");

var input = {
    "name": "BigBunny",
    "url": "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4",
    "fps": 30,
    "ffmpegOptions": {
        "-vf": "scale=200:160",
    }
}
```

Then create the callbacks for the picture stream object. On the data callback, you receive the .png image for every frame.
Don't forget to stop the picturestream (picturestream.stop()), once you don't want to record the pictures from the rtsp stream anymore.
```
// create stream
pictureStream = new PictureStream();

// create mqtt client
mqttClient = mqtt.connect("ws://127.0.0.1:9007", {})    // simply switch between ws or mqtt protocol
    .on("connect", function () {
        console.log("connected");
    });

// start the stream
pstream = pictureStream.startStream(input)

// callbacks
pstream
    .on("data", data => {
        // encode to base64, to show it later in browser
        base64 = Buffer.from(data).toString('base64');

        // publish
        mqttClient.publish("my/exampleTopic", base64)
    })
    .on("error", error => {
        console.log(error)
    })
```

### On client javascript (client_backend.js)
```
window.onload = async function () {
    m = new mqtt_fetch("my");
    await m.init("localhost", 9007); // MQTT over websockets!!
    m.set_callback("my/exampleTopic", showImage, false);
}

function showImage(data) {
    if (document.getElementById("finalImage") != null) {
        document.getElementById("finalImage").setAttribute("src", "data:image/png;base64, " + data);
    } else {
        let node = document.createElement("img");
        node.setAttribute("id", "finalImage");
        node.setAttribute("src", "data:image/png;base64," + data);
        document.getElementById("stream").appendChild(node);
    }
}
```

### On client html
```
<!DOCTYPE html>
<html>
<head>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js"></script>
    <script type="text/javascript" src="./mqtt-fetch-paho.js"></script>
    <script type="text/javascript" src="./client_backend.js"></script>
</head>
<body>
    <div id="stream">

        </div>
</body>
</html>
```

## Dependencies

1. [FFMPEG.exe](https://ffmpeg.org/download.html) - exe file should be in same place of js file
2. [jsmpeg.min.js](https://cdnjs.com/libraries/jsmpeg)
3. [mqtt-fetch.js](https://github.com/informatik-aalen/mqtt-fetch.js) -  make sure you use mqtt-fetch-paho.js
