# Trial Methodologies

> The methodologies described in this document were designed for the TraceTogether application, but we hope the information can be of use to test and trial your own implementation of OpenTrace.

## Table of contents
<!--ts-->
   * [Introduction](#introduction)
   * [Phone manufacturer and model differences](#phone-manufacturer-and-model-differences)
   * [The Ground Truth](#the-ground-truth)
   * [Small-scale Trial Methodology](#small-scale-trial-methodology)
   * [Indoor Office Trial Methodology](#indoor-office-trial-methodology)
   * [Static Trial Methodology and Examples](#static-trial-methodology-and-examples)
   * [Field Trial with Partner Organisation](#field-trial-with-partner-organisation)
   * [Anechoic Chamber Readings](#anechoic-chamber-readings)
   * [Acknowledgements](#acknowledgements)
<!--te-->

## Introduction

TraceTogether is a mobile application developed on top of the BlueTrace protocol to support existing nationwide efforts to combat COVID-19, by enabling community-driven contact tracing using Bluetooth technology. Based on the specifications of the encounter message in the TraceTogether implementation, the two noteworthy values that can be used for distance estimation are the Received Signal Strength Indicator (RSSI) and Device Model.

Distance estimation using BLE is possible using the RSSI measured by a Bluetooth Central reading from a Peripheral. However, the RSSI value exhibits significant variation and is subject to factors such as:
* Multipath fading
* Phone placement on a person, e.g. in a hand, in a pocket, proximal or distal to a receiver
* Environmental factors such as surface textures, geometry, and physical layout
* Device-specific characteristics such as chipset, antenna layout, and OS configurations

All these factors introduce random noise in estimates of distance.

The TraceTogether implementation aims to augment the contact-tracing process used by the Singapore Ministry of Health (MOH). In a nutshell, contact tracers currently interview a patient in order to build an activity mapping of the places and people they have come into contact with in recent memory. However, human memory is quite fallible as a source of truth, and it is highly unlikely for a person to remember all the strangers they have come into contact with.

MOH has come up with a guideline to determine if someone has been a close contact of a patient - the individual must have been within a 2 meter distance and present for at least 30 minutes. A device running the protocol will be able to pick up any other surrounding devices as long as they are in range, which could include devices more than 2 meters away and even behind multiple walls. A problem statement then arises when presenting a list of contacts based on the encounter records: not everyone will fit the criteria of a close contact.

To resolve this issue of too many false positives while maintaining a higher degree of true positives, we aim to apply an algorithm during post-processing to increase our precision of identifying close contacts befitting the criteria. Thus, the following trials in this document are designed to collect distinguishable data matching the close contact criteria, as well as data outside of it for tuning the algorithm.

As mentioned above, Device/Phone model is also essential for distance approximation, the details of which can be found in the [Phone manufacturer and model differences](#phone-manufacturer-and-model-differences) section. To be able to validate all the data collected by the devices, a solid [Ground Truth](#the-ground-truth) must be established.


## Phone manufacturer and model differences

Earlier on in our experiments it was found that the physical characteristics of every device can affect the quantity and quality of data collected. It can be expected that the bluetooth hardware in each chipset, the antenna layout, and even the phone’s OS can affect the performance of the BlueTrace implementation.

Hardware characteristics can be controlled by collecting readings in an EMC chamber (anechoic or semi-anechoic). These readings can then be applied in post-processing to calibrate the RSSI values from the trial. See [Anechoic Chamber Readings(#anechoic-chamber-readings) section for more information.

One critical factor to take into account is the manufacturer’s own battery saving features built into their OS. This is noted in particular for Android vendors, and each flavor of Android can affect the amount of data that can be collected over a period of time. Some Android OEMs aggressively suppress the performance of the app, and at times may even kill the service or throttle the bluetooth hardware. A good reference guide can be found at: https://dontkillmyapp.com/. By following the steps to whitelist the app for each device, we can lessen the impact that manufacturer battery saving features affects our readings.

An added precaution that we have introduced is to have trial participants interact as often as they can with their device, so the power-saving functions do not kick in and disrupt the app service.

<p align="center">
	<img src="/src/images/chipset_spec.jpg">
	<br><i>Pictured above: 3 different models (Samsung Galaxy S9, S9+, Note 9) with identical chipset specifications. Will readings be consistent?</i>
</p>


## The Ground Truth

The Ground Truth refers to data collected throughout the trial that most accurately describes the actual situation that happens on the ground. This data will be compared against the data gathered from the phones to validate if the protocol implementation is working according to expectations. 

There are several methods of gathering the ground truth that we have explored in our trials. The most common approach we used would be to use log sheets and have most if not all important interactions noted down. The data collected can be qualitative and quantitative, whichever makes the most sense for post-processing.

For some of our earlier trials, a script would be written for each participant to follow or roleplay. The movements in the scripts are deliberately arranged so the organisers know which participant has interacted with who. Any participant can then be designated as the “infected” individual afterwards and the script used as the source of truth to his interactions.

The main drawback to our current methods of getting the Ground Truth is that it is manual and tedious work to make sense of the data afterwards. There are other options that can be explored, such as setting up cameras around the trial area and using video analytics to identify and track participants. 

<p align="center">
	<img src="/src/images/movement_schedule.png">
	<br><i>Pictured above: A movement schedule for participants of the trial to follow.</i>
</p>


## Small-scale Trial Methodology

A less movement-intensive trial format was used in the office environment in the early stages of development to validate our implementation. The main objectives here were:

1. To be able to detect devices in close contact with the “infected” user in a closed confined space where participants do not move much for a duration.

2. To be able to obtain consistent readings of those seated in close proximity.

<p align="center">
	<img src="/src/images/meeting_room_layout.png">
	<br><i>Pictured above: Layout of meeting room.</i>
</p>
&nbsp;
<p align="center">
	<img src="/src/images/meeting_results.png">
	<br><i>Pictured above: Sample of results from trial</i>
</p>

The data gathered during these trials revealed that we needed to improve consistency of readings across multiple devices (note the highlighted gaps). 

It was also here that we realised the importance of tuning our field data with 2m distance data collected from [a more controlled environment](#anechoic-chamber-readings) due to the high standard deviations for each device.

<p align="center">
	<img src="/src/images/raw_rssi_chart.png">
	<br><i>Pictured above: Raw RSSI readings of devices at 2m taken in an anechoic chamber.</i>
</p>

## Indoor Office Trial Methodology

Using our office space to simulate regular common daily interactions, we roleplay scenarios where an app user is diagnosed with the virus and the team traces the potential list of affected app users from the data collected.

The main objectives here are:

1. To be able to detect devices in close contact with an “infected” user who moves to predetermined locations around the environment.

2. To be able to approximate the duration of how long a device remains in proximity to an “infected” user.

With these in mind, the team drafted different test scenarios for each participant to roleplay. These scenarios involved movement to various designated areas within the office for a period of time.

<p align="center">
	<img src="/src/images/office_layout.png">
	<br><i>Pictured above: Layout of office environment used for this trial.</i>
</p>
&nbsp;
<p align="center">
	<img src="/src/images/roleplay_scenarios.jpg">
	<br><i>Pictured above: Roleplay scenarios for each participant. “Infected” users are highlighted.</i>
</p>

Post-trial, “infected” users upload their phone data to our servers and the team gathers to make guesses of potentially affected users. As the list of affected users is already defined from the start by the organiser, this is used as the Ground Truth to compare the gathered data against. We also run the data through our data explorer to better visualise the proximity of users.

<p align="center">
	<img src="/src/images/post-trial_analysis.png">
	<br><i>Pictured above: Post-trial Analysis flow.</i>
</p>


## Static Trial Methodology and Examples

A static trial is essentially a trial with many stationary devices designed to “stress test” the stability and connectivity of the devices using our implementation. 

The main objectives in this case are the following:

1. To test connectivity between various devices over a length of time. 
In a real world scenario where there can be many devices near each other (e.g a crowded train), any device should be able to pick up other surrounding devices within a reasonable time.

2. To test for consistency and stability of the protocol implementation over a length of time.
Assuming that devices do not move around, the pair-wise RSSI readings should be of a consistent range without much fluctuation. The devices should also be attempting to establish connections throughout the entire duration.

3. To test battery drain over a length of time.
Battery drain is a concern for our users, we want to ensure that the battery consumption across manufacturers and models is reasonable with our implementation.

4. To note down patterns of gaps within the data caused by manufacturer battery saving features or Android Doze mode.
As covered in an earlier section, the scanning and advertising frequency of each device can be affected by manufacturer battery saving features built into the OS. This type of trial is useful to test tweaks to Bluetooth power settings in our implementation, as well as the workarounds for users.

For all cases above, all devices should be placed within range of each other and allowed to run for a longer period of time. In our own trials, we installed debug builds of the app on all our available test devices and set them within the 2 meter range requirement for close contact to run overnight. We chose test devices that were part of a top 50 list of popular models in Singapore, which includes devices from most major manufacturers as well as older models up to 5 years old.

<p align="center">
	<img src="/src/images/static_trial_devices_1.jpg">
	<img src="/src/images/static_trial_devices_2.jpg">
	<br><i>Pictured above: Various devices within close proximity for an overnight static trial.</i>
</p>
&nbsp;
<p align="center">
	<img src="/src/images/static_trial_results_1.jpg">
	<br><i>Pictured above: A preview of results from a static trial, dots on each line indicate a pairwise connection from another unique device. Note the pattern of dots per line varies by model.</i>
</p>
&nbsp;
<p align="center">
	<img src="/src/images/static_trial_results_2.jpg">
	<br><i>Pictured above: Another set of results later on in the development cycle after tweaking performance settings, much more consistent connection patterns observed.</i>
</p>
&nbsp;
<p align="center">
	<img src="/src/images/static_trial_results_closeup.jpg">
	<br><i>Pictured above: A close-up view of the pair-wise data collected on a device. A way to visualise the data on each device within a debug build is highly recommended for convenience.</i>
</p>

For even smaller scale static trials, we have also used a microwave oven to great effect - the faraday cage of the microwave partially shields our devices from outside interference for cleaner results.

<p align="center">
	<img src="/src/images/static_trial_microwave.jpg">
	<br><i>Pictured above: Small scale stability test in a microwave!</i>
</p>

A different static set-up was also used to determine if pair-wise RSSI values collected from devices closer to each other are really indicative of their close proximity, as compared to devices further away. In this case, we used our office environment to simulate real-world conditions. Each cluster of devices were also left untouched to run overnight.

<p align="center">
	<img src="/src/images/static_trial_cluster_layout.png">
	<br><i>Pictured above: Layout used for this particular trial. Devices in each Zone should have better pair-wise readings to each other versus devices in the other zone.</i>
</p>
&nbsp;
<p align="center">
	<img src="/src/images/static_trial_cluster_zone1.png">
	<br><i>Pictured above: Zone 1 of trial.</i>
</p>
&nbsp;
<p align="center">
	<img src="/src/images/static_trial_cluster_zone2.png">
	<br><i>Pictured above: Zone 2 of trial.</i>
</p>
&nbsp;
<p align="center">
	<img src="/src/images/static_trial_cluster_triangle.jpg">
	<br><i>Pictured above: An extension of the trial to 3 different clusters around the office.</i>
</p>

## Field Trial with Partner Organisation

This trial was conducted later in the development cycle as we worked with another organisation within Singapore to test our implementation on a larger scale. The main objectives were as follows:

1. To allow participants to go about their daily routines without too many deliberate movements on their part.

2. To observe app performance and stability in a different setting; main differences from our office setup are a larger area spanning more than 1 floor and more participants.

3. To compare the effectiveness of our interpreted data with real contact-tracing procedures.

4. To validate how our implementation was used by actual users and identify potential areas for improvement.

### Outline of the entire Trial
#### 1 - Identify areas of interest

Prior to the trial, we did a survey of the location in which the trial would be conducted. In this case, it would be an area within a military camp spanning a few buildings. We identified areas where participants would typically congregate during the course of their daily routines such as meeting rooms and offices.

#### 2 - Design data collection procedure for the Ground Truth

As we did not want to interfere with participant’s routine, we had to come up with a way of knowing the Ground Truth without having the participants perform deliberate movements. We found that placing log sheets within each area of interest (rooms) to let participants clock-in and clock-out was a good way to capture their movements around the area.

<table>
  <thead>
    <caption>LOGSHEET FOR ROOM 1</caption>
	<tr>
	  <th>Name</th>
	  <th>Time-in</th>
	  <th>Time-out</th>
	</tr>
  </thead>
  <tbody>
	<tr>
	  <td>TAN AH KOW</td>
	  <td>10:34</td>
	  <td>12:13</td>
	</tr>
	<tr>
	  <td>LIM HO SEY</td>
	  <td>11:00</td>
	  <td>13:15</td>
	</tr>
  </tbody>
</table>
<i>Pictured above: A sample of the log sheet that would be pasted on the door of each room.</i>

#### 3 - Conduct trial

A briefing to all participants was given before commencing. During the briefing, participants were shown how to install and use the app. Participants had to fill and submit a registration form for us to note their phone number and device model. They were also given explicit instructions to make a best effort at logging their movements when entering the various areas of interest.

<table>
  <thead>
  	<caption>REGISTRATION FORM</caption>
	<tr>
	  <th>Name</th>
	  <th>Mobile No.</th>
	  <th>Device OS</th>
	  <th>Device Manufacturer</th>
	  <th>Device Model</th>
	</tr>
  </thead>
  <tbody>
	<tr>
	  <td>TAN AH KOW</td>
	  <td>91234567</td>
	  <td>Android</td>
	  <td>Samsung</td>
	  <td>Galaxy Note 9</td>
	</tr>
	<tr>
	  <td>LIM HO SEY</td>
	  <td>98765432</td>
	  <td>iOS</td>
	  <td>Apple</td>
	  <td>iPhone XS</td>
	</tr>
  </tbody>
</table>
<i>Pictured above: A sample of the registration form given to participants that would form part of our Ground Truth.</i>

#### 4 - Conduct standard contact-tracing exercise

After the trial, we identified a few random participants as our “infected” cases and asked them to sit down with us for a contact-tracing interview. Our interview was a simulation of a real-world procedure used by the health authorities, in this case an activity mapping exercise.

<table>
  <thead>
	<caption>ACTIVITY MAP FOR: TAN AH KOW</caption>
	<tr>
	  <th>Time From</th>
	  <th>Time To</th>
	  <th>Description of Activity</th>
	  <th>Place visited</th>
	  <th>Contacts if available</th>
	</tr>
  </thead>
  <tbody>
	<tr>
	  <td>11:15</td>
	  <td>12:15</td>
	  <td>Working in office</td>
	  <td>Room 1</td>
	  <td>LIM HO SEY</td>
	</tr>
	<tr>
	  <td>12:15</td>
	  <td>13:30</td>
	  <td>Lunch at Canteen</td>
	  <td>Canteen</td>
	  <td>-</td>
	</tr>
  </tbody>
</table>
<i>Pictured above: A sample of the activity map template used in our contact-tracing interview.</i>

#### 5 - Establish Ground Truth data from trial

<table>
  <thead>
	<caption>GROUND TRUTH</caption>
	<tr>
	  <th>Room</th>
	  <th>Name</th>
	  <th>Mobile No.</th>
	  <th>Device Model</th>
	  <th>Time-in</th>
	  <th>Time-out</th>
	</tr>
  </thead>
  <tbody>
	<tr>
	  <td>1</td>
	  <td>TAN AH KOW</td>
	  <td>91234567</td>
	  <td>SM-N960F</td>
	  <td>10:34</td>
	  <td>12:13</td>
	</tr>
	<tr>
	  <td>1</td>
	  <td>LIM HO SEY</td>
	  <td>98765432</td>
	  <td>iPhone XS</td>
	  <td>11:00</td>
	  <td>13:15</td>
	</tr>
  </tbody>
</table>
<i>Pictured above: Established Ground Truth data from combining registration form and room logs.</i>

#### Assumptions during evaluation
1. All personnel in the same room are deemed to be in close contact.

2. Personnel in different rooms are deemed to be not in close contact.

3. Personnel that are not recorded in any room at any point of time will be considered to be out of sight.

#### Challenges faced
1. Establishing the Ground Truth

As filling in the log sheets is best effort work, we realise that some participants may forget to clock their timings in or out of a room. To mitigate this, we made a prominent sign to remind participants to do so, and periodically sent out reminders via a group chat.

2. Gathering device data

During the course of the trial, we found that some participants had difficulty with utilising the app properly. This is referring to our implementation on iOS where we are limited to collecting data only when the app is in the foreground. 

3. Gathering activity maps for contact-tracing

During our interviews, we found that participants interacted with people who were not part of the trial. Effort was required to clean up our contact-tracing data to only include participants, as well as remove other qualitative data irrelevant to the trial.

With the Ground Truth (or the closest we can get) established, our post-trial analysis involves comparing the contacts established in the Activity Map against the contacts from the interpreted data from the participant’s device

## Anechoic Chamber Readings
Due to the various device-specific characteristics such as chipset, antenna layout and even OS power configurations, we used 2 meter proximity measurements from a full anechoic chamber to tune the RSSI readings from our trials.

We refer to the methodology used in the following paper to collect these readings:

__Distance Estimation of Smart Device using Bluetooth__
<br>https://www.thinkmind.org/download.php?articleid=icsnc_2013_1_30_20039

<p align="center">
	<img src="/src/images/chamber_sa.jpg">
	<img src="/src/images/chamber_ant.jpg">
	<br><i>Pictured above: The portable spectrum analyser equipment used for our readings.</i>
</p>
&nbsp;
<p align="center">
	<img src="/src/images/chamber_main.jpg">
	<br><i>Pictured above: The full anechoic chamber used for the measurements.</i>
</p>

Within the chamber, we would place a single device running the Bluetooth advertisements at a 2 meter distance from the antenna. The device would have all radio functionality except the Bluetooth turned off. The readings for all devices would be taken from a single orientation - with the top of the phone pointing at the antenna. The readings for all our devices will be taken from a single anechoic chamber.

<p align="center">
	<img src="/src/images/chamber_setup1.png">
	<img src="/src/images/chamber_setup2.jpg">
	<br><i>Pictured above: Placement of the device to antenna. We also experimented with the differences between a device in the open and a device in a bag.</i>
</p>

On the spectrum analyser, we used the peak-hold function and recorded the values of each “spike”. We take multiple readings (20-40) and calculate the median and average after factoring the loss from the measurement instruments (e.g the loss from length of cable).

<p align="center">
	<img src="/src/images/chamber_reading_1.jpg">
	<img src="/src/images/chamber_reading_2.jpg">
	<br><i>Pictured above: Readings from different devices, note some devices prefer to advertise at fixed channels.</i>
</p>
<br>&nbsp;
<table>
  <thead>
	<caption>DEVICE CALIBRATION DATA</caption>
	<tr>
	  <th>Manufacturer</th>
	  <th>Model Name</th>
	  <th>Model</th>
	  <th>Orientation</th>
	  <th>Average Reading(dBm)</th>
	</tr>
  </thead>
  <tbody>
	<tr>
	  <td>APPLE</td>
	  <td>iPhone X</td>
	  <td>iPhone X</td>
	  <td>Top</td>
	  <td>-53.84</td>
	</tr>
	<tr>
	  <td>SAMSUNG</td>
	  <td>Galaxy Note 9</td>
	  <td>SM-N960F</td>
	  <td>Top</td>
	  <td>-59.64</td>
	</tr>
	<tr>
	  <td>SAMSUNG</td>
	  <td>Galaxy S8</td>
	  <td>SM-G950F</td>
	  <td>Top</td>
	  <td>-69.13</td>
	</tr>
	<tr>
	  <td>SAMSUNG</td>
	  <td>Galaxy S10+</td>
	  <td>SM-G975F</td>
	  <td>Top</td>
	  <td>-57.87</td>
	</tr>
	<tr>
	  <td>SAMSUNG</td>
	  <td>Galaxy Note 10+</td>
	  <td>SM-N975F</td>
	  <td>Top</td>
	  <td>-49.90</td>
	</tr>
	<tr>
	  <td>APPLE</td>
	  <td>iPhone 6</td>
	  <td>iPhone 6</td>
	  <td>Top</td>
	  <td>-59.06</td>
	</tr>
	<tr>
	  <td>HUAWEI</td>
	  <td>Mate 20 Pro</td>
	  <td>LYA-L29</td>
	  <td>Top</td>
	  <td>-70.56</td>
	</tr>
	<tr>
	  <td>SAMSUNG</td>
	  <td>Galaxy A70</td>
	  <td>SM-A705MN</td>
	  <td>Top</td>
	  <td>-61.78</td>
	</tr>
	<tr>
	  <td>SONY</td>
	  <td>Xperia XZ1</td>
	  <td>G8342</td>
	  <td>Top</td>
	  <td>-59.54</td>
	</tr>
	<tr>
	  <td>XIAOMI</td>
	  <td>Mi A1</td>
	  <td>Mi A1</td>
	  <td>Top</td>
	  <td>-63.52</td>
	</tr>
	<tr>
	  <td>GOOGLE</td>
	  <td>Pixel 3a XL</td>
	  <td>Pixel 3a XL</td>
	  <td>Top</td>
	  <td>-58.41</td>
	</tr>
	<tr>
	  <td>XIAOMI</td>
	  <td>Pocophone F1</td>
	  <td>Pocophone F1</td>
	  <td>Top</td>
	  <td>-55.72</td>
	</tr>
  </tbody>
</table>
<i>Pictured above: A sample of device readings from a single anechoic chamber</i>
<br>&nbsp;
<br>We will be including the dataset used in our tuning as part of the OpenTrace repository, all parties are welcome to contribute. We would also like to put a call out there to major phone manufacturers; any factory calibration data for your BLE implementations will be helpful too!

## Acknowledgements
We are indebted to the:-
* Republic of Singapore Air Force (RSAF)
* Institute for Infocomm Research (I2), a research entity of the Agency for Science, Technology and Research (A\*STAR)
* Rohde & Schwarz Singapore
* Infocomm Media Development Authority (IMDA)
* Nanyang Polytechnic
* Samsung Singapore

for the loan and use of facilities and equipment that was used in these tests and trials.
