# Post to Chat for Engage with xMatters
This is an add-on for the [ServiceNow integration](https://store.servicenow.com/sn_appstore_store.do#!/store/application/5950d7444f2231000e9fa88ca310c78c/4.0.2) to add a `Post to chat` checkbox to the Engage with xMatters form. When checked, this can create a room in the configured chat application. 

<kbd>
  <img src="/media/EngageScreenshot.png" width="575">
</kbd>

---------

<kbd>
  <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
</kbd>


---------


# Pre-Requisites
* ServiceNow account
* [Version 4.0.2](https://store.servicenow.com/sn_appstore_store.do#!/store/application/5950d7444f2231000e9fa88ca310c78c/4.0.2) or higher of the ServiceNow integration
* [Slack shared library](https://github.com/xmatters/xm-labs-slack) - Shared library for interacting with Slack. 
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
1. Login as a developer and navigate to the Developer tab > ServiceNow 4.0 Comm Plan.
2. Install the [Slack](https://github.com/xmatters/xm-labs-slack/) shared library. For other chat applications, add the necessary code to create the room and post a message. **NOTE**: Make sure to create the Slack endpoint!
3. Navigate to the Properties and add two new properties:

| Property Name  | Property Type | Description | 
| ------------- | ------------- | ----------- |
| `chat_link_disp`  | Text  | This holds the css value to display or hide the chat link in the case there is no chat room associated. |
| `chat_link`  | Text  | This holds a link to the chat room. |

4. Add both properties to the Enage with xMatters form and Conference Bridge layouts. 
2. Head to the Integration Builder and expand the Inbound Integrations. Click on the `Engage with xMatters` script. 
3. After the `trigger = JSON.parse(request.body);` line, paste in one of the following code snippet.
```javascript
if( trigger.task_chat ) {
    
    // Create the channel using the Incident Number as the name. Then get the Team details. 
    var channel = Slack.createChannel( trigger.properties.number.toLowerCase() );
    var team    = Slack.getTeam();

    // Build a message to display to users entering the room. 
    var text = "ServiceNow Incident <" + trigger.properties.servicenowurl + '/nav_to.do?uri=incident.do?sys_id=' + trigger.properties.sys_id + '|' + trigger.properties.number + '>: ' + trigger.properties.short_description;
    var payload = { 
      "channel": "#" + channel.channel.name, 
      "username": "xmatters",
      "icon_url": "https://www.xmatters.com/wp-content/uploads/2016/12/xmatters-x-logo.png", 
      "text": text
    };

    // Post the message to Slack, then set the chat_link properties so the notifications contain the Slack room link. 
    Slack.postMessage( payload );
    trigger.properties.chat_link = 'https://' + team.name + '.slack.com/messages/' + trigger.properties.number;
    trigger.properties.chat_link_disp = "visible";

}
```

4. Navigate to the `Engage with xMatters` form and click Messages. Click Edit next to Email / Fax / Push. 
5. Drag the `chat_link` and `chat_link_disp` properties onto the canvas, near the bottom. Note what they are next to. 
6. Click Show Source and scroll to the bottom. Add the following code:

```html
<tr style="border: solid 1px #e4e4e4; display: ${chat_link_disp UUID HERE}">
   <td style="padding: 5px 10px;" tabindex="0"><a href="${chat_link UUID HERE}">Join Chat</a></td>
</tr>
```
Then replace the `chat_link_disp UUID here` and the `chat_link UUID HERE` values with the UUIDs of the properties respectively. 


# Testing
View an existing Incident and click the Engage with xMatters button. In the dialog displayed, enter the required information and mark the `Post to Chat?` checkbox. Click Submit. 

The notifications will be sent to the targeted recipients and a chat room will be created with a name matching the Incident number. 

# Troubleshooting
After submitting a new Enage with xMatters request, inspect the Activity Stream for either the Engage with xMatters or the Conference Bridge, depending on the value of the Conference Bridge drop down. The activity stream will have the value of the `chat` checkbox and any subsequent errors related to creating the chat room. 
