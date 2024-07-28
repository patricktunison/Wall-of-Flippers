
# Wall of Flippers, Logstash and Azure Log Analytics / Sentinel

The purpose for this fork is to incorporate Wall of Flippers into an environment in which SIEM alerts can be configured. While rudimentary, this fork can eventually be expanded to detect Bluetooth Low-Energy threats apart from just the Flipper Zero. This fork utilizes Wall of Flippers capability of writing a JSON log file, but expands it to create rolling JSON logs which are captured every five seconds. 

Theoretically this could be deployed on a Raspberry Pi to be used as a persistent BLE threat monitor. An example use case could be a small business with critical assets utilizing BLE. 

The workflow utilizes Logstash to actively observe the log folder, and report the logs to Azure Log Analytics. I have included the Logstash configuration file in the repository.


## Deployment

There are several important steps to deploying this project. I have found it useful to have a standalone USB Bluetooth adapter, and essential if you are running the project in a VM. The Bluetooth adapter I'm using is listed here:
https://www.amazon.com/dp/B0C1ZDYFSZ

### Wall of Flippers Installation

First and foremost, you will need to install Wall of Flippers. [I have left the original README available in this repository.](https://github.com/patricktunison/Wall-of-Flippers/blob/main/WallOfFlippers_README.md)

If you are deploying on a VM running Kali Linux and unable to get the Bluetooth dependencies installed correctly, you may need to manually install libbluetooth-dev:

```bash
sudo apt-get install libbluetooth-dev 
```
Create a folder within Wall-Of-Flippers called "logs"

If Wall Of Flippers is failing to recognize the Bluetooth service running, you may need to restart the Bluetooth service:

```bash
sudo service bluetooth restart
```

Once this fork of Wall of Flippers is installed, we will need to install and configure Logstash. 

### Logstash Configuration

For this project, I am using [Logstash 8.8.0.](https://www.elastic.co/downloads/past-releases/logstash-8-8-0) Microsoft states in [their documentation](https://learn.microsoft.com/en-us/azure/sentinel/connect-logstash):

```"Microsoft Sentinel's Logstash output plugin supports only Logstash versions 7.0 to 7.17.10, and versions 8.0 to 8.9 and 8.11"```

Download and unpack the file, and cd into the directory. We next need to install the Log Analytics plugin for Logstash:

```bash
sudo bin/logstash-plugin install microsoft-logstash-output-azure-loganalytics
```
Place the included ```wof.conf``` file in the config folder within logstash-8.8.0. Edit this config file to include the path to the Wall of Flippers "logs" folder, and your Log Analytics Agent information. Please note that these credentials are stored in plaintext, so it is advisable to utilize a more secure solution for your Azure credentials. 

Lastly, Microsoft also recommends disabling ECS in the pipeline. To do so, open the ```logstash.yml``` file in the config folder within logstash. Put the following at the bottom of the file:

```pipeline.ecs_compatibility: disabled```

Now that Wall of Flippers and Logstash are configured, we should be able to launch them and start collecting logs. 

### Running Wall of Flippers and Logstash

Launch Wall of Flippers first, and make sure everything is working correctly. 

Launching Logstash should be simple as well. Open a new terminal, cd into the Logstash 8.8.0 folder, and run (replacing "path/to" with your path):

```bash
sudo bin/logstash -f /path/to/logstash-8.8.0/config/wof.conf
```

Once both are operating correctly, Wall Of Flippers will begin to create JSON log files if a Flipper Zero is detected. It will not create log files if a Flipper is not detected. If everything is configured correctly, you should be able to see a new table created within Log Analytics once a Flipper is detected, which will allow you to query it and create alerts within Sentinel. 

## Current Features

Here are the current fields the logs are generating:
- RSSI
- UUID
- Sighting Count
- MAC
- Unix Last Seen 
- Unix First Seen
- Flipper Type
- Last Counted Sigting (in Unix)
- Flipper Name
- Timestamp

The sighting count will signify the amount of times a Flipper has been seen in the past. The count increases if a Flipper has been seen again after 24 hours. 

## Future Features

- Easier deployment
- Accurate BLE Spam recognition
- Raspberry Pi deployment documentation
- Expand devices and patterns recognized
- LLM integration?