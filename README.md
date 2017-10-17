# Post to Chat for Engage with xMatters
This is an add-on for the [ServiceNow integration](https://store.servicenow.com/sn_appstore_store.do#!/store/application/5950d7444f2231000e9fa88ca310c78c/4.0.2) to add a `Post to chat` checkbox to the Engage with xMatters form. When checked, this can create a room in the configured chat application. 

<kbd>
  <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
</kbd>


<kbd>
  <img src="/media/EngageScreenshot.png">
</kbd>

# Pre-Requisites
* ServiceNow account
* Version 4.0.2 or higher of the ServiceNow integration
* xMatters account - If you don't have one, [get one](https://www.xmatters.com)!

# Files
* [ServiceNow_Add_Chat.xml](ServiceNow_Add_Chat.xml) - The update set to add the checkbox. 


# How it works
A checkbox is added to the Engage with xMatters form. The value of this checkbox is passed from client side form to server side via the ajax call, which is then added to the Engage with xMatters record. The `xMatters Task` business rule fires when this record is inserted and a payload is built, including the checkbox value and sent off to the xMatters Inbound Integration script. Either the `Engage with xMatters` or the `Conference Bridge` integration scripts are invoked, depending on the value of the Conference Bridge drop down passed through. 
In the inbound script, code determines the value of the chat checkbox and then triggers the api requests to create a room in the configured chat application. 

# Installation

## ServiceNow set up
1. Login to the ServiceNow instance as an administrator. Navigate to `Retrieved Update Sets` and click the `Import Update Set from XML` link. 

<kbd>
  <img src="/media/ImportUpdateSet.png">
</kbd>

2. Import the [ServiceNow_Add_Chat.xml](ServiceNow_Add_Chat.xml) file. Click Preview, then Commit. 


## xMatters set up

# Testing
View an existing Incident and click the Engage with xMatters button. In the dialog displayed, enter the required information and mark the `Post to Chat?` checkbox. Click Submit. 

The notifications will be sent to the targeted recipients and a chat room will be created with a name matching the Incident number. 

# Troubleshooting
After submitting a new Enage with xMatters request, inspect the Activity Stream for either the Engage with xMatters or the Conference Bridge, depending on the value of the Conference Bridge drop down. The activity stream will have the value of the `chat` checkbox and any subsequent errors related to creating the chat room. 
