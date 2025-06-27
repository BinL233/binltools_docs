## Introduction
Doc-modifier is a Microsoft Word add-in. This program specifically developed to assist with the formatting requirements of FDA submission documents.

## Get Started
### 0. Prerequisites
1. A machine (iPad, desktop, laptop, VM...) 🙃
2. Microsoft Word (2016 or later) / Microsoft Word Web

### 1. Download the Manifest
The manifest file is required to access the service. Click [here](https://www.binltools.com/api/download/manifest_binltools.xml) to download.

### 2. Sideload the Manifest
Choose the OS you use and finish the sideloading:
1. Mac / iPad:
   1. Use Finder to sideload the manifest file. Open Finder and then enter `Cmd+Shift+G` to open the Go to folder dialog.
   2. Enter `/Users/<username>/Library/Containers/com.microsoft.Word/Data/Documents/wef`. If the wef folder doesn't exist on your computer, create it.
2. Windows:
   1. Share the folder is the simplest way to sideload the manifest. Check [this page](https://learn.microsoft.com/en-us/office/dev/add-ins/testing/create-a-network-shared-folder-catalog-for-task-pane-and-content-add-ins).
3. Linux:
   1. Linux may only use the web version to sideload. Check step 2.4.
4. Web:
   1. Open [Word on the web](https://word.cloud.microsoft/?wdOrigin=OFFICECOM-WEB.APPGALLERY)
   2. Select Home > Add-ins, then select More Settings.
   3. On the Office Add-ins dialog, select Upload My Add-in.
   4. Browse to the add-in manifest file, and then select Upload.