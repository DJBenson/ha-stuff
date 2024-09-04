# Octopus Energy Free Electricity Solution for Home Assistant
Octopus Energy have started a nationwide scheme of offering free electricity slots to customers - they have not (yet!) however added those slots to their API so there is (currently) no way to trigger automations on the back of them. This walkthrough aims to address that using the Home Assistant IMAP integration and a (very) fragile collection of input_datetime fields - see [below](https://github.com/DJBenson/ha-stuff/blob/main/homeassistant/free_electricity.md#caveats) for more on that.

**This is not the entire solution, it simply creates two input_datetimes (one for the start of the event and one for the end) on which to base your solution which is not covered here as people will have different requirements.**

## Pre-requistes
* Home Assistant
* An email account which supports IMAP to which your notifications for the free energy sessions are delivered. Of course this also means you should be signed up to said emails.
* A text editor to create the template sensor

## How to configure

### Configure the IMAP integration

I won't go into great depth on this bit as [this](https://www.home-assistant.io/integrations/imap/) page on the Home Assistant site has most of what's required - but there are some specific steps for this to work so I'll set those out below. If you are using GMail then ensure you follow the specific steps on that page. The same may also apply to iCloud if you're using iCloud Mail.

* Go to Settings > Integrations and click "Add Integration"
* Search for IMAP
* Fill in the following details, leave the rest alone unless you know what you're doing;
  * Username
  * Password
  * Port
  * In the IMAP search field, enter the following: UnSeen UnDeleted Subject "Free Electricity" FROM "hello@octopus.energy"
  * Ensure "Body text" is checked
* Click "Submit".

You will now have a new entry in your integrations which will pick up emails from Octopus which have the keywords in the subject line.

If your email server is not PUSH-enabled (or it's unreliable) click on "Configure" on your IMAP entity and toggle "Enable Push-IMAP if the server supports it. Turn off if Push-IMAP updates are unreliable" to suit.

### Import the package

I have packaged the solution into a yaml package file which you can simply copy/paste to your config/packages folder. The package can be found [here](https://github.com/DJBenson/ha-stuff/blob/main/homeassistant/packages/octopus_free_electricity.yaml).

Your packages folder should be as per the below. For more on packages see [here](https://www.home-assistant.io/docs/configuration/packages/).

<img width="311" alt="image" src="https://github.com/user-attachments/assets/2ec2f3d2-5454-44a6-a233-fdb034dd9e06">

Once there and you've reloaded the yaml configuration (namely Automation and Input Date Time).

<img width="1053" alt="image" src="https://github.com/user-attachments/assets/7417277f-79b6-4b08-b9e6-60b0f3c0f98e">

You'll be presented with a number of fields;

<img width="486" alt="image" src="https://github.com/user-attachments/assets/6993bb5d-bf0f-4f7f-af13-dfe755d83b2f">

* input_datetime.octopus_free_electricity_start - this holds the start of the event in a datetime object
* input_datetime.octopus_free_electricity_end - this holds the start of the event in a datetime object
* input_text.octopus_free_electricity_regex_date - this holds the regex which selects the date from the email - only change this if Octopus change their email layout
* input_text.octopus_free_electricity_regex_start - this holds the regex which selects the event start time from the email - only change this if Octopus change their email layout
* input_text.octopus_free_electricity_regex_end - this holds the regex which selects the event end time from the email - only change this if Octopus change their email layout

## How to test

* Navigate to the developer tools section
* Click on the "Events" tab
* Set the "Event type" to "imap_content"
* Fill in the event data as shown below
* Click the "Fire Event" button
* Go back and check your sensor - it should have data.

```
entry_id: 01J5RGADKJ8FM0RK9TW7WDVZ3E
server: your.email.server
username: user@domain.com
search: UnSeen UnDeleted Subject "Free Electricity" FROM "hello@octopus.energy"
folder: INBOX
initial: true
date: "2024-09-04T15:48:41+01:00"
sender: hello@octopus.energy
subject: "FYI: Free Electricity tomorrow 1-2pm"
uid: "33234"
text: "snip Fill your boots on Saturday 4 September: 3-4pm. snip"
```

![image](https://github.com/user-attachments/assets/16b39cc5-1b57-49e3-9542-6f068f3f9865)

## Caveats
This entire solution relies on the format of the email Octopus are sending not changing. The following must be true or it will not work;
* The sender must be hello@octopus.energy
* The subject must include the words "Free Electricity"
* The body of the email must include the words "Fill your boots"
* The time period must include a designator (AM/PM)

If any of that changes the sensor will not trigger. I have now exposed regex fields to allow the code to be amended through the UI. If (and when) it does change, I will publish new regex as part of the package.
