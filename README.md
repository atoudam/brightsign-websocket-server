# brightsign websocket server
 Websocket server that runs in a roHtmlWidget
 
The html code to be hosted in the roHtmlWidget:
 
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

The creation of the widget:

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
  }
}
```
