# MQTT adapter

MQTT adapter provides an MQTT API for sending and receiving messages through the 
platform.

## Configuration

The service is configured using the environment variables presented in the
following table. Note that any unset variables will be replaced with their
default values.

| Variable             | Description         | Default               |
|----------------------|---------------------|-----------------------|
| MF_MQTT_ADAPTER_PORT | Service MQTT port   | 1883                  |
| MF_MQTT_WS_PORT      | WebSocket port      | 8880                  |
| MF_NATS_URL          | NATS instance URL   | nats://localhost:4222 |
| MF_THINGS_URL        | Things service URL  | localhost:8181        |

## Deployment

The service is distributed as Docker container. The following snippet provides
a compose file template that can be used to deploy the service container locally:

```yaml
version: "2"
services:
  mqtt:
    image: mainflux/mqtt:[version]
    container_name: [instance name]
    ports:
      - [host machine port]:[configured port]
    environment:
      MF_THINGS_URL: [Things service URL]
      MF_NATS_URL: [NATS instance URL]
      MF_MQTT_ADAPTER_PORT: [Service MQTT port]
      MF_MQTT_WS_PORT: [Service WS port]
```

To start the service outside of the container, execute the following shell script:

```bash
# download the latest version of the service
go get github.com/mainflux/mainflux

cd $GOPATH/src/github.com/mainflux/mainflux/mqtt

# install dependencies
npm install

# set the environment variables and run the service
MF_THINGS_URL=[Things service URL] MF_NATS_URL=[NATS instance URL] MF_MQTT_ADAPTER_PORT=[Service MQTT port] MF_MQTT_WS_PORT=[Service WS port] node mqtt.js ..
```

## Usage

To use MQTT adapter you should use `channels/<channel_id>/messages`. Client key should
be passed as user's password. If you want to use MQTT over WebSocket, you could use
[Paho client](https://www.eclipse.org/paho/):

```
<!DOCTYPE html>
<html>
  <head>
  <meta http-equiv="Content-Type" content="text/html;charset=utf-8"/>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.2/mqttws31.min.js" type="text/javascript"></script>
  <script type="text/javascript">
    var wsbroker = "localhost";  //mqtt websocket enabled broker
    var wsport = 443 // port for above

    var client = new Paho.MQTT.Client(wsbroker, Number(wsport), '');

    client.onConnectionLost = function (responseObject) {
      console.log("connection lost: " + responseObject.errorMessage);
    };

    client.onMessageArrived = function (message) {
      console.log("Msg", message.destinationName, ' -- ', message.payloadString);
    };

    var options = {
      timeout: 3,
      userName: "<thing_id>",   // Replace <:your_thing_id>
      password: "<thing_key>",  // Replace <:your_thing_key>
      useSSL: true,
      onSuccess: function () {
        console.log("mqtt connected");
        // Connection succeeded; subscribe to our topic, you can add multile lines of these
        client.subscribe('channels/<channel_id>/messages', {qos: 1});  // Replace <:your_channel_id>

        // use the below if you want to publish to a topic on connect
	      var payload = [
	        {
	          "bn":"e35b157f-21b8-4adb-ab59-9df21461c815",
	          "bt":1.276020076001e+09,
	          "bu":"A",
	          "bver":5,
	          "n":"voltage",
	          "u":"V",
	          "v":120.1
	        },
	        {
	          "n":"current",
	          "t":-5,
	          "v":1.2
	        },
	        {
	          "n":"current",
	          "t":-4,
	          "v":1.3
	        }
	      ];
        var message = new Paho.MQTT.Message(JSON.stringify(payload));
        message.destinationName = "channels/<channel_id>/messages";  // Replace <:your_channel_id>
        client.send(message);
      },
      onFailure: function (message) {
        console.log("Connection failed: " + message.errorMessage);
      }
    };

    function init() {
      client.connect(options);
    }

  </script>
  </head>
  <body onload="init();">
  </body>

</html>
```
