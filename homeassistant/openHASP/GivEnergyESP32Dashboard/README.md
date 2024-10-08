# GivEnergy Dashboard for WT32-SC01
This directory contains the openHASP jsonl file and the yaml file for Home Assistant to implement the GivEnergy dashboard on a WT32-SC01 ESP32 device with inbuilt display.

## Pre-requisites:

* [A WT32-SC01 or WT32-SC01-Plus device](https://aliexpress.com/item/1005005561671980.html?gatewayAdapt=glo2deu)
* A [GivEnergy](https://givenergy.co.uk/) battery storage system (or other system which provides the requisite entities)
* A [GivEnergy](https://givenergy.co.uk/) EV chartger (or other charger which provides the requisite entities)
* An EV which provides data such as state of charge (I have a Kia EV6 so use [github.com/Hyundai-Kia-Connect/kia_uvo](https://github.com/Hyundai-Kia-Connect/kia_uvo)
* [Home Assistant](https://www.home-assistant.io/)
* [HACS](https://hacs.xyz/)
* [GivTCP](https://github.com/britkat1980/giv_tcp)
* [openHASP](https://github.com/HASwitchPlate/openHASP-custom-component)
* [Octopus Energy plugin](https://github.com/BottlecapDave/HomeAssistant-OctopusEnergy)

There are enough guides on the internet for me to not document how to set up any of the above.

## Steps to Implement:

* Upload the images from this repository to the openHASP web interface for your target device - for other manufacturers just replace the image with your preferred logo with dimensions 120px x 26px.
* Copy/paste the [jsonl](./pages.jsonl) file to the openHASP web interface for your target device
* Copy/paste the [yaml](./openhasp.yaml) file to the relevant place in Home Assistant - I have a seperate openhasp.yaml file which is included in my configuration.yaml file - YMMV
* Replace all placeholders with the correct entities for your setup (there are placeholders for the outdoor temperature sensor, your WT32-SC01 device name, your GivEnergy serial, your Octopus sensors plus sensors for the EV charger and EV state of charge).
  * \<YourDevice>: The device name of the WT32-SC01
  * \<YourOutsideTemperatureSensor>: Entity providing outside temperature - can be a local sensor or from a weather entity
  * \<YourHomeBatterySoCSensor>: Entity providing the state of charge (in %) of your home battery setup
  * \<YourPVPowerSensor>: Entity providing the current output of your PV setup (in watts)
  * \<YourImportPowerSensor>: Entity providing the grid import power (in watts)
  * \<YourExportPowerSensor>: Entity providing the grid export power (in watts)
  * \<YourHomeBatteryPowerSensor>: Entity providing the current power from/to the home battery (in watts)
  * \<YourGridPowerSensor>: Entity providing the grid power (both import/export) (in watts)
  * \<YourCurrentElectricityUnitRateSensor>: Entity providing the current unit rate for electricity consumemed
  * \<YourLoadPowerSensor>: Entity providing the current house load demand (in watts)
  * \<YourEVSoCSensor>: Entity providing the current SoC of your EV battery
  * \<YourEVChargerPowerSensor>: Entity providing the current power of your EV charger (in kW)
* Restart Home Assistant

## Customising:

* This setup assumes you have a GivEnergy setup, a GivEnergy EV Charger and a car that provides sensors for the charging power and state of charge. If you don't have substitutes for these sensors, you need to remove the lines from the openhasp.yaml file which relate to those entities as well as removing them from pages.jsonl. For example if you wish to remove the EV battery state of charge and charging power, you can remove line 38-45 in pages.jasonl and lines 912 to 931 in openhasp.yaml.

### Day Time
![image](https://github.com/user-attachments/assets/2ce00172-75f7-4fc3-b4ff-40f9a2ac79ce)
![image](https://github.com/user-attachments/assets/1b5aa8c0-bf95-4c75-a616-52da42555605)
![image](https://github.com/user-attachments/assets/fb12a0b2-8dec-4203-8837-5d67353ad497)
![image](https://github.com/user-attachments/assets/7aabef5b-70ca-4721-9f9b-b3cd286cf830)

### Night Time
![image](https://github.com/user-attachments/assets/be1c84ca-b576-4463-8232-4f5e5e415522)
![image](https://github.com/user-attachments/assets/651423d1-d590-4940-9893-695c4777bf43)
![image](https://github.com/user-attachments/assets/818c8282-ea7f-428f-865d-ca82311b79ab)
![image](https://github.com/user-attachments/assets/6179f44c-1ceb-4a7a-982a-ea9e0ae0bd03)
![image](https://github.com/user-attachments/assets/e2a01264-0f93-45a7-9475-affcc1d7ac84)


## 3D Printed Case:

The WT32-SC01-Plus Stand with socket.stl file provides a handy way to display the device with cable routing underneath. I got mine printed by [JLCPCB](https://jlcpcb.com/) and they cost about $7 each printed in white resin.
