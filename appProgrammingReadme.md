# Overview of the app programming

The app was coded so that any request to open the URL which is in the format of `www.omniacorp.com.au//cosme/site/inc/omniacam.php?[patient_information]` will open the app on the device it's installed on instead of redirecting its mobile browser to another web page. 

The app will then extract patient data from the `[patient_information]` passed to it and will create folders to save images taken for those patients. The patient data will then be used to name the corresponding patient folders/images inside the device storage system. 

In order to achieve this, several cordova plugins were utilised. However, only 2 of them are essential to the core functionality of the app:

&nbsp;

## cordova-universal-links-plugin

This plugin allows the programmer to create a **link** between a specific URL and a javascript function that your app will execute when this specific URL is requested on the mobile web browser. In this instance, that URL is `www.omniacorp.com.au//cosme/site/inc/omniacam.php?*`

This indicates that whenever the mobile browser requests that URL, the device will open the app instead of redirecting its web browser to another URL. The URL will be *subscribed* to a specific javaScript function. This is a step by step function on how it works:

1. Install the plugin by going into the cordova project directory and type `cordova plugin add cordova-universal-links-plugin`
2. Go to the `<project folder>/config.xml` and add these lines inside config.xml: 

``` 
   <universal-links>
        <host name="www.omniacorp.com.au">
            <path event="linkFunctionHandler" url="/cosme/site/inc/omniacam.php*" />
        </host>
    </universal-links>
```

The asterisk (*) indicates that the app will be opened regardless of what kind of string follows `omniacam.php` in the URL.  

3. Go to index.js, inside the `onDeviceReady: function()` the event `linkFunctionHandler` is subscribed to a  javaScript function named `linkFunctionHandler()` like this:

```    
   onDeviceReady: function() {
        console.log("Device is ready for work");
        this.receivedEvent('deviceReady');
  
        universalLinks.subscribe('linkFunctionHandler',linkFunctionHandler);
    },
```

4. A function called `linkFunctionHandler` is created inside index.js like so:

```
function linkFunctionHandler(eventData){
    // Do anything you want right here with the eventData object passed
    console.log("The linkFunctionHandler javascript function is executed");
    console.log("The URL which made the app executed this function was" + eventData.url);
}
```

The outpout of `linkFunctionHandler()` function on terminal in this case would be:

```
The linkFunctionHandler javascript function is executed
The URL which made the app executed this function was http://www.omniacorp.com.au/cosme/site/inc/omniacam.php?[patient_information]
```

&nbsp;

&nbsp;

# cordova-plugin-camera-with-exif

This plugin enables the app to take pictures and extract the EXIF data from the picture that was taken by the camera. EXIF **(Exchangable Image File Format)** data contains many information about a picture that is taken by the device's camera. It can tell user the GPS coordinate where the image was taken, the date/time stamp when the image was taken. The latitude/longitude where the images was taken, etc. 

Therefore, using this plugin also makes it possible to extract the date/time stamp of each picture taken by the camera and subsequently rename the image taken with this date/time stamp.

In order to utilise this plugin, a function called `takingPictures(eventData,parentDirectory)` is added and it takes 2 parameters:
1. eventData is the object passed the linkFunctionHandler() function and it contains the URL that opens the app as well as the corresponding patient information that is passed to the app from the mobile browser. 

2. `parentDirectory` is the name of the directory the app wants the image to be saved. If the blue button is pressed then the name of this `parentDirectory` shall be **Patient Pictures** and if the green button is pressed then the name of parentDirectory will be **Treatment Sheet Pictures**

The structure of this function is as follow: 


```
function takingPictures(eventData,parentDirectory){

    //below is the method that is provided by the plugin camera API
    navigator.camera.getPicture(function cameraSuccessCallBack(result){


        // convert JSON string to JSON Object
        var thisResult = JSON.parse(result);

        // convert json_metadata JSON string to JSON Object 
        var metadata = JSON.parse(thisResult.json_metadata);

        //extract image file URI from the JSON object 
        var imageFileURI = thisResult.filename; 

        //get date and time stamp and format it accordingly
        var dateTimeStamp = metadata.datetime;
        dateTimeStamp = dateTimeStamp.replace(/\s/g, '_');
        dateTimeStamp = dateTimeStamp.replace(/:/g, '');

        console.log("the image file URI is " + imageFileURI);
        console.log("The URL that opened the camera is " + eventData.url);
        console.log("The URL path component that opened the camera is " + eventData.url);
        console.log("The type query parameter passed is " + eventData.params['type']);
        console.log("The consultation number parameter passed is " + eventData.params['code']);
        console.log("The date and time stamp is " + dateTimeStamp);


        //save consultation and date/time number in a variable
        var consNumber = eventData.params['code'];
        var consFolderName = "cons_" + consNumber;
        var imageFormat = imageFileURI.substring(imageFileURI.lastIndexOf('.'));
        var newImageName = consNumber + "_" + dateTimeStamp + imageFormat;

        console.log("The new image name is " + newImageName);


        /*
        ===============================================================================================================
                                  THIS SECTION BELOW IS MEANT TO MOVE THE IMAGE FILE TO:
                  <root directory folder>/Consultation Images/cons_[consNumber]/{Patient Pictures OR Treament Sheet Pictures}
        ===============================================================================================================
        */

        window.resolveLocalFileSystemURL(imageFileURI,function(fileEntry){


            cordova.plugins.diagnostic.getExternalSdCardDetails(function(details){

                //save to external SD card if there's any on the device
                if(details.length > 0){
                    details.forEach(function(detail){
                        if(detail.canWrite && detail.type == "application"){

                            saveToNewPath(fileEntry,newImageName,detail.filePath,consFolderName,parentDirectory);
                        }
                    });

                //otherwise, save it to the device's internal storage 
                //with root folder as the new parent directory
                } else{
                    saveToNewPath(fileEntry,newImageName,cordova.file.externalRootDirectory,consFolderName,parentDirectory);
                }

            }, function(error){
                console.error(error);
                console.log("Have trouble with storing in SD Card");
            });
          
        },onErrorGettingImageFileEntry);


        //take more pictures
        takingPictures(eventData,parentDirectory);

    },cameraExit,options);


};
```
