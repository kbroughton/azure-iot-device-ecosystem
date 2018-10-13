---
platform: raspbian
services: iot-hub, stream-analytics, event-hubs, web-apps, documentdb, storage-accounts
language: c
---

# Connect Raspberry Pi to Azure IoT Hub in the cloud

In this tutorial, you begin by learning the basics of working with Raspberry Pi that's running Raspbian. You then learn how to seamlessly connect your devices to the cloud by using [Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-what-is-iot-hub). For Windows 10 IoT Core samples, go to the [Windows Dev Center](http://www.windowsondevices.com/).

## What you do

* Setup Raspberry Pi.
* Create an IoT hub.
* Register a device for Pi in your IoT hub.
* Run a sample application on Pi to send sensor data to your IoT hub.

Connect Raspberry Pi to an IoT hub that you create. Then you run a sample application on Pi to collect temperature and humidity data from a BME280 sensor. Finally, you send the sensor data to your IoT hub.

## What you learn

* How to create an Azure IoT hub and get your new device connection string.
* How to connect Pi with a BME280 sensor.
* How to collect sensor data by running a sample application on Pi.
* How to send sensor data to your IoT hub.

## What you need

![What you need](media/iot-hub-raspberry-pi-kit-c-get-started/0_starter_kit.jpg)

* The Raspberry Pi 2 or Raspberry Pi 3 board.
* An active Azure subscription. If you don't have an Azure account, [create a free Azure trial account](https://azure.microsoft.com/free/) in just a few minutes.
* A monitor, a USB keyboard, and mouse that connect to Pi.
* A Mac or a PC that is running Windows or Linux.
* An Internet connection.
* A 16 GB or above microSD card.
* A 5-volt 2-amp power supply with the 6-foot micro USB cable.

The following items are optional:

* An assembled Adafruit BME280 temperature, pressure, and humidity sensor.
* A breadboard.
* 6 F/M jumper wires.
* A diffused 10-mm LED.


> **NOTE:** 
These items are optional because the code sample support simulated sensor data.

## Create an IoT hub

