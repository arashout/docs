# Quick Start
This guide will walk you through setting up a Node.js project that listens to device activations and messages and responds to every 3rd message.

> In a hurry? Skip to [Connect](#connect) to get your application **region**, **Application ID** and **Access Key** and then jump to the [Live Example](#live-example) to run the script we'll write here directly in your browser!

## Setup
Let's install Node.js, create a Node.js project and install the TTN Client.

1.  [Download](https://nodejs.org/en/download/) and install Node.js.
2.  Create a new Node.js project:

    ```bash
    cd $HOME
    mkdir app
    cd app
    npm init
    ```

    > Just press return to accept the default answer to any question asked.

3.  Install and save the [TTN Client](http://flows.nodered.org/node/node-red-contrib-ttn) as dependency:

    ```bash
    npm install --save ttn
    ```

## Connect
Next, we will write the script that requires the TTN Client module and uses it to connect.

1.  Create the main script and open it in your favorite editor:

    ```bash
    touch index.js
    open index.js
    ```

2.  Require the TTN Client module:

    ```js
    var ttn = require('ttn');
    ```

3.  In the [dashboard](https://preview.dashboard.thethingsnetwork.org/applications), navigate to the application you'd like to connect to.

5.  In the editor, create an instance of the client:

    ```js 
    var region = 'eu';
    var appId = 'HELLO-WORLD';
    var accessKey = '2Z+MU0T5xZCaqsD0bPqOhzA6iygGFoi4FAgMFgBfXSo=';
    
    var client = new ttn.Client(region, appId, accessKey);
    ```

	Here's where you can find the values in the dashboard:
	
	* For `region`, copy the part following `ttn-handler-` for **Handler Status** in the **Application Overview** box.
	* For `appId`, copy the value for **Application ID** in the **Application Overview** box.
	* For `accessKey`, scroll down and then show and copy the value for **default key** in the **Access Key** box.

6.  Add a listener for the `connect` and `error` events to test the connection:

    ```js 
	client.on('connect', function(connack) {
		console.log('[DEBUG]', 'Connect:', connack);
	});
	
	client.on('error', function(err) {
		console.error('[ERROR]', err.message);
	});
    ```
 
7.  Run the script to test the connection:

    ```bash
    node .
    ```

    You should see something like:

    ```bash
	[DEBUG] Region: eu
	[DEBUG] Application ID: HELLO-WORLD
	[DEBUG] Application Access Key: 2Z+MU0T5xZCaqsD0bPqOhzA6iygGFoi4FAgMFgBfXSo=
	[DEBUG] Connect: Packet {
	  cmd: 'connack',
	  retain: false,
	  qos: 0,
	  dup: false,
	  length: 2,
	  topic: null,
	  payload: null,
	  sessionPresent: false,
	  returnCode: 0 }

    ```

    Use `Ctrl + C` to terminate the script.

    If you get an error it should say what is wrong:

    ```bash
    [ERROR] Connection refused: Not authorized
    ```

    > Make sure you have used the **Application ID** and not an **Application EUI** like version 1.x of the client.

## Receive Activations
Now that we are connected, let's listen for new device activations.

> Be aware that you will only receive `activation` events since the moment the script connects.

1.  Add a listener for the `activation` event:

    ```js
	client.on('activation', function(data) {
		console.log('[INFO] ', 'Activation:', data);
	});
    ```

2.  Run the script again:

    ```bash
    node .
    ```

3.  Power up, reset or upload a new sketch to a device to force it to activate and you should see something like:

    ```bash
    [INFO]  Activated: {
		"app_id": "HELLO-WORLD",
		"app_eui": "70B3D57ED0000AFB",
		"dev_id": "MY-UNO",
		"dev_eui": "0004A30B001B7AD2",
		"dev_addr": "260023BB",
		"metadata": {
			"time": "2016-09-07T12:43:17.97454032Z",
			"frequency": 867.1,
			"modulation": "LORA",
			"data_rate": "SF7BW125",
			"coding_rate": "4/5",
			"gateways": [{
				"eui": "0000024B08060112",
				"timestamp": 3546311603,
				"time": "2016-09-07T12:43:17.938537Z",
				"channel": 2,
				"rssi": -107,
				"snr": 1.2
			}]
		}
    }
    ```

    Use `Ctrl + C` to terminate the script.    

## Receive Messages
Now let's listen for actual messages coming in from devices.

I use the same script as [The Things Uno Quick Start](/uno/#quick-start), which both sends and receives messages.

1.  Add a listener for the `message ` event:

    ```js
	client.on('message', function(data) {
		console.info('[INFO] ', 'Message:', JSON.stringify(data, null, 2));
	});
    ```

2.  Run the script again:

    ```bash
    node .
    ```

    If you uploaded the Quick Start sketch you should see something like:

    ```bash
    [INFO]  Message: {
      "app_id": "HELLO-WORLD",
      "dev_id": "MY-UNO",
      "payload_fields": {
        "message": "Hello"
      },
      "payload_raw": "SGVsbG8=",
      "counter": 29,
      "port": 1,
      "metadata": {
        "frequency": 868.5,
        "datarate": "SF7BW125",
        "codingrate": "4/5",
        "gateway_timestamp": 326035051,
        "gateway_time": "2016-08-31T13:33:47.369434Z",
        "channel": 2,
        "server_time": "2016-08-31T13:33:47.384994459Z",
        "rssi": -71,
        "lsnr": 7.5,
        "rfchain": 1,
        "crc": 1,
        "modulation": "LORA",
        "gateway_eui": "B827EBFFFE87BD22",
        "altitude": 10,
        "longitude": 5.90418,
        "latitude": 52.95904
      }
    }
    ```

## Send Messages
To send a message you will have to address a specific device by its **Device ID**. Devices will only receive the last (downlink) message send to them in response to the next (uplink) message they send themselves. So let's send a (downlink) message in response to each 3rd (uplink) message we receive from a device.

Again, I use the same script as [The Things Uno Quick Start](/uno/#quick-start) as it is set up to both send and receives messages.

1.  Follow the [The Things Uno Quick Start](/uno/#quick-start) or upload another sketch to your device which sends messages every few seconds and listens for a response it will then print to `Serial`.

2.  In the Arduino IDE, select **Tools > Serial Monitor** `Ctrl/⌘ + Shift + M` to open the [Serial Monitor](/arduino/#serial-monitor).

3.  In the editor for the script, add another listener for the `uplink` event that responds to every 3rd message:

    ```js
	client.on('message', function(data) {
	
		// respond to every third message
		if (data.counter % 3 === 0) {
			console.log('[DEBUG]', 'Sending');
	
			var payload = new Buffer('4869', 'hex');
			client.send(data.dev_id, payload, data.port);
		}
	});
    ```

4.  Run the script again:

    ```bash
    node .
    ```

    After each 3rd message the script should output:

    ```
    [DEBUG] Sending
    ```

    In the Serial Montor you should see something like this if you uploaded the Quick Start sketch:

    ```
    Sending: mac tx uncnf 1 with 5 bytes
    Successful transmission. Received 2 bytes
    Received Hi
    ```

Congratulations! Now you know how to send and receive messages from a Node.js script.	