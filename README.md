# brightsign websocket server
 Websocket server that runs in a roHtmlWidget
 
 Here is an example of connecting from a 

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
