# Overview of the app programming

The app was coded so that any request to open the URL which is in the format of `www.omniacorp.com.au//cosme/site/inc/omniacam.php?[patient_information]` will open the app on the device it's installed on instead of redirecting its mobile browser to another web page. 

The app will then extract patient data from the `[patient_information]` passed to it and will create folders to save images taken for those patients. The patient data will then be used to name the corresponding patient folders/images inside the device storage system. 

In order to achieve this, several cordova plugins were utilised. Each of them is  essential to the core functionality of the app.

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

   The asterisk (\*) indicates that the app will be opened regardless of what kind of string follows `omniacam.php` in the URL.  

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

## cordova-plugin-camera-with-exif

This plugin enables the app to take pictures and extract the EXIF data from the picture that was taken by the camera. EXIF **(Exchangable Image File Format)** data contains many information about a picture that is taken by the device's camera. It can tell user the GPS coordinate where the image was taken, the date/time stamp when the image was taken. The latitude/longitude where the images was taken, etc. 

Therefore, using this plugin also makes it possible to extract the date/time stamp of each picture taken by the camera and subsequently rename the image taken with this date/time stamp.

In order to utilise this plugin, a function called `takingPictures(eventData,parentDirectory)` is added and it takes 2 parameters:
1. `eventData` is the object passed the `linkFunctionHandler()` function and it contains the URL that opens the app as well as the corresponding patient information that is passed to the app from the mobile browser. 

2. `parentDirectory` is the name of the directory the app wants the image to be saved. If the blue button is pressed then the name of this `parentDirectory` shall be named **Patient Pictures** and if the green button is pressed then the name of parentDirectory will be **Treatment Sheet Pictures**

The structure of this function is as follow: 


```
function takingPictures(eventData,parentDirectory){

    // Define the options for camera
    var options = { correctOrientation: true,
                    quality: 100};

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


        //save consultation and date/time number in a variable
        var consNumber = eventData.params['code'];
        var consFolderName = "cons_" + consNumber;
        var imageFormat = imageFileURI.substring(imageFileURI.lastIndexOf('.'));
        var newImageName = consNumber + "_" + dateTimeStamp + imageFormat;


        /*
        ===============================================================================================================
                                  THIS SECTION BELOW IS MEANT TO MOVE THE IMAGE FILE TO:
                  <root directory folder>/Consultation Images/cons_[consNumber]/{Patient Pictures OR Treament Sheet Pictures}
        ===============================================================================================================
        */

       window.resolveLocalFileSystemURL(imageFileURI,function(fileEntry){


            cordova.plugins.diagnostic.getExternalSdCardDetails(function(sdCardArray){

                //save to external SD card if there's any on the device
                if(sdCardArray.length > 0){
                    sdCardArray.forEach(function(sdCard){
                        if(sdCard.canWrite && sdCard.type == "application"){

                            saveToNewPath(fileEntry,newImageName,sdCard.filePath,consFolderName,parentDirectory);
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
&nbsp;

&nbsp;

## cordova.plugins.diagnostic

In the previous section of the *cordova-plugin-camera-with-exif*, this block of code can be seen:

```
   /*
        ===============================================================================================================
                                  THIS SECTION BELOW IS MEANT TO MOVE THE IMAGE FILE TO:
                  <root directory folder>/Consultation Images/cons_[consNumber]/{Patient Pictures OR Treament Sheet Pictures}
        ===============================================================================================================
        */

       window.resolveLocalFileSystemURL(imageFileURI,function(fileEntry){


            cordova.plugins.diagnostic.getExternalSdCardDetails(function(sdCardArray){

                //save to external SD card if there's any on the device
                if(sdCardArray.length > 0){
                    sdCardArray.forEach(function(sdCard){
                        if(sdCard.canWrite && sdCard.type == "application"){

                            saveToNewPath(fileEntry,newImageName,sdCard.filePath,consFolderName,parentDirectory);
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
```

This block of code uses the cordova.plugin.diagnostic API to detect any presence of external SD Card inside the device storage system. Unlike other plugins, the usage of this API is quite straightforward:

1. The function `cordova.plugins.diagnostic.getExternalSdCardDetails(function(sdCardArray))` detects the presence of external SD Card inside the device. It will then create an array named `sdCardArray` which contains `sdCard` object as its elements. 

Subsequently, a simple check is necessary to find out whether the array is empty (no external SD Card detected). This is simply done by:
 
```
//scan the presence of external SD Cards
if(sdCardArray.length > 0){
 
      //scan each sdCard object
      
 }else{
      
      //no external SD Card detected, save to internal storage
 }
 
```

3. Scan each sdCard object until the app finds an external SD Card sandboxed storage for the app that is writable. Next,go to the specified subdirectory folder where the image will be saved by using the `saveToNewPath()` function. The `saveToNewPath()` function will automatically create the subdirectory structure if it hasn't been created. 

```
sdCardArray.forEach(function(sdCard){
   if(sdCard.canWrite && sdCard.type == "application"){

       saveToNewPath(fileEntry,newImageName,sdCard.filePath,consFolderName,parentDirectory);
   }
});
```

&nbsp;

&nbsp;

## cordova-plugin-file

The `saveToNewPath()` function from the previous 2 sections utilise the API offered by cordova-plugin-file. This plugin enables the creation of new directories inside an Android device's file system and it's also responsible for the renaming process of the image file taken by the camera. 


```
function saveToNewPath(fileEntry,newImageName,rootDirectory,consFolderName,parentDirectory){

    var rootDirectory = rootDirectory;

    /* 
    ========================================================================
                                SAVE FILE TO:
    <rootDirectory>/Consulation Images/consFolderName/parentDirectory
    ========================================================================
    */
    window.resolveLocalFileSystemURL(rootDirectory,function(rootDirEntry){

        rootDirEntry.getDirectory("Consultation Images", {create: true}, function(imgDirEntry){
            imgDirEntry.getDirectory(consFolderName, {create: true}, function(consDirEntry){
                consDirEntry.getDirectory(parentDirectory, {create: true}, function(parentDirEntry){

                    // move the file entry to new directory
                    fileEntry.moveTo(parentDirEntry, newImageName, function(newFileEntry){

                        console.log("The new file entry full path is " + newFileEntry.fullPath);

                    }, function(Error){
                        console.log("the move operation has failed");
                    });

                }, onErrorCreatingParentDirEntry);

            }, onErrorCreatingConsDirEntry);
        }, onErrorCreatingImagesDirEntry);

    });

}

```

The code used here is quite straightforward and self-explanatory. 
- *rootDirEntry* refers to either:

  1.`/storage/emulated/0 -- internal storage path when there's no external SD Card detected`
  2.`/storage/externalSdCard/Android/data/com.OmniaCam.app/files -- the app's sandboxed storage inside the device's external SD Card`
  
- *imgDirEntry* is refers to the folder **Consultation Images**
- *consDiEntry* refers to the **cons_[patient_consultationID]** folder name
- *parentDirEntry* is either **Patient Pictures** or **Treatment Sheet Pictures** folder where the image itself will be saved
- *newFileEntry* is the new file entry that points to the new image created after moving the old image to this specified subdirectory.


