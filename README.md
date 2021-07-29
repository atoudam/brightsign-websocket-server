# brightsign websocket server
 Websocket server that runs in a roHtmlWidget
 
 # 1. HTML / JavaScript server
 
This code creates a websocket server listening on port 8080. This server relies on the "ws" node package and therefore node.js is required for this program.

When a message is received by the server it is broadcast to all connected clients, including the source of the message.
 
```html
<!DOCTYPE html>
<html>
<script>
    function init() {
        const WebSocketServer = require('/storage/sd/server/node_modules/ws').Server;

        let wss = new WebSocketServer({ port: 8080 });

        wss.on('connection', function connection(ws) {
            console.log (ws);
            ws.onmessage = function (event) {
                console.log(event);
                wss.clients.forEach(function each(client) {
                    console.log(client);
                    if (client.readyState === WebSocket.OPEN) {
                        client.send(event.data);
                    }
                });
            };
        });
    }
</script>

<head>
    <meta charset="utf-8">
    <title>Websocket Server</title>
    <base href=".">
</head>

<body onload=init()>
</body>

</html>
```

# 2. autorun.brs BrightScript Widget

This code creates an html widget that contains the websocket server. Note that the url may need to be changed to suit your use case. Add this to your autorun.brs. There is no roMessagePort requirement for this widget.

```brightscript
serverWidget = CreateObject ("roHtmlWidget", CreateObject("roRectangle", 0, 0, 1, 1), {
    brightsign_js_objects_enabled: true,
    inspector_server: { port: 3000 },
    nodejs_enabled: true,
    url: "file:/server/index.html"
})

print "type serverWidget", type(serverWidget)
if type(serverWidget) <> "roHtmlWidget"
    print "ERROR - Failed to create http widget for server"
    Sleep(5)
    Reboot()
end if

serverWidget.Show()
```

# 3. websocket client

This code creates javascript function object that manages a connection to the above websocket server. This example client connects over the local network to a server running on a brightsign with local ip address '192.168.123.198'.

the 'socket' object handles disconnections by retrying a connection once per second.

The 'socket' object supplies two public functions:

socket.connect () :  creates and connects the websocket. In this case the method could be made private as the connection is initialised inside the socket object. As the 'socket' handles disconnections it is not necessary to call the connect method from outside the object.

socket.send (message) : sends a message to the server. The message should be of type object.

The 'socket' has a private function that delivers preprocessed messages to the client application:

socket.websocket.onmessage (event) : Receives messages from the server and processes them into javascript objects. These messages are sent to an function external to the 'socket' object (in this case handleEvent) in order to apply state changes to the client.

```javascript
let socket = function () {
    let websocket;

    let publicAPIs = {};

    publicAPIs.connect = function () {
        websocket = new WebSocket("ws://192.168.123.198:8080");

        websocket.onopen = function () {
            console.log('CONNECTED');
            websocket.send(JSON.stringify({
                type: 'message',
                message: 'hello from iPad interface'
            }));
        };

        websocket.onmessage = function (event) {
            let message = JSON.parse(event.data);
            if (message.type == 'event') {
                handleEvent(message.event, message.value);
            }
        };

        websocket.onclose = function () {
            console.log("Reconnecting ...");
            setTimeout(function () {
                publicAPIs.connect();
            }, 1000);
        }
    }

    publicAPIs.send = function (message) {
        websocket.send(message);
    };

    publicAPIs.connect();

    return publicAPIs;
}();

function handleEvent(type, value) {
	// handle event here
}
```
