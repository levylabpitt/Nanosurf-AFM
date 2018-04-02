# Control Program for the NanoSurf EasyScan 2 Microscope

This repository is the home for all of the software relating to the NanoSurf EasyScan 2 Microscope, also known as the SuitCase AFM. For the rest of this README file it will be referred to as **The Microscope**, or **NSES2**.

---
## Before You Start

### Mandatory External Software

Before you can use this software, you will need to download The Microscope's software, which can be found [HERE](https://www.nanosurf.com/en/software/easyscan-2). This will allow you to run the COM automation (essentially use The Microscope's API) from this program. 

### Access Codes

You will also need to provide the access codes in The Microscope software. This is explained in the user manual of The Microscope, so I will not go into detail here. The access codes for the microscope can be found on ASANA or in the NSES2 notebook on OneNote.

---

## COM Automation Flow

The Microscope control software is by reference, and you must destroy references in the reverse order that you make them. In order to do this, I have found that the simplist way to go about this is to have the **Application Reference** be the *only* reference open in the Main Program. The **Application Reference** is opened at the start of the program, and then closed at the end of the program. All other references, which are derived from this **Application Reference** are to be opened inside of SubVIs and destroyed at the end of the execution of the SubVI. This makes it easier to keep track of what order references were opened and thus what order they need to be closed in. 

---

## OneNote Flow

### REST APIs

The OneNote HTTPS API follows the **REST** convention. More information on the basic architecture of REST APIs, from the creators dissertation on the subject can be found [here.](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) Also a good guide for REST APIs can be found [here.](https://restfulapi.net/)

### Authentication Procedure

I am currently trying handle all of the authentication from a single VI. This lets us have a single state in the JKI state machine which handles all refresh/Access tokens and Authorization Codes. Reading and writing from the files that these are stored is also handled within this subVI. 

In the future I am planning on adding a part that will let you get an Authorization code from this VI also, but am currently considering the cases in which this would be needed. (I have encountered this in the past, where I needed to get a new Authorization Code, but cannot currently recall when that would be.)

### Creating a New Page

When creating a new page, I will be recording all of the settings of the microscope to OneNote, listed in a table using the HTML tags that are allowed. I plan on also having a method that will allow you to send up another table using a PATCH method if you chose to change your settings. 

I believe that once you create a page, the resource URL will be send down in the response headers. I will still need to test this, but all developement at this point should assume that there is a way to programmatically get this value back. According to the [OneNote MSDN Documentation on creating pages](https://msdn.microsoft.com/en-us/office/office365/howto/onenote-create-page) the resource URL will be in the ***location*** response header.

### Adding Content to a Page

Once you have created a page, you can then send images with text up to OneNote. I have created a VI (though still in heavy development) that allows users to highlight text and then place the needed tags for formatting around this highlighted text in order to make their lives easiers. This VI also displays the image they are sending up to onenote.

Images sent up to the API in a PATCH request should be BASE64 Encoded and sent up in the raw HTML as part of the commands section. Placing them in their own block of a multipart request will *not* work.

---

## Files and Paths

### Paths
I have created a VI that will automatically create a **PATH** for all of the files that will be saved here. As of now, there is 3 paths that are created, one for the BMP file that will be saved, one for the BMP that is modified to work with the lithography program, and a path for the InkScape file. In the future I plan on creating a path for .NID files also.
**Paths are arranged as follows**
1. There is a folder that is created in the base path according to the date that the path is requested.
2. In this folder there are 3 subfolders created, One for BMP files, One for the modified BMP files for the lithography, and One for the InkScape files.
3. Every time you save a BMP file, the program should automatically create a BMP for the lithography and an Inkscape file that is already presized and ready to be used. They will be incremented (zero indexed), and named based on the date.
