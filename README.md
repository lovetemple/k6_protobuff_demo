# k6_protobuff_demo
a demo implement of google protobuf with the k6[https://k6.io] load test tool

1. install [browserify](https://www.npmjs.com/package/google-protobuf) in global mode with  
```npm install -g browserify``` command

2. install [google-protobuf](https://www.npmjs.com/package/google-protobuf) using npm 

3. using browserify `browserify ./node_modules/google-protobuf/google-protobuf.js -s google-protobuf > google-protobuf.js`  [section-npm-modules](https://docs.k6.io/docs/modules#section-npm-modules) 

4. generate  `proto.js`  file with `protoc --js_out=import_style=commonjs,binary:. <your-proto-file-name>.proto`

5. change the 
```var jspb = require('google-protobuf');``` 
to
```var jspb = require('./google-protobuf.js');```
accorading your `google-protobuf.js` file path which geerated by `step 3`

6. now you can use your proto in `script.js` like below
```
import ws from "k6/ws";
import { check, crpyto } from "k6";

import proto from "./demo_pb.js"


export default function() {
    let url_array = ["ws://", "127.0.0.1:9999", "/ws"]
    var url = url_array.join("")
  
  var response = ws.connect(url, null, function (socket) {
    socket.on('open', function open() {
      console.log('connected');
      var message_proto = new proto.Message()
      message_proto.setText("hello")
      let message = message_proto.serializeBinary()
     
      socket.send(message);

      socket.setInterval(function timeout() {
        var hearbeat = new proto.Heartbeat()
        socket.send(hearbeat.serializeBinary());
        console.log("Heart beat every 15 sec");
      }, 15000);
    });

    socket.on('message', function (message) {
      console.log(`Received message: ${message}`);
    });

    socket.on('close', function () {
      console.log('disconnected');
    });

    socket.on('error', function (e) {
      if (e.error() != "websocket: close sent") {
        console.log('An unexpected error occured: ', e.error());
      }
    });

    socket.setTimeout(function () {
      console.log('2 seconds passed, closing the socket');
      socket.close();
    }, 30000);
  });

  check(response, { "status is 101": (r) => r && r.status === 101 });
}
```
