# xPico Wi-Fi WebAPI Javascript library

## Summary
A Javascript library that makes it easy to use the xPico Wi-Fi's WebAPI to access the hardware features of the xPico Wi-Fi.
For example, you can send and receive data from the serial ports, update configuration, and toggle GPIOs (future) with simple
calls. The library takes care of all the HTTP transactions and XML building and parsing for you.

Please note, this library uses a new feature, Web to Serial, which is in Alpha on versions 1.4 of the xPico Wi-Fi. Please see the Lantronix wiki for more details: http://wiki.lantronix.com/developer/XPicoWiFi/WebAPItoDevice

See the section on adding your own webpages to learn how to load custom web pages onto the xPico Wi-Fi's filesystem:
http://wiki.lantronix.com/developer/XPicoWiFi/CustomizeWebPages#Creating_your_own_webpages

## Samples
### serial.html
This file shows how to send data to the serial port by clicking on a color. It also reads data from the serial port and displays it on
the webpage. It has a sample of reading the configuration of the serial port and also setting the configuration of the serial port.

![Serial screenshot](/screenshots/serial.jpg?raw=true)

### chat.html
This file shows how to send and receive data to the serial port in a single transaction. This uses the new "Command" feature in firmware version 1.4.0.0R21 or later.

The reason to use this instead of the separate Transmit and Receive from the previous example is to make sure that the same client sending data receives a response.

![Chat screenshot](/screenshots/chat.png?raw=true)

### gpio.html
This files shows how to read/write to the GPIO pins on the xPico Wi-Fi (also called CP for Configurable Pins). You can turn them to Input, Output, read	the current value, or set the output value.

![GPIO screenshot](/screenshots/gpio.jpg?raw=true)

### quickconnect.html
This file shows how to create your own custom QuickConnect web page. This is useful if you want to present the user a page for provisioning WLAN Profiles either on the same page as the rest of your page, or if you want to change the CSS significantly, for example to make it more mobile-friendly.

![QuickConnect screenshot](/screenshots/quickconnect.png?raw=true)

## How-To

Put the `xpwLib.js` file on the xPico Wi-Fi's filesystem, in the http/ directory (which you need to create). Add it to the header of your HTML file like this:

```
<script src="xpwLib.js"></script>
```

This will load the `xpw` object which contains a number of functions to access the xPico Wi-Fi WebAPI.

If you would like to reduce the size that the library takes on the xPico Wi-Fi's filesystem, you should minify and gzip the library. To minify it, you can use any of a number of tools. We have used http://jscompress.com with good results. Then use gzip to create a .gz file.

 Place the `xpwLib.min.js.gz` (or whatever you called it) file in the filesystem instead (also in the http/ directory).

Note that when adding a GZIP'ed file, you don't specify the .gz extension when you load it via the browser. The xPico Wi-Fi will automatically serve .gz files if the browser supports GZIP encoding (which all modern browsers do). So to include the library in your HTML file, you would add this to the header:

```
<script src="xpwLib.min.js"></script>
```

## Available functions

### Send data to the serial port

```
xpw.serialTransmit({
		line: "1",		// could also be "2" or "CDC_ACM" (new 1.5 firmware), if ommitted or some other value,
                  // defaults to "1"
		message: "This will go out to the serial port\r\n",			// required
		callback: function(data) {
				// Do something, data is an object with a boolean called success:
				// {success: true}
			}
		});
```

### Receive data from the serial port

```
xpw.serialReceive({
		line: "1",		// could also be "2" or "CDC_ACM" (new 1.5 firmware), if ommitted or some other value,
                  // defaults to "1"
		done: function(data) {			// optional
				// Do something, data is an object with the following:
				// { success: true|false, message:The data that came from the serial port}
			}
	});
```

### Make a send/receive transaction to the serial port

```
xpw.serialTransact({
		line: "1",		// could also be "2" or "CDC_ACM" (new 1.5 firmware), if ommitted or some other value,
                  // defaults to "1"
		message: "This will go out to the serial port\r\n",			// required
		n: "10",		// Optional, number of characters to wait for (default unlimited)
		m: "1000",		// Optional, number of milliseconds to wait for (default 1000)
		t: "0D",		// Optional, terminating character in Hex (0D for carriage return) (default None)
		done: function(data) {			// optional
				// Do something, data is an object with the following:
				// { success: true|false, message:The data that came from the serial port}
			}
	});
```

