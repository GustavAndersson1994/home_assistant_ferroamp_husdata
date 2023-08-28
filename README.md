# How to Integrate Ferroamp and Husdata with Home Assistant
If you're looking to integrate your Ferroamp system and Husdata into your Home Assistant setup, this small guide will walk you through the configuration process step by step. 

I thought the installing process was a bit confusing the first time so I hope this guide can help someone else out there!


## Prerequisite
- Home Assistant installed on a Raspberry Pi 4. Follow this guide: https://www.home-assistant.io/installation/raspberrypi/

## Step 1: Install Ferroamp Integration
The communication is handle with the MQTT protocol. Therefore, we first need a MQTT broker.

### 1.1 Install Mosquitto MQTT Broker
Open the Home Assistant dashboard and navigate to the Supervisor section.
Click on "Add-on Store" and search for "Mosquitto Broker."
Install the "Mosquitto broker" add-on and wait for the installation to complete.


### 1.2 Configure Mosquitto MQTT Broker
In Mosquitto broker, go to settings, press the three dots and press "Edit in YAML". 

Toggle the `customize` attribute to `true``

```
logins: []
require_certificate: false
certfile: fullchain.pem
keyfile: privkey.pem
customize:
  active: true
  folder: mosquitto
```
and save. You need to restart the broker to enable the new settings.

When the customize is set to true, the mosquitto broker will load the configuration file in /share/mosquitto/mosquitto.conf instead of the default template which makes it easier for us to add several bridges in the same configuration.


### 1.3. Install the terminal
We will add the mosquitto configuration file through the terminal. Be aware of that this might require some minor knowledge about how to use a linux terminal. Once again, go to the ad-on Store, and install the "Terminal & SSH" extension. 

### 1.4 Add MQTT configuration for Ferroamp
Open the terminal. We will first change directory (cd), create a new folder (mkdir) and last create the config file (touch):

```
cd share
mkdir mosquitto
touch mosquitto.conf
```

Edit the file /share/mosquitto/mosquitto.conf. This can be done with your favorite editor such as nano or vim:

```
nano mosquitto.conf
```

Add following configuration

```
# Bridge to EnergyHub MQTT Broker
connection bridge-01
address 192.168.1.245:1883
try_private false
start_type automatic
bridge_insecure false
cleansession true
topic # in 0
username extapi
password ferroampExtApi
```

Change the address, username and password if your system is configurated differently.

Save the file with ctrl + x. Press Enter and close the file with ctrl + x again. 

The configuration for the MQTT broker from the Ferroamp system is now done. You can open the logs to make sure that everything was successful in the startup. 

Now, we need to install the Ferroamp extension to listen to the signals from the broker. 


## 1.5 Install Ferroamp extension
Now, we will add the Ferroamp extension which will catch up the messages from the broker. 

The easiest way to install the Ferroamp extension is with HACS:

- Install HACS by following https://hacs.xyz/docs/setup/prerequisites/.
- Open HACS and install the `Ferroamp MQTT Sensor` ad-on 

Open the Ferroamp extension. You should now see the sensors in the dashboard.



## Step 2: Husdata Integration

In this section, we will guide you through the process of integrating Husdata with Home Assistant and configuring the necessary settings.

### 2.1 Configure a second Bridge to Husdata MQTT Broker
Now, we will add a second bridge to the MQTT broker. Open the mosquitto.conf file in the same way as in section 1.4.

Add another bridge connection configuration to integrate Husdata:

```
# Bridge Husdata
connection bridge-02
topic # in 0 /HP/
address 192.168.1.235
username mqtt
password <my-password>
```

The full configuration should now look something like:


```
# Bridge to EnergyHub MQTT Broker
connection bridge-01
address 192.168.1.245:1883
try_private false
start_type automatic
bridge_insecure false
cleansession true
topic # in 0
username extapi
password ferroampExtApi
# Bridge Husdata
connection bridge-02
topic # in 0 /HP/
address 192.168.1.235
username mqtt
password 1234
```

Make sure that you are using the username and password that you can find in your Husdata portal. 

Restart the broker and once again make sure that the logs looks alright.

### 2.2 Add a new user in Home Assistant
Add a new user to your Home Assistant instance. For example, create a user named `mqtt` with the password `1234`. We will later use this user to configurate the MQTT.

### 2.3 Add MQTT
In integration, add a new integration `MQTT`. Select configure, Add the broker address, port, username and password. Something like:

```
192.168.1.145
1883
mqtt
1234
```

Save, the sensors should automatically be found! 

