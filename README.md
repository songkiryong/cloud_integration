# 목표  
iPaaS와 IoT를 활용해 산업 화재 대응 솔루션을 제작합니다.  

# 아키텍처  

![image](https://user-images.githubusercontent.com/73922068/134149465-ca480df2-ab1e-4570-bb30-3f4ea319496c.png)

-> 산업 현장에서 화재가 발생했을 때, 사람들에게 대피 문자를 발송하고, 긴급 화상회의를 생성하고, 현장에 있는 센서 데이터를 자동 저장하는 등 일련의 화재 대응 솔루션을 자동화합니다


# 준비사항  
### Tinkerforge ( 센서 장치 )  
- 주소 : https://www.tinkerforge.com/en/  
- 센서  
  - Master Brick : 여러 센서 장치를 통합  
  - CO2 2.0 :	Measures CO2 concentration, temperature and humidity
  - Motion Detector 2.0	: Passive infrared (PIR) motion sensor with 12m range and dimmable backlight
  - Sound Pressure Level :	Measures Sound Pressure Level in dB(A/B/C/D/Z)
  - Piezo Speaker :	Creates beep with configurable frequency

### Raspberry Pi 4    
 - MQTT 라이브러리 : MQTT 프로토콜을 이용해 센서 데이터를 읽고 cumulocity 로 보내는 역할
 ```
 pip install paho-mqtt
 ```
 - MQTT 프로그램  
 ```
 import time
import sys

# mqtt
import paho.mqtt.client as mqttClient

# tinkerforge
from tinkerforge.ip_connection import IPConnection
from tinkerforge.bricklet_co2_v2 import BrickletCO2V2
from tinkerforge.bricklet_sound_pressure_level import BrickletSoundPressureLevel
from tinkerforge.bricklet_gps_v2 import BrickletGPSV2

#On Connect Call back function
def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print("Connected to broker")
    else:
        print("Connection failed")


### MQTT Connect
client = mqttClient.Client("clientId")  # create new instance
client.on_connect=on_connect #binding on_connect function
client.username_pw_set("song2109/kiryong48@gmail.com", "qwer1234!")  # set username and password
client.connect("mqtt.song2109.us.cumulocity.com", 1883)  # connect to broker

### TinkerForge Connect
ipcon = IPConnection() # Create IP connection
co2 = BrickletCO2V2("VZr", ipcon) # Create device object
spl = BrickletSoundPressureLevel("UVk", ipcon)
gps = BrickletGPSV2("PvA", ipcon)

ipcon.connect("localhost", 4223) # Connect to brickd
# Don't use device before ipcon is connected

while True:
    # Get current all values
    co2_concentration, temperature, humidity = co2.get_all_values()
    decibel = spl.get_decibel()
    latitude, ns, longitude, ew = gps.get_coordinates()
    print(gps.get_coordinates())

    tMsg = "200,c8y_TemperatureMeasurement,C," + str(temperature/100.0)
    co2Msg = "200,c8y_CO2Measurement,ppm," + str(co2_concentration)
    hMsg = "200,c8y_HumidityMeasurement,%RH," + str(humidity/100.0)
    dMsg = "200,c8y_SoundPreasureMeasurement,dB(A)," + str(decibel/10.0)
    gps_latMsg = "200,c8y_GPSMeasurement,'," + str(latitude/1000000.0)
    gps_nsMsg = "200,c8y_GPSMeasurement," + str(ns)
    gps_longMsg = "200,c8y_GPSMeasurement,'," + str(longitude/1000000.0)
    gps_ewMsg = "200,c8y_GPSMeasurement," + str(ew)





    try:
        print("CO2 Concentration: " + str(co2_concentration) + " ppm")
        print("Temperature: " + str(temperature/100.0) + " C")
        print("Humidity: " + str(humidity/100.0) + " %RH")
        print("Decibel: " + str(decibel/10.0) + " dB(A)")
        print("Latitude: " + str(latitude/1000000.0) + " '")
        print("N/S: " + ns)
        print("Longitude: " + str(longitude/1000000.0) + " '")
        print("E/W: " + ew)
        #Temperature, co2, decibel, humidity
        client.publish("s/us", tMsg ,qos=0) #Publish templates
        client.publish("s/us", co2Msg ,qos=0) #Publish templates
        client.publish("s/us", hMsg ,qos=0) #Publish templates
        client.publish("s/us", dMsg, qos=0)
        #GPS
        client.publish("s/us", gps_latMsg, qos=0)
        client.publish("s/us", gps_nsMsg, qos=0)
        client.publish("s/us", gps_longMsg, qos=0)
        client.publish("s/us", gps_ewMsg, qos=0)
    except KeyboardInterrupt:
        client.disconnect() #Disconnect Client

    print("--------------------------")
    time.sleep(5)
   
```

# 작동순서  

1. clone  
``` 
https://github.com/songkiryong/2-tier-wordpress_Terraform.git 
```

2. Terraform  
``` 
terraform init  
terraform apply 
```

3. Ansible    
``` 
ansible-playbook deploy.yaml -b --private-key "~/.ssh/id_rsa" 
```
# 삭제  
1. Terraform  
```
terraform destroy
```
2. Ansible
```
ansible-playbook remove.yaml -b --private-key "~/.ssh/id_rsa"
```

