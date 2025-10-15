## Introduction
File Formatting Tool (FFT) is a Microsoft Word add-in. This program is specifically developed to assist with the formatting requirements of the health authorities' submission.

## Get Started
### 0. Prerequisites
1. A machine (iPad, desktop, laptop, VM...) 🙃
2. Microsoft Word (2016 or later) / Microsoft Word Web

### 1. Download the Certificate and Manifest
1. Create a new folder to save them.
2. Click [001. Certificate and Manifest](https://topalliancebiousa.sharepoint.com/:f:/r/sites/TopallianceRA/Regulatory%20Tools/File%20Formatting%20Translator%20(FFT)/001.%20Certificate%20and%20Manifest?csf=1&web=1&e=uYJVga) to download.
   

### 2. Import the Certificate to the Local Machine
1. Double-click the cert and select "Install Certificate".
      ![1](/fft/images/6.png)  
2. Select current user / Local Machine (access to all the users).
3. Select "Place all certificates in the following store". -> Browse -> "Trusted Root Certification Authorities" -> Ok
      ![1](/fft/images/7.png)  
5. Pop-up window: "The import is successful".

### 3. Sideload the Manifest
Choose the Operating System (OS) you use and finish the sideloading:
1. **MacOS / iPadOS**:
   1. Use Finder to sideload the manifest file -> Open Finder and then enter `Cmd+Shift+G` to open the Go to folder dialog.
   2. Enter `/Users/<username>/Library/Containers/com.microsoft.Word/Data/Documents/wef`.
   3. If the wef folder doesn't exist on your computer, create it.
2. **Windows**:
   
   *Reference: [this page](https://learn.microsoft.com/en-us/office/dev/add-ins/testing/create-a-network-shared-folder-catalog-for-task-pane-and-content-add-ins).*
   1. select the new folder with saved manifest.
   2. Right click -> Show more options -> Give access to -> Specific people
   3. Select the owner -> Share -> Done. 
   
      ![1](/fft/images/1.png)
   
   4. Open Word Options -> Trust Center -> Trust Center Setting -> Trusted Add-in Catalogs
   5. Right Click saved manifest -> Sharing -> Copy the text under "Network Path"
   
      ![2](/fft/images/2.png)
   
   6. Paste the text into "Catalog Url" -> Add catalog -> Check "Show in Menu" -> OK
   
      ![3](/fft/images/3.png)
   
   7. Restart the Word -> Open File -> Home -> Add-ins -> Advanced -> "Contoso Task Pane" -> Add -> Show Task Pane
   
      ![4](/fft/images/4.png)
3. ~~Linux:~~ (No longer support)
   1. ~~Linux may only use the web version to sideload. Check step 2.4.~~
4. ~~Web:~~ (No longer support)
   1. ~~Open [Word on the web](https://word.cloud.microsoft/?wdOrigin=OFFICECOM-WEB.APPGALLERY)~~
   2. ~~Select Home -> Add-ins, then select More Settings.~~
   3. ~~On the Office Add-ins dialog -> select Upload My Add-in.~~
   4. ~~Browse to the add-in manifest file -> Select Upload.~~

### 4. Add CA cert and Trust
1. **MacOS**
   1. Click [here]() to download the CA cert `rootCA.pem`.
   2. Double-click `rootCA.pem`. This will open "Keychain Access".
   3. Select "System" keychain (top left).
   4. Right-click the certificate you just downloaded -> Click "Get Info".
   5. Expand "Trust" -> Set When using this certificate to "Always Trust".

### 5. FAQ
1. I can't refresh the FFT, What should I do?
   Try to clean cache in Word then restart the Word and refresh Add-in.
   1. Win + R -> `%LOCALAPPDATA%`
   2. Direct to `\Microsoft\Office\16.0\Wef`
   3. Delete the cache files folder
   ![5](/fft/images/5.png)
