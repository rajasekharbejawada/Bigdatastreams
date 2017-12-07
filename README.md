# IoT Connected Car Sample
Modified from the IoT-Connected-Vehicle-Starter-Kit (https://github.com/ibm-messaging/iot-connected-vehicle-starter-kit). To be used in IBM Bluemix IoT hands-on workshops.

To get instructables for connecting popular physical devices or sensors, see: https://developer.ibm.com/recipes

## Introduction
The "IoT Connected Car Sample" is a modified version from "Connected Vechicle Starter Kit" and it consists of three applications:

1. A Node.js connected vehicle simulator
2. An OpenLayers HTML5 map application
3. An HTML5 tester application to send commands

The applications use IBM IoT Foundation for real-time publish/subscribe messaging, using the MQTT protocol.  The simulated devices (vehicles) publish telemetry data frequently (5 msgs/sec) and subscribe to a command topic whereby they accept commands from the tester application (ex. set speed to 60).  The Map app subscribes to the vehicle telemetry topic in order to display position and status in real-time.  The Tester app allows you to publish control commands to the vehicles and notifications to the Map app.

![Messaging][pic messaging]

## Getting started

### 1. Bluemix
If you do not already have an IBM Bluemix account, visit http://www.bluemix.net to sign up for an account.  The basic account is free for 30 days, and provides enough compute resources to run this tutorial.  Once the account request is submitted, you should be receiving a confirmation email within few minutes.

### 2. Internet of Things Foundation

- From your Bluemix dashboard, select **Add a Service**.

    ![pic iot 1]

- Choose **Internet of Things** from the service list.

    ![pic iot 2]

- Provide service a name that you will recognize later, e.g: **iot-Foundation**. This service creates a unique **IoT Organization** for you, which is a shared space under your control where your devices and approved applications can securely connect and share data.  Note, that the service name you enter here is just for your reference - it will not collide with other users possibly have created their services with the same name.

- Press **Create** to add the service.  After the service is loaded, you will see the service information page with instructions and documentation for registering devices.  

- Click the **Launch dashboard** button on the left of the page to access your IoT organization dashboard. If this is the first time, it may take a while to open.

    ![pic iot 4]

- From the dashboard, you will be able to manually register devices, generate API keys for applications, and invite others to your organization.

### 3. Configure (IoT Foundation)

This simulator application uses IoT Foundation for near real-time messaging between simulated vehicles (devices) and the Map and Tester apps (applications).  To facilitate this communication, you first have to register the devices and generate an API key for the applications.

#### 3.1 Register devices

The vehicle simulator allows you to model multiple vehicles from a single Node.js runtime instance.  Each vehicle simulator will be treated as a **device** by IoT Foundation.  You will eventually run 3 vehicle simulators, so the first step is to manually register these simulators to obtain access credentials.

> REST APIs for device registration with IoT Foundation are also [available](https://docs.internetofthings.ibmcloud.com)

* From the organization dashboard's "Devices" tab, press **Add Device** on the bottom of the page.

    ![pic iot 6_2]

* In the "Add Device" page, press **Create device type...** and enter **vehicle** (case sensitive) in the name field. Press **Next** at the bottom right corner of the page.

    ![pic iot 6_3]

* On the "Define template" and "Submit information" pages, do not select anything, just press Next.
* Finally, on the "Metadata (optional)" page press **Create**.

You have now added a new device type "vehicle", which is also selected and will next create a new device under that device type.

* Having the **vehicle** device type selected in the drop down list press **Next**

    ![pic iot 6_4]

* For the **Device ID**, enter a text of any length to describe this vehicle (e.g. CAR, BUS, ABC etc) and select **Next**.

    ![pic iot 6_5]

* On the "Metadata (optional)" and "Security" pages do not change anything, just press **Next**.
* If the last page "Summary" looks like this, press **Add**.

    ![pic iot 6_6]

* The next screen will show you credentials for your device. For example:

        Organization ID: o4ze1w
        Device Type: vehicle
        Device ID: BUS
        Authentication Method: token
        Authentication Token: xyz1rWHG9JhDw-rzyx

* **IMPORTANT:** Save this information in a file before closing as you will need these credentials later on and will not be able to see them afterwards. Close the page by pressing **X** when done.


* Now add another vehicle with Device Id **"BUS-1"** under the device type of **"vehicle"**. Note, that you can now reuse the device type "vehicle" you created earlier. As before, save the credentials for later use. For example:

        org=o4ze1w
        type=vehicle
        id=BUS-1
        auth-method=token
        auth-token=x7QmQ*!jQz6NUF&rAx

The reason why this is needed is because in the sample code the first Device Id you created will be used to transport telemetry data from all the vehicles at once in a single MQTT message. This second Device Id will be used to demonstrate *single vehicle telemetry data*. 


#### 3.2 Generate an API key

You next will generate an API key for your organization.  An API key provides credentials for any application (i.e. not a device) to connect to IoT Foundation using an MQTT client.

* From your organization IoT dashboard, click on the **ACCESS**, then **API Keys** and then press **Generate API Key** on the bottom.

* Again, copy down the key and token presented for later use. You may save them with other information you recorded earlier. **IMPORTANT:** Press **Finish** on the bottom right corner of the page before exiting to make sure the API key gets created.

        Key:        a-z4ze1w-b7xr3cozyx
        Auth Token: q4QD9Y@XyzrshlCD7q


* On the API Keys tab, you will now see a new row for the API key you just created. You may add a comment to the key to associate it with your Connected Vehicle application.

    ![pic iot 8_2]


### 4. Clone & Configure the Sample

* Clone this git repository to your local machine either by using "git clone" command line or downloading & extracting the [master.zip](https://github.com/jruponen/iot-connected-car-sample/archive/master.zip). If you do not yet have git command line set up, here's the [git set up instructions](https://help.github.com/articles/set-up-git). 

* Choose a globally unique application name to be used in Bluemix deployment (e.g. using your initials in the beginning of the name: **jriot-connected-car**) and enter this name for the **host** and **name** fields in  [manifest.yml](https://github.com/jruponen/iot-connected-car-sample/blob/master/manifest.yml). Also ensure the number of **instances** is **1** and **VEHICLE_COUNT** is **3**.

        applications:
        - host: iot-connected-car-sample
          disk: 128M
          name: iot-connected-car-sample
          command: node app.js
          path: .
          domain: mybluemix.net
          memory: 640M
          instances: 1
          env:
             VEHICLE_COUNT: 3


* All configuration for IoT Foundation is found in [config/settings.js](https://github.com/jruponen/iot-connected-car-sample/blob/master/config/settings.js).  This file stores all device tokens and API keys.  Edit the file according to your keys.

  * For **iot_deviceType**, enter **vehicle**
  * For **iot_deviceOrg**, enter your 6-character organization ID (e.g. **o4ze1w**)
  * For **iot_deviceSet**, enter the two Device Id's and their tokens you registered
  * For **iot_apiKey**, enter the API key you created
  * For **iot_apiToken**, enter the Authentication token for the API key
 

### 5. Deploy to Bluemix

To deploy the Connected Vehicle application to Bluemix, use the Cloud Foundry [command line tools](https://github.com/cloudfoundry/cli#downloads).

##### Push the application to Bluemix

* From the terminal, change to the root directory of this project and type:


        cf login


* Follow the prompts, entering **https://api.ng.bluemix.net** for the API endpoint, and your Bluemix login name (email) and password as login credentials.  If you have more than one Bluemix organization and space available, you will also have to choose these.  Next, enter the following to create your application, without starting it:


        cf push <APP_NAME> --no-start
    

* Create one more required service, "Geospatial Analysis" and then bind the two existing services to your application.

        cf create-service "Geospatial Analytics" Standard iot-Geospatial-Analytics
        cf bind-service <APP_NAME> iot-Geospatial-Analytics
        cf bind-service <APP_NAME> iot-Foundation


* Now start your Connected Car application

        cf start <APP_NAME>

* If the Geospatial Analytics service does not start automatically, you may start it via REST API: http://APP_NAME.mybluemix.net/GeospatialService_start
    
* If the application does not start successfully, view the error logs:

        cf logs <APP_NAME> --recent
    
    
* Check manifest.yml for errors, such as tabs in place of whitespace.


* Now, visit the Map app at your URL: **http://APP_NAME.mybluemix.net**.  You will see the simulated vehicles moving across the map.  Click on a vehicle to view the telemetry data.

    ![pic map 1]

##### Increase the vehicle count per simulator

Each vehicle simulator can model multiple vehicles.  The number of vehicles is controlled by a Bluemix environment variable, which can be set from the Bluemix application dashboard.

* In the dashboard, click on the **SDK for Node.js** icon

    ![pic vehicle count 1]

* Go to **Environment Variables** and change the value of USER-DEFINED variable **VEHICLE_COUNT** to **5**.

    ![pic vehicle count 2]

* Notice that the change was applied dynamically and the number of cars on the map increases.


### 6. Use the Tester app

The Tester app is used to send commands to the simulated vehicles and the Map app.  The vehicle simulator subscribes to commands of type **setProperty** and will dynamically change its own properties, speed, and state.  The Map app subscribes to commands of type **addOverlay** and will dynamically display a popup of text over a vehicle.

* Open the Map app and Tester app (http://APP_NAME.mybluemix.net/tester) side-by-side, so that you can see both windows.

##### 7.1  **setProperty** API

* Select a vehicle, then enter the ID in the second form on the Tester page.
* Enter a property of **speed** and a value of 120, and press **Update Property**.  An MQTT message will be published from the Tester app (topic and payload on screen), and the vehicle you chose will receive the message and change speed to 120 km/hr.

    ![pic set property 1]

The vehicles simulate a set of **static properties** (location, speed, state, and type) and **custom properties**.  The **setProperty** API allows you to dynamically add/change/delete a custom property on a vehicle.

* To add a property, simply publish a property that doesn't yet exist.  For example, use property **driverWindow** and value **UP**:

    ![pic set property 2]

* To delete a property, update the property with an empty value; the vehicle will cease including the property in its telemetry message.

##### 7.2  **addOverlay** API

The Map app subscribes to commands of type **addOverlay**, to allow external applications to display messages on the map over a vehicle.

* In the Tester app, use the upper form to display a message over a vehicle, for example:

    ![pic add overlay 1]

## Next Steps - Node-RED

The application can be extended with [Node-RED](http://nodered.org/) to easilly create flows to deal with data streams to/from devices. Bluemix contains two sample boilerplates (templates) for Node-RED, [Internet of Things Foundation Starter](https://console.ng.bluemix.net/catalog/starters/internet-of-things-foundation-starter/) and [Node-RED Starter](https://console.ng.bluemix.net/catalog/starters/node-red-starter/). You can start with either, but [Internet of Things Foundation Starter](https://console.ng.bluemix.net/catalog/starters/internet-of-things-foundation-starter/) already contains a sample flow for temperature/humidity simulated sensor.  Read section 9 in this [tutorial](http://m2m.demos.ibm.com/dl/iot-connected-vehicle-tutorial.pdf) to understand how to use it with connected vehicle simulator.

[MQTT]:http://mqtt.org
[Iot Foundation]:http://internetofthings.ibmcloud.com
[pic messaging]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/messaging.png
[pic iot 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/iot1.png
[pic iot 2]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/iot2.png
[pic iot 3]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/iot3.png
[pic iot 4]:https://raw.githubusercontent.com/jruponen/iot-connected-car-sample/master/docs/iot4_2.png
[pic iot 5]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/iot5.png
[pic iot 6]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/iot6.png
[pic iot 6_2]:https://raw.githubusercontent.com/jruponen/iot-connected-car-sample/master/docs/iot6_2.png
[pic iot 6_3]:https://raw.githubusercontent.com/jruponen/iot-connected-car-sample/master/docs/iot6_3.png
[pic iot 6_4]:https://raw.githubusercontent.com/jruponen/iot-connected-car-sample/master/docs/iot6_4.png
[pic iot 6_5]:https://raw.githubusercontent.com/jruponen/iot-connected-car-sample/master/docs/iot6_5.png
[pic iot 6_6]:https://raw.githubusercontent.com/jruponen/iot-connected-car-sample/master/docs/iot6_6.png
[pic iot 7]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/iot7.png
[pic iot 8]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/iot8.png
[pic iot 8_2]:https://raw.githubusercontent.com/jruponen/iot-connected-car-sample/master/docs/iot8_2.png
[pic config 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/config1.png
[pic map 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/map1.png
[pic vehicle count 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/vehicle_count1.png
[pic vehicle count 2]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/vehicle_count2.png
[pic set property 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/set_property1.png
[pic add overlay 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/add_overlay1.png
[pic set property 2]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/set_property2.png
[pic geospatial 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/geospatial1.png
[pic geospatial 2]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/geospatial2.png
[pic geospatial 3]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/geospatial3.png
[pic geospatial 4]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/geospatial4.png
[pic node red 0]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red0.png
[pic node red 1 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red1_1.png
[pic node red 1 2]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red1_2.png
[pic node red 1 3]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red1_3.png
[pic node red 2 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red2_1.png
[pic node red 2 2]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red2_2.png
[pic node red 2 3]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red2_3.png
[pic node red 2 4]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red2_4.png
[pic node red 3 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red3_1.png
[pic node red 3 2]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red3_2.png
[pic node red 3 1]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red4_1.png
[pic node red 4 2]:https://raw.githubusercontent.com/ibm-messaging/iot-connected-vehicle-starter-kit/master/docs/node_red4_2.png