1. In the [Azure portal](https://portal.azure.com/), click **New** > **Internet of Things** > **IoT Hub**.

   ![Create an iot hub in the Azure portal](https://github.com/Microsoft/azure-docs/raw/master/articles/iot-hub/media/iot-hub-create-hub-and-device/1_create-azure-iot-hub-portal.png)
1. In the **IoT hub** pane, enter the following information for your IoT hub:

   **Name**: It is the name for your IoT hub. If the name you enter is valid, a green check mark appears.

   **Pricing and scale tier**: Select the free F1 tier. This option is sufficient for this demo. See [pricing and scale tier](https://azure.microsoft.com/pricing/details/iot-hub/).

   **Resource group**: Create a resource group to host the IoT hub or use an existing one. See [Using resource groups to manage your Azure resources](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-portal).

   **Location**: Select the closest location to you where the IoT hub is created.

   **Pin the dashboard**: Check this option for easy access to your IoT hub from the dashboard.

   ![Fill in the fields for creating your Azure IoT hub](https://github.com/Microsoft/azure-docs/raw/master/articles/iot-hub/media/iot-hub-create-hub-and-device/2_fill-in-fields-for-azure-iot-hub-portal.png)

1. Click **Create**. It could take a few minutes for your IoT hub to be created. You can see progress in the **Notifications** pane.

   ![See notifications of your IoT hub creation progress](https://github.com/Microsoft/azure-docs/raw/master/articles/iot-hub/media/iot-hub-create-hub-and-device/3_notification-azure-iot-hub-creation-progress-portal.png)

1. Once your IoT hub is created, click it from the dashboard. Make a note of the **Hostname**, and then click **Shared access policies**.

   ![Get the hostname of your IoT hub](https://github.com/Microsoft/azure-docs/raw/master/articles/iot-hub/media/iot-hub-create-hub-and-device/4_get-azure-iot-hub-hostname-portal.png)

1. In the **Shared access policies** pane, click the **iothubowner** policy, and then copy and make a note of the **Connection string** of your IoT hub. For more information, see [Control access to IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-security).

   ![Get your IoT hub connection string](https://github.com/Microsoft/azure-docs/raw/master/articles/iot-hub/media/iot-hub-create-hub-and-device/5_get-azure-iot-hub-connection-string-portal.png)

## Register a device in the IoT Edge hub for the your device

1. In the [Azure portal](https://portal.azure.com/), open your IoT hub.
1. On the left panel under "Explorers" click **IoT Devices**.
1. In the Device Explorer pane, click **Add** to add a device to your IoT hub.

   **Device ID**: The ID of the new device.

   **Authentication Type**: Select **Symmetric Key**.

   **Auto Generate Keys**: Check this field.

   **Connect device to IoT Hub**: Click **Enable**.

   ![Add a device in the "Automatic Device Management" of your IoT hub](https://github.com/kbroughton/azure-iot-device-ecosystem/raw/master/get_started/mbk_edge_not_devices.png)

1. Click **Save**.
1. After the device is created, open the device in the **Device Explorer** pane.
1. Make a note of the primary key of the connection string.

   ![Get the device connection string](https://github.com/Microsoft/azure-docs/raw/master/articles/iot-hub/media/iot-hub-create-hub-and-device/7_get-device-connection-string-in-device-explorer-portal.png)


## Setup Raspberry Pi

### Enable SSH and SPI

1. Connect Pi to the monitor, keyboard and mouse, start Pi and then log in Raspbian by using `pi` as the user name and `raspberry` as the password.
2. Click the Raspberry icon > **Preferences** > **Raspberry Pi Configuration**.

   ![The Raspbian Preferences menu](media/iot-hub-raspberry-pi-kit-c-get-started/1_raspbian-preferences-menu.png)

3. On the **Interfaces** tab, set **SPI** and **SSH** to **Enable**, and then click **OK**.

   ![Enable SPI and SSH on Raspberry Pi](media/iot-hub-raspberry-pi-kit-c-get-started/2_enable-spi-ssh-on-raspberry-pi.png)

> **NOTE:** 
To enable SSH and SPI, you can find more reference documents on [raspberrypi.org](https://www.raspberrypi.org/documentation/remote-access/ssh/) and [RASPI-CONFIG](https://www.raspberrypi.org/documentation/configuration/raspi-config.md).

### Connect the sensor to Pi

Use the breadboard and jumper wires to connect an LED and a BME280 to Pi as follows. If you don’t have the sensor, skip this section.

![The Raspberry Pi and sensor connection](media/iot-hub-raspberry-pi-kit-c-get-started/3_raspberry-pi-sensor-connection.png)

For sensor pins, use the following wiring:

| Start (Sensor & LED)     | End (Board)            | Cable Color   |
| -----------------------  | ---------------------- | ------------: |
| LED VDD (Pin 5G)         | GPIO 4 (Pin 7)         | White cable   |
| LED GND (Pin 6G)         | GND (Pin 6)            | Black cable   |
| VDD (Pin 18F)            | 3.3V PWR (Pin 17)      | White cable   |
| GND (Pin 20F)            | GND (Pin 20)           | Black cable   |
| SCK (Pin 21F)            | SPI0 SCLK (Pin 23)     | Orange cable  |
| SDO (Pin 22F)            | SPI0 MISO (Pin 21)     | Yellow cable  |
| SDI (Pin 23F)            | SPI0 MOSI (Pin 19)     | Green cable   |
| CS (Pin 24F)             | SPI0 CS (Pin 24)       | Blue cable    |

Click to view [Raspberry Pi 2 & 3 Pin mappings](https://developer.microsoft.com/windows/iot/docs/pinmappingsrpi) for your reference.

After you've successfully connected BME280 to your Raspberry Pi, it should be like below image.

![Connected Pi and BME280](media/iot-hub-raspberry-pi-kit-c-get-started/4_connected-pi.jpg)

Turn on Pi by using the micro USB cable and the power supply. Use the Ethernet cable to connect Pi to your wired network or follow the [instructions from the Raspberry Pi Foundation](https://www.raspberrypi.org/learning/software-guide/wifi/) to connect Pi to your wireless network.

![Connected to wired network](media/iot-hub-raspberry-pi-kit-c-get-started/5_power-on-pi.jpg)


