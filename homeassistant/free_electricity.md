# Octopus Energy Free Electricity Sensor for Home Assistant
Octopus Energy have started a nationwide scheme of offering free electricity slots to customers - they have not (yet!) however added those slots to their API so there is (currently) no way to trigger automations on the back of them. This walkthrough aims to address that using the Home Assistant IMAP integration and a (very) fragile template sensor - see [below](https://github.com/DJBenson/ha-stuff/blob/main/homeassistant/free_electricity.md#caveats) for more on that.

**This is not the entire solution, it simply creates a sensor on which to base your solution which is not covered here as people will have different requirements.**

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

### Configure the template sensor
Copy/paste the below into your editor of choice - if you are using split-configuration then amend as appropriate (i.e. remove the sensor: tag and back-space the rest of the config).
```
sensor:
  - trigger:
      - platform: event
        event_type: "imap_content"
        id: "custom_event"
    sensor:
      - name: octopus_free_electricity
        state: >
          {% set date_string = trigger.event.data['text'] | regex_findall_index('Fill your boots on ([A-Za-z]+ \d+ [A-Za-z]+):') %}
          {% set time_string = trigger.event.data['text'] | regex_findall_index('(\d+)-(\d+)(am|pm)') %}
          {% if date_string and time_string %}
            {% set current_year = now().year %}
            {% set event_datetime = date_string + ' ' + current_year|string + ', ' + time_string[0] + '-' + time_string[1] + time_string[2] %}
          {% else %}
            {% set event_datetime = 'None' %}
          {% endif %}
          {{ event_datetime }}
        attributes:
          event_date: >
            {% set date_string = trigger.event.data['text'] | regex_findall_index('Fill your boots on ([A-Za-z]+ \d+ [A-Za-z]+):') %}        
            {% if date_string %}
              {% set current_year = now().year %}
              {% set date_object = strptime(date_string ~ ' ' ~ current_year|string, '%A %d %B %Y') %}
              {{ date_object.strftime('%Y-%m-%d') }}
            {% else %}
              None
            {% endif %}
            {{ event_date }}
          event_start_time: >
            {% set time_string = trigger.event.data['text'] | regex_findall_index('(\d+)-(\d+)(am|pm)') %}
            {% if time_string %}
              {% set start_hour = time_string[0] | int %}
              {% set period = time_string[2] %}
              {% if period == 'pm' and start_hour < 12 %}
                {% set start_hour_24 = start_hour + 12 %}
              {% elif period == 'am' and start_hour == 12 %}
                {% set start_hour_24 = 0 %}
              {% else %}
                {% set start_hour_24 = start_hour %}
              {% endif %}
              {% set event_starttime = '%02d:00' | format(start_hour_24) %}
            {% else %}
              {% set event_starttime = 'None' %}
            {% endif %}
            {{ event_starttime }}
          event_end_time: >
            {% set time_string = trigger.event.data['text'] | regex_findall_index('(\d+)-(\d+)(am|pm)') %}
            {% if time_string %}
              {% set end_hour = time_string[1] | int %}
              {% set period = time_string[2] %}
              {% if period == 'pm' and end_hour < 12 %}
                {% set end_hour_24 = end_hour + 12 %}
              {% elif period == 'am' and end_hour == 12 %}
                {% set end_hour_24 = 0 %}
              {% else %}
                {% set end_hour_24 = end_hour %}
              {% endif %}
              {% set event_endtime = '%02d:00' | format(end_hour_24) %}
            {% else %}
              {% set event_endtime = 'None' %}
            {% endif %}
            {{ event_endtime }}
          event_start_datetime: >
            {% set date_string = trigger.event.data['text'] | regex_findall_index('Fill your boots on ([A-Za-z]+ \d+ [A-Za-z]+):') %}
            {% set time_string = trigger.event.data['text'] | regex_findall_index('(\d+)-(\d+)(am|pm)') %}
            {% if date_string and time_string %}
              {% set start_hour = time_string[0] | int %}
              {% set period = time_string[2] %}
              {% set start_hour_24 = start_hour + 12 if period == 'pm' and start_hour != 12 else start_hour %}
              {% set start_hour_24 = 0 if period == 'am' and start_hour == 12 else start_hour_24 %}
              {% set current_year = now().year %}
              {% set date_object = strptime(date_string + ' ' + current_year|string, '%A %d %B %Y') %}
              {% set event_start_datetime = date_object.replace(hour=start_hour_24, minute=0).strftime('%Y-%m-%d %H:%M') %}
            {% else %}
              {% set event_start_datetime = 'None' %}
            {% endif %}
            {{ event_start_datetime }}
          event_end_datetime: >
            {% set date_string = trigger.event.data['text'] | regex_findall_index('Fill your boots on ([A-Za-z]+ \d+ [A-Za-z]+):') %}
            {% set time_string = trigger.event.data['text'] | regex_findall_index('(\d+)-(\d+)(am|pm)') %}
            {% if date_string and time_string %}
              {% set start_hour = time_string[0] | int %}
              {% set end_hour = time_string[1] | int %}
              {% set period = time_string[2] %}
              {% set current_year = now().year %}
              {% set end_hour_24 = end_hour + 12 if period == 'pm' and end_hour != 12 else end_hour %}
              {% set end_hour_24 = end_hour_24 if period == 'am' and end_hour != 12 else (0 if period == 'am' and end_hour == 12 else end_hour_24) %}
              {% set date_object = strptime(date_string ~ ' ' ~ current_year|string, '%A %d %B %Y') %}
              {% set event_end_datetime = date_object.replace(hour=end_hour_24, minute=0).strftime('%Y-%m-%d %H:%M') %}
            {% else %}
              {% set event_end_datetime = 'None' %}
            {% endif %}
            {{ event_end_datetime }}
```

In the developer tools, run a check on the config and by clicking the "Check Configuration" button - it should check out if you've done the above correctly but if not fix whatever issue it is throwing.

Scroll down and reload the template sensors by clicking the "Template Entities" link.

You should now have a sensor named sensor.octopus_free_electricity - at this point it's probably unavailable - this is fine - it's not been triggered yet! The sensor has (or will have once it's triggered) a state of the day and time of the next event - this is mostly for display purposes, whilst the attributes contain the good stuff. I've included several so you can pick and choose what to use - the attribute names are fairly self-explanatory;

![image](https://github.com/user-attachments/assets/23409a53-e5ae-42dd-b3a0-219d7ca52bd3)

## How to test

* Navigate to the developer tools section
* Click on the "Events" tab
* Set the "Event type" to "imap_content"
* Fill in the event data as shown below
* Click the "Fire Event" button

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

If any of that changes the sensor will not trigger, it should be fairly easy to fix and I will pick up fixes as I spot them.
