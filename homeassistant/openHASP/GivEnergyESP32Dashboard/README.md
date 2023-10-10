# GivEnergy Dashboard for WT32-SC01
This directory contains the openHASP jsonl file and the yaml file for Home Assistant to implement the GivEnergy dashboard on a WT32-SC01 ESP32 device with inbuilt display.

## Pre-requisites:

* [A WT32-SC01 or WT32-SC01-Plus device](https://aliexpress.com/item/1005005561671980.html?gatewayAdapt=glo2deu)
* A [GivEnergy](https://givenergy.co.uk/) battery storage system
* [Home Assistant](https://www.home-assistant.io/)
* [HACS](https://hacs.xyz/)
* [GivTCP](https://github.com/britkat1980/giv_tcp)
* [openHASP](https://github.com/HASwitchPlate/openHASP-custom-component)
* [Octopus Energy plugin](https://github.com/BottlecapDave/HomeAssistant-OctopusEnergy)

There are enough guides on the internet for me to not document how to set up any of the above.

## Steps to Implement:

* Copy/paste the [jsonl](./pages.jsonl) file to the openHASP web interface for your target device
* Copy/paste the [yaml](./openhasp.yaml) file to the relevant place in Home Assistant - I have a seperate openhasp.yaml file which is included in my configuration.yaml file - YMMV
* Replace all instances of YOURSERIAL with the correct GivTCP entities for your device
* Restart Home Assistant

![OpenHASP_GivEnergy_1_Discharging_No_Solar](https://github.com/DJBenson/ha-stuff/assets/1013909/cd845a1b-7101-4d77-8f70-3b63126aea1a)
![OpenHASP_GivEnergy_Original](https://github.com/DJBenson/ha-stuff/assets/1013909/ed158efe-7e95-430c-bf99-02ee69258e7c)
![OpenHASP_GivEnergy_2_Charging_House_Load_No_Solar](https://github.com/DJBenson/ha-stuff/assets/1013909/a3d6d59a-0cb3-475c-ace9-f04da80a4ca0)
![OpenHASP_GivEnergy_2_Charging_No_Solar](https://github.com/DJBenson/ha-stuff/assets/1013909/0d8a514b-dd32-44c8-b4f7-566147593df8)