Uses feature new to firmware 1.4.0.0R21 or newer. See information here:
http://wiki.lantronix.com/developer/XPicoWiFi/WebAPItoDevice#Sending_and_receiving_in_one_transaction

### Get serial port configuration

```
xpw.getLineConfig({
		line: "1",		// could also be "2" or "CDC_ACM" (new 1.5 firmware), if ommitted or some other value,
                  // defaults to "1"
		done: function(data) {				// optional
				// Do something, data is an object with the following:
				// { success: true|false,
				//   lineConfig: 			//    Another object with the serial port data, for example:
				//  {
				//		Baud Rate: "115200 bits per second",
				//		Data Bits: "8",
				//		Flow Control: "None",
				//		Gap Timer: "<Four Character Periods>",
				//		Parity: "None",
				//		Protocol: "None",
				//		State: "Enabled",
				//		Stop Bits: "1",
				//		Threshold: "56 bytes"
				//	}
				// }
			}
	});
```

### Set serial port configuration

```
xpw.setLineConfig({
		line: "1", 		// Must be "1", "2" or "CDC_ACM" (new 1.5 firmware)
		items: {		// Required object with the properties that need to change, for example:
					{ Protocol: "None",
					  Baud Rate: "9600 bits per second"}	// Available items are the same as returned by the getLineConfig
				},
		done: function(data) {			// optional
				// Do something, data is an object with the following:
				// { success: true|false, message:The data that came from the serial port}
			}
});
```

### Get GPIO status

```
xpw.getGpioStatus({
	done: function(data) {				// Optional
				// Do something, data is an object with the following:
				// { success: true|false,
				//   status : [					// A list of objects for each GPIO line
				//   	  { gpio: "1",			// GPIO number
				//				isOutput: true|false,
        //				assertLow: true|false,
				//				isAsserted: true|false,
				//				enabled: true|false,
        //        pullup: true|false,  // if false, CP is set to push-pull
				//			},
				//		...]
				//	}
	}
});
```

### Set GPIO

```
xpw.setGpio({
	gpio: "1",			// Must be 1 through 8, for the specific CP pin
	command: "disable",    // Can be: disable, input, output, assert, unassert, assertHigh, assertLow, pullup, push-pull
	done: function(data) {
			// Do something, data is an object with a boolean called success:
			// {success: true}
		}
	});
```

### Start a scan of Wi-Fi networks

```
xpw.startScan();
```

### Retrieve the results from the last scan

```
xpw.getScanResults({
	done: function(data) {
		// Data is an object that contains:
		//  { success: true|false,
		//    networks: [  	// A list of objects for each access point scanned
		//			{ ssid: "ssid",		// The SSID of the network
		//        bssid: "bssid",	// The BSSID of the access point
		//				channel: "1",		// The Wi-Fi channel used
		//				rssi: "-50dB",	// The RSSI of this signal
		//				flags: "WPA2-CCMP"   // A string that shows the security of the network
		//		   },
		//     ...]
		//   }
	}
	});
```

### Retrieve the WLAN Profiles stored in configuration

```
xpw.getWlanProfiles({
	done: function(data) {
		// Data is an object that contains:
		//  { success: true|false,
		//    profiles: [  	// A list of objects for each access point scanned
		//			{ name: "name" },		// The name of the profile
		//     ...]
		//   }
	}
	});
```

### Add a new WLAN Profile

```
xpw.addWlanProfile({
	network:   // Must be a network object with the same properties as returned
						 // by getScanResults
	password:  // A string with the WPA2/WPA passphrase or WEP Hex key (10 or 26
						 // characters). Not needed if there is no security
	done:			 // A function to be called when configuration is done
	});
```

### Delete WLAN Profile

```
xpw.deleteWlanProfile({
	profile:   // Must be a profile object with the same properties as returned
						 // by getWlanProfiles
	done:			 // A function to be called when configuration is done
	});
```
