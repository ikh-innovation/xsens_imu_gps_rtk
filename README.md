# xsens_imu_gps_rtk

This repository is about robot localization with RTK application. Briefly, the main equipment that we used is an [Xsens MTi-680G](https://www.xsens.com/products/mti-600-series) for the Rover kinematic measurement, as is RTK-enabled sensor, and secondly, we use a [simpleRTK2B](https://www.ardusimple.com/simplertk2b/) with [ZED-F9P module](https://www.u-blox.com/en/product/zed-f9p-module) on it, as the Base station. The communication between the Base and the Rover succeed with [XBee SX 868](https://www.ardusimple.com/product/radio-module-long-range/) for up to 14 km RF communication.


## Description
The robot localization process is unavoidable in mobile robotics. In order to accomplish really precise information about the location of the mobile robot (aka Rover), several methods could take place. Only a single GNSS receiver at the Rover to receive polar coordinates could be really worthless, as its accuracy is close to 2-4 meters. The [RTK](https://www.everythingrf.com/community/what-is-real-time-kinematics) is a method that allows us to increase that precision to centimeter-level by sending RTCM3.3 corrections messages from the Base to the Rover. For RTK applications with the Base-to-Rover distance smaller than 20 km, we can achieve 1-2 cm accuracy. This accuracy is still relative to the Base location, and not to the global coordinate system. That can be solved by using [NTRIP Servers](https://www.agsgis.com/What-is-NTRIP_b_42.html) to localize the Base in cm-level in the first place, by using RTCM messages that a local server provides. By doing this, we achieve to know the exact position of the Mobile Robot in centimeters relative to the Geographic Coordinate System. Furthermore, the information of the position in addition to the IMU information of the MTi-680G sensor can be fused in order to correct random drifting or misbehaving of the RTK method.


## Objective
This mobile robotics project aims to create a Mobile Robot for Agriculture applications, as mowing grass on a solar farm. The field first must be mapped in order to produce the lawn mowing path avoiding solar panels or other obstacles. Before the navigation of the robot took place, the localization problem must be solved with high precision. The navigation algorithm must know the current position of the robot and the positions of every obstacle. If this information is not well given by us the robot may crash and damage its parts, as it would be unable to follow the path.

As we mentioned before, we have to integrate some methods to result in the exact position of the robot. Without these methods, we could not estimate the position in order to solve the localization problem.

## Equipment
<p align="center">
  <img src="/images/equipment.PNG" width="500">
</P>

#### Base Station parts:
- simpleRTK2B with U-Blox ZED-F9P module ([link](https://www.ardusimple.com/simplertk2b/)))
- XBee SX 868 RF module + Antenna ([link](https://www.ardusimple.com/product/radio-module-long-range/))
- Calibrated Survey GNSS Multiband antenna ([link](https://www.ardusimple.com/product/calibrated-survey-gnss-multiband-antenna-ip67/))

#### Rover parts:
- Xsens MTi-680G ([link](https://www.xsens.com/products/mti-600-series))
- RS232-to-Everything adapter ([link](https://www.ardusimple.com/product/rs232-to-xbee-everything-adapter-3m-cable/))
- XBee SX 868 RF module + Antenna ([link](https://www.ardusimple.com/product/radio-module-long-range/))
- Calibrated Survey GNSS Multiband antenna ([link](https://www.ardusimple.com/product/calibrated-survey-gnss-multiband-antenna-ip67/))
<!--- - - U-Blox GNSS Multiband antenna ([link](https://www.ardusimple.com/product/ann-mb-00-ip67/))-->

## Implemantation

### Xsens MTi-680G Integration

- Download and Install [MT Software Suite](https://content.xsens.com/mt-software-suite-download?hsCtaTracking=ca04936d-827e-43ef-a5e9-1797c4d9a297%7C1e7a3b81-a8f1-40af-8464-97db700b3dec)
- Connect the sensor from the Host-Interface port via CA-MP-MTI-12 cable to the computer using the USB adapter.
- (optional) Open Magnetic Field Mapper and follow this [guide](https://www.xsens.com/hubfs/Downloads/Manuals/MT_Magnetic_Calibration_Manual.pdf) to calibrate magnetic sensors
- Open MT Manager and get used to it by this [guide](https://www.xsens.com/hubfs/Downloads/Manuals/MT%20Manager%20User%20Manual.pdf)
(you can watch some tutorials also to [tutorials.xsens](https://tutorial.xsens.com/))
  
<!--- 
- In Linux (ubuntu 18.04) follow the next steps:
  - Download MT Software Suite
```bash
lsusb
```

![alt text](/Screenshot%20from%202020-11-18%2014-15-05.png)
-->

### Base Station Configuration

- First, we took a camera tripod to set the base station. At the top an 5/8″ to 1/4″ adapter is used to place the GNSS receiver. Then we put in a box the 10000 mAh PowerBank (for stand alone using) and the simpleRTK2B with the XBee-LR module on it.

<p align="center">
  <img src="/images/base_station_tripod.jpg" width="400">
</P>

the inside of the box is like the image below:

<p align="center">
  <img src="/images/base_station_box.jpg" width="400">
</P>

*The reason that we set an additional power supply is to use the base station as a stand alone device. We plug the simpleRTK2B in a laptop to Survey-in and the we unplug the laptop. The base station powered by the powerbank and continues to send the RTCM messages.*

- (optional) Register in an NTRIP network to get access to RTCM messages to localize accurately the Base Station. We choose [igs-ip.net](http://www.igs-ip.net/) as it was free after registration. Before register to any network check to find if that network provides NTRIP Casters in your local area. 
   - Download [SNIP](https://www.use-snip.com/download/), which is a free NTRIP Caster software.
   - Open SNIP and look at the log the available IPs:
   
     ![alt text](/images/found.PNG)
   - Connect to one of the available IPs with port 2101:
   
     ![alt text](/images/connectip.PNG)
   - If everything is fine, this green label appears:
   
     ![alt text](/images/connected_green.PNG)
   - Add a new stream in Relay Streams tab:
   
     ![alt text](/images/addnewstream.PNG)
   - Add the IP of your network at the default port with your username and password if existed and press get mount points to find the available casters:
   
     ![alt text](/images/addnewstreamgu.PNG)
   - If everything is fine you will see this:
   
     ![alt text](/images/addnewstreamcompleted.PNG)
   
   - At this point, you have successfully set an NTRIP Caster in your local network. You are able to see the Input data increase. Only when we enable the connection with the NTRIP Client you will be able to see the output data.
 
 - Download and Install [U-Center](https://www.u-blox.com/en/product/u-center) from U-Blox
 - Read the [hookup-guide](https://www.ardusimple.com/simplertk2b-hookup-guide/#hw_overview) from Ardusimple to get used to the simpleRTK2B
 - Connect the simpleRTK2B from POWER+GPS port to your PC. At U-Center choose the baudrate (115 kbps) and connect to the available COM. You will be able to see this green blinking indicator at the bottom of the GUI:
 
   ![alt text](/images/baseconnected.PNG)
 - Follow this [guide](https://www.ardusimple.com/zed-f9p-firmware-update-with-simplertk2b/) to update Firmware (if necessary) of the ZED-F9P. (if the ZED-F9P bricked check [this](https://www.ardusimple.com/simplertk2b-hack-2/))
 - Follow this [guide](https://www.ardusimple.com/configuration-files/) to set the simpleRTK2B as the Base Station with U-Center by changing its configuration. You can also find in this repo the base config file in "config_files/simpleRTK2B_FW113_Base-00.txt"
 
 - (optional) If you set NTRIP Caster to your local network connect it to your Base Station:
 
   ![alt text](/images/ntripclient1.png)
 
 - (optional) Add your local IP and the port, update source table and select mount point.
 
   ![alt text](/images/ntripclient2.PNG) 
 
 - (optional) So you will be able to see something like this at the bottom of the GUI:
 
   ![alt text](/images/ntclient.PNG) 
   
 - (optional) After a few seconds, you will be able to see at the Data the DGNSS state, and 3D Acc decreases rapidly.
   
   ![alt text](/images/dgnss.PNG) 
   
 - As we configure successfully the base station (and set the NTRIP Caster) it's about time to localize the base station with U-Center by Survey-in. Go to View/MessageView or by pressing F9 will open Messages window. Navigate to UBX/CFG/TMODE3. Select Survey-in, add the minimum time and the position accuracy of your choice and press Send at the bottom-left of the GUI.
   
   ![alt text](/images/survey.png) 
 
*With NTRIP client enabled only in half of a minute and view to clear sky is possible to result in less than 2cm accuracy. Else you have to wait several hours or even a day to result in about 10 to 20 cm accuracy. (e.g. survey-in without NTRIP running for 4 hours to get 28cm accuracy)*

 - Navigate to UBX/NAV/SVIN and check the Status. This process will reach the end **only when both the observation time and the Mean 3D StdDev go True**
 - When the Status is "Successfully Finished" you will be able to see the TIME flag at the Fix Mode:
 
   ![alt text](/images/time.png)

 - At this step, we successfully set the base station localization. Now let's config the Base Station to export RTCM message via UART2 to the XBee. Go to UBX/CFG/PRT and choose UART2 as Target. And send this:
 
   ![alt text](/images/baseports1.PNG)
 
 - Before setting the radio communication, let's be sure that everything goes right by another method. We will provide those messages via the MT-Manager as an NTRIP Client using the local network. First go to U-Center and navigate to UBX/CFG/MSG at the messages tab. Find at message every RTCM message that the following table consists of. Be sure to enable USB port also and press send for every message.
 
| Message | Description                               |
|--------------|--------------------------------------|
| RTCM3.3 1005 | Stationary RTK reference station ARP |
| RTCM3.3 1074 | GPS MSM4 |
| RTCM3.3 1084 | GLONASS MSM4 |
| RTCM3.3 1094 | Galileo MSM4 |
| RTCM3.3 1124 | BeiDou MSM4 |
| RTCM3.3 1230 | GLONASS code-phase biases |

 - Then go to UBX/CFG/PRT and choose USB. Send the following configuration:
 
 ![alt text](/images/baseports2.PNG)
  
 - Then go to U-Center and choose NTRIP Server/Caster at the Receiver tab. Choose a port *(be careful, no to use the same port that SNIP uses)* and if you want Authentication, enable it with your desire account. Go to the Mount Point tab and select the desired messages to sent and click ok (unselect and select again the Get configuration automatically).
  
  ![alt text](/images/ntclientconf.PNG)
 
  - You will be able to see the NTRIP server at the bottom of the GUI. Only when you connect the sensor the grey indicator will turn green.
 
  ![alt text](/images/ntripserver.PNG)
  
- It's about time to assembly the Rover. Attach to a plate or to a construction the MTi-680G with the GNSS receiver to have fixed positions like this:
 
   ![alt text](/images/rover.png)
 
 - The MTi-680G gives the ability to insert the GNSS receiver position relative to the sensor for Level-Arm corrections with the MT-Manager. Measure the distance between the center of the sensor to the antenna. Insert the x,y,z vector in Device Settings/Level-Arm like this:
 
   ![alt text](/images/levelarm.PNG)
   
 - Select from Tools the NTRIP client to provide RTCM messages via the USB adapter to the sensor. Fill in your data and press start.
 
   ![alt text](/images/xsensntrip.PNG)
   
 - Define a workspace with a nice view of the sky. Add some markers for testing the RTK application.
 
   <!-- ![alt text](/images/workspace.PNG) -->
 
 - From the MT-Manager open the Status Data. The RTK_Status has to be HIGH.
   
   ![alt text](/images/rtkstatus.PNG)

      Flag State | Meaning
      --- | --- 
      0 | No RTK 
      0.5 | No RTK, Rx RTCM 
      1 | RTK Enable 

 - Also at the GNSS Satelite Status tab, you will be able to see some satellites with a light green color. These satellites are used for the RTK.
 
   ![alt text](/images/usedsatelites.PNG)

 - Open the Position Data tab to plot the sensor's position. Take the Rover and pass through the markers. Try to pass from markers several times to see if any offset in location took place. The repeatability has to be visible. (*you can watch our testing video in this [link](https://iknowhow-my.sharepoint.com/:v:/p/lgriparis/EcjeoDWlXOFAl_UHEqjRPOoB4hWtjdKegH0ogdybcZjFIw?e=StjAk3)*)

 - For the last step, lets include the XBee modules to the game, to enable the RF communication. 
 (*If you haven't already plug the XBee to the base station, turn off the power supply before and then plug it. So you may have to redo some steps from above.*)

The MTi-680G provides a different port for the RTCM correction messages. This port supports the RS232 communication protocol. Ardusimple provides a module (see [equipment](#Equipment)) for making the XBee module's output compatible to RS232 protocol. We connect a RS232 male connector to the CA-MP-MTI-4 cable. You have to be careful to connect the Tx of the RS232-to-everything to the Rx of the MTi-680G.

<p align="center">
  <img src="/images/rtcm_cable_rs232svg.png" width="400">
</P>

When you fix the cable, plug in the XBee module with its antenna to the board. Find a power supply at 5v (100mA in average) and connect it to the board too.

<p align="center">
  <img src="/images/rs232_to_everything.png" width="500">
</P>


So lets make a recap about our equipment. The Base Station with XBee looks like this:

<p align="center">
  <img src="/images/base.png" width="500">
</P>

And the Rover looks like this:
<p align="center">
  <img src="/images/rover_compact.jpg" width="500">
</P>

with its components inside the box:
<p align="center">
  <img src="/images/rover_with_xbee.png" width="500">
</P>

 - The XSens MTi-680G default baudrate of the RTCM port is set at 38 kbps. Our communication is set at 115kbps. To change the baudrate open the "Device Data View" of the MT-Manager
     
   ![alt text](/images/low_level_com.PNG)

 - Press the "GoTo Config" button to set the MTi-680G in the configuration state. 
  
   ![alt text](/images/gotoconfig.PNG)
 
 - At "Compose" find the "SetPortConfig" message and click edit.

   ![alt text](/images/edit_msg.PNG)

- Choose the "Number of items" 3, and send the a message similar to a following. Click "OK".

  <img src="/images/msg.PNG" width="450">

 - Then, click "Send" to send the message that you edit. 
   ![alt text](/images/send_msg.PNG)

Alternative, in MTmanager v2020.3 there is a dedicated button to change the RTCM port without having to sent a low level command. Open the Device Settings tab and choose your optimal baudrate of the RTCM config. See below:

<p align="center">
  <img src="/images/rtcmport.png" width="900">
</P

Hopefully, at this stage, you are done. Check the "Status Data" about the RTK_Status Flag. A very useful feature of the MT-Manager is that it can export KMZ files for plotting your tests in Google Earth. Press record (red circle) at the GUI, do your test movements, and press the record button again to stop it. MT-Manager exports the file of the recording as btm. Go to File/Open to open the btm file and export it as KMZ. The following gif, shows our testing. We tried to pass with the Rover two times above the sidewalk of the stadium. The orange arrow is the orientation vector of the MTi-680G. The test proved that the RTK application is really accurate, with centimeter-level precision.

<p align="center">
  <img src="/images/stadium_path.gif" width="800">
</P>

## Contact us

<!-- <img align="right" src="/images/ikh_logo.png" width="120"> -->

**Michalis Logothetis** | Email: mlogothetis@iknowhow.com
**Lefteris Griparis** | Email: lgriparis@iknowhow.com
<p align="right">
  <img src="/images/ikh_logo.png" width="100">
</P>