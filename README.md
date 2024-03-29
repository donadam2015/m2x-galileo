Intel Galileo M2X API Client
=====================

The Intel Galileo library is used to send/receive data to/from [AT&amp;T's M2X service](https://m2x.att.com/) from [Intel Galileo](http://arduino.cc/en/ArduinoCertified/IntelGalileo) based devices.

**NOTE**: Unless stated otherwise, the following instructions are specific to Gen 1 boards. If you are using other boards, the exact steps may vary.

Getting Started
==========================
1. Signup for an [M2X Account](https://m2x.att.com/signup).
2. Obtain your _Master Key_ from the Master Keys tab of your [Account Settings](https://m2x.att.com/account) screen.
3. Create your first [Device](https://m2x.att.com/devices) and copy its _Device ID_.
4. Review the [M2X API Documentation](https://m2x.att.com/developer/documentation/overview).
5. Obtain an Intel Galileo with a compatible Mini PCI-E wifi or ethernet.

Please consult the [M2X glossary](https://m2x.att.com/developer/documentation/glossary) if you have questions about any M2X specific terms.

How to Install the library
==========================

This library depends on [jsonlite](https://github.com/citrusbyte/jsonlite), the installation steps are as follows:

1. Clone the repository and sync submodules:

   ```
   $ git clone https://github.com/attm2x/m2x-arduino
   $ git submodule update --init
   ```

   **NOTE**: You might notice that we add jsonlite as a submodule, this is due to the fact that we are still using an older version of jsonlite now. We might remove this restriction once we migrate to latest version.

2. Open the Intel Galileo IDE, click `Sketch->Import Library...->Add Library...`, then navigate to `vendor/jsonlite/amalgamated/jsonlite` folder in the repository. The jsonlite library will be imported to Intel Galileo this way.

   **NOTE**: There will be three (3) folders named jsonlite:
   * `vendor/jsonlite`: the repo folder
   * `vendor/jsonlite/jsonlite`: the un-flattened jsonlite source folder
   * `vendor/jsonlite/amalgamated/jsonlite`: the flattened jsonlite source for arduino

   You should use the final library listed here as the first two won't work!
3. Use the instructions outlined in Step 2 above to import the `M2XStreamClient` library in the repository.
4. Now you can find M2X examples under `File->Examples->M2XStreamClient`

WiFi Hardware Setup & SSH Connection
=======================

Getting WiFi working on the Galileo may require some additional steps. There are several helpful instructionals that we recommend:
  * http://bentuino.com/galileo-goes-wireless/
  * http://www.malinov.com/Home/sergey-s-blog/intelgalileo-addingwifi
  * http://www.hackshed.co.uk/how-to-use-wifi-with-the-intel-galileo/

PCI WiFi Drivers
----------------

http://wireless.kernel.org/en/users/Drivers/iwlwifi

Troubleshooting
---------------

#### How do I transfer a file over SSH into the Galileo?

```
scp /path/to/file root@1.1.1.1:/lib/firmware/
```

Replace [1.1.1.1] with the Galileo's IP Address

---

#### How do I know my mini PCIe card is being detected?

You can run the following command to look for a PCIe card that is plugged in:
```
lspci -k | grep wifi
```

And you will see something like:
```
01:00.0 Class 0280: 8086:4235 iwlwifi
```

Referenced from: https://communities.intel.com/message/225732

---

#### ifconfig: SIOCGIFFLAGS: No such device
```
wlan0: Failed to initialize driver interface
ifconfig: SIOCGIFFLAGS: No such device
```

##### Solution

Drivers are not properly installed or device needs to be restarted so drivers can initialize.

---

#### No lease, failing
```sh
root@clanton:~# ifup wlan0
udhcpc (v1.20.2) started
Sending discover...
Sending discover...
Sending discover...
No lease, failing
```
##### Solution
```sh
killall wpa_supplicant
killall iwlwifi
ifup wlan0
```

Sensor Setup
------------

Different sensors can be hooked up to an Intel Galileo board to provide different properties including temperatures, humidity, etc. You can use a breadboard as well as wires to connect different sensors to your Intel Galileo. For a detailed tutorial on connecting different sensors, please refer to the Intel Galileo [Examples page](http://arduino.cc/en/Tutorial/HomePage) as the Intel Galileo has the same pin outs.


Variables used in Examples
==========================

In order to run the given examples, different variables need to be configured. We will walk through those variables in this section.

Network Configuration
---------------------

If you are using a Wifi PCI Card, the following variables need configuration:

```
char ssid[] = "<ssid>";
char pass[] = "<WPA password>";
```

Just fill in the SSID and password of the Wifi hotspot and you should be good to go.

For onboard Ethernet, the following variables are needed:

```
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
IPAddress ip(192,168,1,17);
```

You can use any MAC address, as long as it does not conflict with another network device within the same LAN.

The IP address here is only used when DHCP fails to give a valid IP address. It is recommended, though not required, to provide a unique IP address.

M2X API Key
-----------

Once you [register](https://m2x.att.com/signup) for an AT&amp;T M2X account, an API key is automatically generated for you. This key is called a _Primary Master Key_ and can be found in the _Master Keys_ tab of your [Account Settings](https://m2x.att.com/account). This key cannot be edited or deleted, but it can be regenerated. It will give you full access to all APIs.

However, you can also create a _Device API Key_ associated with a given Device, you can use the Device API key to access the streams belonging to that Device.

You can customize this variable in the following line in the examples:

```
char m2xKey[] = "<M2X access key>";
```

Device ID
-------

A device is a source of data (it could be a physical device, a virtual device, a service, or an application). It is a set of data streams, such as streams of locations, temperatures, etc. The following line is needed to configure the device used:

```
char deviceId[] = "<device id>";
```

Stream Name
------------

A stream in a device is a set of times series data of a specific type (i,e. humidity, temperature). You can use the M2XStreamClient library to send stream values to M2X server, or receive stream values from M2X server. Use the following line to configure the stream if needed:

```
char streamName[] = "<stream name>";
```

Using the M2XStreamClient library
=========================

The M2X Galileo library can be used with both a Wifi connection and an Ethernet connection. For a Wifi connection, use the following code:

```
WiFiClient client;
M2XStreamClient m2xClient(&client, m2xKey);
```

For an Ethernet connection, use the following code:

```
EthernetClient client;
M2XStreamClient m2xClient(&client, m2xKey);
```

In the M2XStreamClient, the following API functions are provided:

* `updateStreamValue`: Send stream value to M2X server
* `postDeviceUpdates`: Post values from multiple streams to M2X server
* `listStreamValues`: Receive stream value from M2X server
* `updateLocation`: Send location value of a device to M2X server
* `readLocation`: Receive location values of a device from M2X server
* `deleteValues`: Delete stream values from M2X server

Returned values
---------------

For all those functions, the HTTP status code will be returned if we can fulfill an HTTP request. For example, `200` will be returned upon success, and `401` will be returned if we didn't provide a valid M2X API Key. A full-list of M2X API error codes can be found here: [M2X API Error Codes] (https://m2x.att.com/developer/documentation/overview#Client-Errors)

Otherwise, the following error codes will be used:

```
static const int E_NOCONNECTION = -1;
static const int E_DISCONNECTED = -2;
static const int E_NOTREACHABLE = -3;
static const int E_INVALID = -4;
static const int E_JSON_INVALID = -5;
```

Update stream value
-------------------

The following functions can be used to post one single value to a stream, which belongs to a device:

```
template <class T>
int updateStreamValue(const char* deviceId, const char* streamName, T value);
```

Here we use C++ templates to generate functions for different types of values, feel free to use values of `float`, `int`, `long` or even `const char*` types here.

Post device updates
-------------------

M2X also supports posting multiple values to multiple streams in one call, use the following function for this:

```
template <class T>
int postDeviceUpdates(const char* deviceId, int streamNum,
                      const char* names[], const int counts[],
                      const char* ats[], T values[]);
```

Please refer to the comments in the source code on how to use this function, basically, you need to provide the list of streams you want to post to, and values for each stream.

List stream values
------------------

Since mbed microcontroller contains very limited memory, we cannot put the whole returned string in memory, parse it into JSON representations and read what we want. Instead, we use a callback-based mechanism here. We parse the returned JSON string piece by piece, whenever we got a new stream value point, we will call the following callback functions:

```
typedef void (*stream_value_read_callback)(const char* at,
                                           const char* value,
                                           int index,
                                           void* context,
                                           int type);
```

The implementation of the callback function is left for the user to fill in, you can read the value of the point in the `value` argument, and the timestamp of the point in the `at` argument. We even pass the index of this this data point in the whole stream as well as a user-specified context variable to this function, so as you can perform different tasks on this.

`type` indicates the type of value stored in `value`: 1 for string, 2 for number. However, keep in mind that `value` will always be a pointer to an array of char, even though `type` indicates the current value is a number. In this case, `atoi` or `atof` might be needed.

To read the stream values, all you need to do is calling this function:

```
int listStreamValues(const char* deviceId, const char* streamName,
                     stream_value_read_callback callback, void* context,
                     const char* query = NULL);
```

Besides the device ID and stream name, only the callback function and a user context needs to be specified. Optional query parameters might also be available here, for example, the current query parameter picks value from a specific range:

```
start=2014-10-01T00:00:00Z&end=2014-10-10T00:00:00Z
```

Update Device Location
--------------------------

You can use the following function to update the location for a device:

```
template <class T>
int updateLocation(const char* deviceId, const char* name,
                   T latitude, T longitude, T elevation);
```

Different from stream values, locations are attached to devices rather than streams. We use templates here, since the values may be in different format, for example, you can express latitudes in both `double` and `const char*`.

Read Device Location
------------------------

Similar to reading stream values, we also use callback functions here. The only difference is that different parameters are used in the function:

```
void (*location_read_callback)(const char* name,
                               double latitude,
                               double longitude,
                               double elevation,
                               const char* timestamp,
                               int index,
                               void* context);

```

For memory space consideration, now we only provide double-precision when reading locations. An index of the location points is also provided here together with a user-specified context.

The API is also slightly different, in that the stream name is not needed here:

```
int readLocation(const char* deviceId, location_read_callback callback,
                 void* context);

```

Delete stream values
--------------------

The following function can be used to delete stream values within a date range:

```
int deleteValues(const char* deviceId, const char* streamName,
                 const char* from, const char* end);
```

`from` and `end` fields here follow ISO 8601 time format.

Examples
========

We provide a series of examples that will help you get an idea of how to use the `M2XStreamClient` library to perform all kinds of tasks.

Note that the examples contain fictionary variables, and that they need to be configured as per the instructions above before running on your Intel Galileo board. Each of the examples here also needs either a Wifi PCI Card or onboard Ethernet hooked up to your device.

In the `GalileoPost`, `EthernetGalileoPost`, a temperature sensor, a breadboard and 5 wires are also needed to get temperature data.

After you have configured your variables and the board, plug the Intel Galileo board into your computer via a Micro-USB cable, click `Verify` in the Intel Galileo IDE, then click `Upload`, and the code should be uploaded to the board. You can check all the outputs in the `Serial Monitor` of the Intel Galileo IDE.

GalileoPost
-------

This example shows how to post temperatures to M2X.

GalileoPostMultiple
---------------

This example shows how to post multiple values to multiple streams in one API call.

GalileoFetchValues
--------------

This example reads stream values from M2X server and prints the stream data point to Serial interface. You can find the actual values in the Intel Galileo `Serial Monitor`.

GalileoReadLocation
---------------

This example reads location data of a device from M2X, and prints them to Serial interface. You can check the output in the `Serial Monitor` of the Intel Galileo IDE.

GalileoUpdateLocation
-----------------

This example sends location data to M2X. Ideally a GPS device should be used here to read the coordinates, but for simplicity, we just use pre-set values here to show how to use the API.

GalileoDelete
---------

This example shows how to delete values within a stream by providing a date/time range.

EthernetGalileoPost
---------------

This example is similar to the `GalileoPost`, except that EthernetClient is used instead of WifiClient. If you are using the onboard Ethernet instead of a Wifi PCI Card, you can use this example.

EthernetGalileoReceive
------------------

This example is similar to the `GalileoReceive`, except that EthernetClient is used instead of WifiClient.

LICENSE
=======

This library is released under the MIT license. See [`M2XStreamClient/LICENSE`](M2XStreamClient/LICENSE) for the terms.
