Table of Contents
=================

   * [OmniaCam](#omniacam)
      * [the App name and Icon](#the-app-name-and-icon)
      * [Nothing shows up when app is opened on its own](#nothing-shows-up-when-app-is-opened-on-its-own)
   * [How it works](#how-it-works)
   * [Video Demonstration](#video-demonstration)
      * [Taking Patient Pictures](#taking-patient-pictures)
      * [Taking Treatment Sheet Pictures](#taking-treatment-sheet-pictures)
   * [How Images are Saved](#how-images-are-saved)
      * [Image file name](#image-file-name)
      * [The subdirectory structure](#the-subdirectory-structure)
      * [Devices with no external SD Card](#devices-with-no-external-sd-card)
      * [Devices with external SD Card](#devices-with-external-sd-card)


# OmniaCam
A simple camera app that allows you to take multiple pictures and save each picture in dateTimeStamp format to external SD Card (if available)

&nbsp;

## the App name and Icon
<img src="https://user-images.githubusercontent.com/26863547/34756193-4937564a-f61d-11e7-92d5-0828db818d20.png" width="286" height="495">

## Nothing shows up when app is opened on its own
<img src="https://user-images.githubusercontent.com/26863547/34756313-05f5cdde-f61e-11e7-9483-8fa7b1efa2b7.png" width="286" height="495">

&nbsp;

&nbsp;



# How it works
The camera app only works when a patient record is passed from `www.omniacorp.com.au//cosme/site/inc/omniacam.php?[patient information]`

For instance, below is a **list of 4 patients**: 
![patient list](https://user-images.githubusercontent.com/26863547/34756394-a93b0540-f61e-11e7-8487-70363bcc1bae.png)


In order for the app to capture a corresponding patient information, it needs to capture the patient information after `omniacam.php?`. In order to do that, the user needs to tap on the corresponding patient's camera icon:
![camera icon to press](https://user-images.githubusercontent.com/26863547/34756909-2b459a84-f622-11e7-934d-ee46fb99ccae.png)

Once the corresponding camera icon is pressed, the app will automatically fire up and displays 4 things:
- the patient name [in red]
- the patient consultation ID [in bolded red]
- a blue button used to take patient pictures
- a green button used to take patient treatment sheet pictures

<img src="https://user-images.githubusercontent.com/26863547/34757124-54bbbe6a-f623-11e7-9e0e-3b67fa2cdd14.png" width="286" height="495">

When the user presses either **TAKE PATIENT PICTURES** or **TAKE TREATMENT SHEET PICTURES**, a blue check or a red check will appear next to the corresponding button pressed and the camera interface will open up. 

<img src="https://user-images.githubusercontent.com/26863547/34797823-cbec1766-f6ad-11e7-8f8c-3ee20d6dc7e1.png" width="286" height="495">
                                                                                                                                                                                                                                                                  <img src="https://user-images.githubusercontent.com/26863547/34797825-cd6a8c44-f6ad-11e7-9778-8b1f5c61f6b6.png" width="286" height="495">
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    Once the 2 buttons have been pressed, an additional **EXIT BUTTON** button will appear. This button essentially does the exact same thing as the back button but was added to adhere with client's request.
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    

For a little demonstration of the app, please take a look at the video demonstration section below

&nbsp;

&nbsp;


# Video Demonstration

## Taking Patient Pictures
https://www.youtube.com/edit?o=U&video_id=Qrlq5RYPklk

## Taking Treatment Sheet Pictures
https://www.youtube.com/edit?o=U&video_id=Ow339h15r_o

&nbsp;

&nbsp;


# How Images are Saved

## Image file name

Each image taken will be named **[patient_consultationID]_[yyyymmdd]_[hhmmss]**, where:
- **patient_consultationID** is the consultation ID of the corresponding patient given by the website that opens the app
- **yyyymmdd** is the year, month and date when the image is taken. 
- **hhmmss** is the hour, minute and second when the image is taken. 

For instance, in the demonstration above, one of the images was taken such that:
- **patient_consultationID** passed by the website was 5
- the image was taken on 10th January 2018
- the image was taken at 8:27:45 am in the morning

As such, this particular image was named **5_20180110_082745.jpg**

<img src="https://user-images.githubusercontent.com/26863547/34797611-15b617d0-f6ad-11e7-82da-189d262483aa.png">

&nbpsp;

&nbsp;

#Where Imagas are Saved


## The subdirectory structure

The subdirectory created by the app to save images is rooted at **Consultation Images**

<img src="https://user-images.githubusercontent.com/26863547/34758208-3430f258-f62a-11e7-99f6-da093a2b7002.png" width="286" height="495">

Inside this folder will be a list of folders, each one of them is aptly named **cons_[patient_consultationID]** and corresponds to the consultation ID for each patient whose pictures were taken by the app. 

<img src="https://user-images.githubusercontent.com/26863547/34796058-75e622e4-f6a8-11e7-85a7-83b85a449b80.png" width="286" height="495">


Inside each **cons_[patient_consultationID]** folder, 2 folders will be created. One of them will be called **Patient Pictures** and the other one will be named **Treatment Sheet Pictures**. 

<img src="https://user-images.githubusercontent.com/26863547/34796118-a09f30c0-f6a8-11e7-8324-d99ab1ec43c7.png" width="286" height="495">

When user chooses **TAKE PATIENT PICTURES** button, each corresponding image taken afterwards will be saved inside the **Patient Pictures** folder. On the other hand, when the **TAKE TREATMENT SHEET PICTURES** button is pressed, each corresponding image taken afterwards wil be saved inside the **Treatment Sheet Pictures** folder. 


***Patient Pictures***

<img src="https://user-images.githubusercontent.com/26863547/34796717-8079e1c6-f6aa-11e7-8b9a-d348581acf76.png" width="286" height="495">

***Treatment Sheet Pictures***

<img src="https://user-images.githubusercontent.com/26863547/34796718-8199ebfa-f6aa-11e7-904e-3a4e21d32dfc.png" width="286" height="495">

## Devices with no external SD Card

When the app is installed to devices with no external SD Card, just like the Google Pixel XL (1st gen) that I used to test the app on in this case, a subdirectory structure with a root folder named **Consultation Images** will be created in the root path of the device internal storage, usually at `/storage/emulated/0`. 

## Devices with external SD Card

For devices with external SD card, the app will automatically detect the external storage and the subdirectory structure rooted with **Consulatation Images** folder will be created in the app sandboxed storage folder, which is `/storage/emulated/externalSdCard/Android/data/com.OmniaCam.app`.


                                                                                                                                    
                                                                                                                                    
                                                                                                                                    
                                                                                                                                    
                                                                                                                                    















