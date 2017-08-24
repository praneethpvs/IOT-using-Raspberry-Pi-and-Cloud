# IOT-using-Raspberry-Pi-and-Cloud

Python app to demonstrate Raspberry Pi control.  

This is a sample application shows how to use simple inputs and outputs on a
Raspberry Pi.  There is a server side component that runs in Bluemix (Cloud Foundry) and
a Python client that runs on a Raspberry Pi.

The input is a simple button.  The output is a simple LED.  If you click on/off
in the web UI in Bluemix it will turn a LED on/off.  If you press the button
it will send a text message using Twilio to the phone number and message you
type in the web ui.

### To Use (Client Side - Raspberry Pi)
1. First login to your raspberry Pi OS. Run the following commands to update the Raspberry Pi.It will ask you to confirm with the “Y” key
```
sudo apt-get update
sudo apt-get upgrade
```

2. Next we need to install a helper library for using GPIO
```
git clone git://git.drogon.net/wiringPi
cd wiringPi
./build
```

3. Next we need to install the IoT library on the Raspberry Pi
```
curl -LO https://github.com/ibm-messaging/iot-raspberrypi/releases/download/1.0.2/iot_1.0-1_armhf.deb
sudo dpkg -i iot_1.0-1_armhf.deb
```
4. We need to get the device ID of our raspberry Pi do this run the following and save the output
```
service iot getdeviceid
#example output
The device id is _yourDeviceId_
```
Copy this device id qand save it.

5. Go to the IOT section in you IBM Bluemix account and Add a new device in the IOT section.
* Click “Add Device”.
* For the second field (it says e.g. mydevice type) type in exactly raspberrypi (you need to have it spelled like this or there will be issues). For device ID paste in the device ID we got from previous step.
* On the next page it will show something like below, copy this and switch back to terminal on your Raspberry Pi.
> org=pwftki  
> type=raspberrypi  
> id=000000000000  
> auth-method=token  
> auth-token=cXQaGx8o!a9HwxM-ka  

6. Next we need to download the Python code to run on the Raspberry Pi, and install the package manager for python:
```
cd ~
git clone https://github.com/IBM-Bluemix/python-iot-raspberry-pi.git
cd python-iot-raspberry-pi
sudo apt-get install python-pip
sudo pip install -r requirements.txt
```

The client requires a file ```/home/pi/device.cfg``` to exist. so create one file.
The file should look like the following
```
[application]
org=youriotorg
id=yourdeviceid
type=apikey
auth-key=yourapikey
auth-token=yourapitoken
```
The org, auth-key, auth-token can be generated at (sign into bluemix.net first) https://internetofthings.ibmcloud.com/dashboard.  Click on API keys to generate one.

### To Use Server Side.
1.  Install the Cloud Foundry CLI to deploy our app to Bluemix. To do this head over to https://github.com/cloudfoundry/cli/releases. Choose the appropriate installer for your platform, download it and run the installer.
2. We need to download the Python code for our app.
```
git clone git clone https://github.com/IBM-Bluemix/python-iot-raspberry-pi.git
cd python-iot-raspberry-pi
```
3. Get you API Credentials from Twilio. There is two pieces of information here, the Account SID and Auth Token. We will need to copy these back to the Bluemix tab we have open with Twilio. Paste your Account SID in the field in Bluemix that says Account SID, paste your Auth Token in the Auth Token field.
4. We need to login to Bluemix, to do that, type cf login -a https://api.ng.bluemix.net in the CLI.
5. Now create a service for your application and setup your phone number.

```
cf create-service iotf-service iotf-service-free iot-python
#replace the phone number with a phone number you registered with twilio
cf set-env python-iot-hackathon PHONE_NUMBER_FROM "+19192006462"
#replace the device ID with the device id of your raspberry pi
#you can get it by running service iot getdeviceid on your raspberry pi
cf set-env python-iot-hackathon DEVICE_ID "b827eb36267d"
```
6. To deploy our application all we need to do now is type ***cf push myappname*** where myapp name is a unique name you choose for your app.
7. It will take about a minute or two to deploy your application but eventually you will get some output that looks like the following:
> requested state: started  
> instances: 1/1  
> usage: 1G x 1 instances  
> urls: testapp-jbs.mybluemix.net  
> last uploaded: Fri Jul 31 00:25:17 UTC 2015  
> stack: lucid64  
> buildpack: SDK for Node.js(TM) (ibm-node.js-0.12.7)  

There is a row that says urls:, copy that URL and paste it into your browser.  
If everything went well you should have a page that looks like the following:  
![webapp](https://user-images.githubusercontent.com/31011479/29692513-54a1053a-88e5-11e7-83de-dd4c8adf251c.png)
