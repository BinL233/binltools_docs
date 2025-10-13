## Introduction
File Formatting Tool (FFT) is a Microsoft Word add-in. This program is specifically developed to assist with the formatting requirements of the health authorities' submission.

## Get Started
### 0. Prerequisites
1. A machine (iPad, desktop, laptop, VM...) 🙃
2. Microsoft Word (2016 or later) / Microsoft Word Web

### 1. Download the Manifest
1. The manifest file is required to access the service. Click [here](https://www.binltools.com/api/download/manifest_binltools.xml) to download.
2. create a new folder to save the manifest.

### 2. Sideload the Manifest
Choose the Operating System (OS) you use and finish the sideloading:
1. Mac / iPad:
   1. Use Finder to sideload the manifest file -> Open Finder and then enter `Cmd+Shift+G` to open the Go to folder dialog.
   2. Enter `/Users/<username>/Library/Containers/com.microsoft.Word/Data/Documents/wef`.
   3. If the wef folder doesn't exist on your computer, create it.
2. Windows:
   Reference: [this page](https://learn.microsoft.com/en-us/office/dev/add-ins/testing/create-a-network-shared-folder-catalog-for-task-pane-and-content-add-ins).
   1. select the new folder with saved manifest.
   2. Right click -> Show more options -> Give access to -> Specific people
   3. Select the owner -> Share -> Done. <img width="591" height="418" alt="Screenshot 2025-10-13 141807" src="https://github.com/user-attachments/assets/ff5aa902-a0b5-4840-87bf-d1765276bddd" />
   4. Open Word Options -> Trust Center -> Trust Center Setting -> Trusted Add-in Catalogs
   5. Right Click saved manifest -> Sharing -> Copy the text under "Network Path" <img width="363" height="481" alt="Screenshot 2025-10-13 151549" src="https://github.com/user-attachments/assets/e791012b-5a48-4616-8dd5-faf01d45de62" />
   6. Paste the text into "Catalog Url" -> Add catalog -> Check "Show in Menu" -> OK  <img width="813" height="395" alt="Screenshot 2025-10-13 151749" src="https://github.com/user-attachments/assets/37c5eaf8-873b-4896-9683-3af5513dd0d5" />
   7. Restart the Word -> Open File -> Home -> Add-ins -> Advanced -> "Contoso Task Pane" -> Add -> Show Task Pane <img width="366" height="177" alt="Screenshot 2025-10-13 152018" src="https://github.com/user-attachments/assets/e0f74560-3197-4658-8300-f7434c23e497" />
3. Linux:
   1. Linux may only use the web version to sideload. Check step 2.4.
4. Web:
   1. Open [Word on the web](https://word.cloud.microsoft/?wdOrigin=OFFICECOM-WEB.APPGALLERY)
   2. Select Home -> Add-ins, then select More Settings.
   3. On the Office Add-ins dialog -> select Upload My Add-in.
   4. Browse to the add-in manifest file -> Select Upload.

### 3. FAQ
1. I can't refresh the FFT, What should I do?
   Try to clean cache in Word then restart the Word and refresh Add-in.
   1. Win + R -> `%LOCALAPPDATA%`
   2. Direct to `\Microsoft\Office\16.0\Wef`
   3. Delete the cache files folder <img width="1089" height="440" alt="Screenshot 2025-10-13 154944" src="https://github.com/user-attachments/assets/1d1a5add-fbb3-436e-8808-0291a3bc510a" />
