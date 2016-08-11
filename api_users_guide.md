# CTM API Getting Started

The goal of this example is to show you how to use the API to setup more advanced routing. Using this example, you can direct callers to a voice menu during business hours or to a different voice menu to leave a message (voicemail) after hours. The business hours voice menu will be a simple example of "Press 1 for sales, 2 for support, or 3 to leave a message."

Although you can use the API to update routes, schedules, and voice menus, we will be creating the routing objects in a logical order based on dependenices.

## API Keys

For the following examples, you will need either agency or account API keys and secrets. You can obtain these by logging into your account and going to either Settings > Agency Settings or Settings > Account Settings. This guide will create and use a sub-account, so it is recommended to use agency keys as you follow the examples.

> NOTE: Agency keys can be used across all accounts. Account keys are specific to an account and will be limited to making API requests against that account.

You can put your keys in your environment and then you should be able to copy and paste the examples.

```bash
export CTM_API_HOST='api.calltrackingmetrics.com'
export ACCESS_KEY='...'
export SECRET_KEY='...'
```

## Basic Auth

For curl, you can use -u ${ACCESS_KEY}:${SECRET_KEY} or an authorization header. This documentation will use -u for simplicity.

## Headers

You want to send the `application/json` content type header with each request.

```bash
--header 'content-type: application/json'
```

## Sub-Accounts

Before purchasing phone numbers, you will want to create a sub-account. This step requires agency API keys.

```bash
curl --request POST \
  --url https://${CTM_API_HOST}/api/v1/accounts \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --header 'content-type: application/json' \
  --data   '{"account": { "name": "My First Account", "timezone_hint":"America/Los_Angeles"}, "billing_type":"existing"}'
```

If successful, you will now have an account id for the new sub-account (in this case named "My First Account.") The response will be similar to this; specifically, the account id will differ.

```json
{
  "status": "success",
  "id": 18647,
  "name": "My First Account"
}
```

If you export the account id, you can use it to copy and paste the following examples.

```bash
export ACCOUNT_ID=id_from_response
```
  
## Purchase Phone Numbers

Purchasing phone numbers is a two-step process. First, you search for numbers. Second, you purchase the desired numbers. There are several ways to search for phone numbers (which will be used as tracking numbers). You can search by area code, address (city, state, and/or zip code), and you can include a pattern as well (for example, the NXX or prefix of a phone number.)

> NOTE: Searching for US/CA numbers has more options than international searching.

### Area Code Search

To perform an area code search, you need to specify the `country`, set `searchby` to `area`, and set `areacode` to the desired area code.

```bash
curl --request GET \
  --url 'https://'${CTM_API_HOST}'/api/v1/accounts/'${ACCOUNT_ID}'/numbers/search.json?country=US&searchby=area&areacode=443' \
  --header 'content-type: application/json'
```

If you want to add a pattern (for example, in order to search for a prefix), you can add a `pattern` to search for.

```bash
curl --request GET \
  --url 'https://'${CTM_API_HOST}'/api/v1/accounts/'${ACCOUNT_ID}'/numbers/search.json?country=US&searchby=area&areacode=443&pattern=341' \
  --header 'content-type: application/json'
```

### Zip Code Search

To perform a zip code search, set `searchby` to `address` and set `address` to the desired zip code. The list of results may include nearby zip codes if there are not enough numbers available in the specified area.

```bash
curl --request GET \
  --url 'https://'${CTM_API_HOST}'/api/v1/accounts/'${ACCOUNT_ID}'/numbers/search.json?country=US&searchby=address&address=21401' \
  --header 'content-type: application/json'
```

### Address Search

You can also specify a street address or a city and state as the `address`.

```bash
curl --request GET \
  --url 'https://'${CTM_API_HOST}'/api/v1/accounts/'${ACCOUNT_ID}'/numbers/search.json?country=US&searchby=address&address=beverly%20hills%2C%20ca' \
  --header 'content-type: application/json'
```

> NOTE: Make sure to properly encode your parameters. For example, if you do not use %20 or a `+` as the space character in the above url, the search will fail.

### Search Results

No matter the type of search, if numbers are found that match, you will receive a response similar to the following:

```json
{
  "numbers": [
    {
      "source": 1,
      "friendly_name": "(424) 332-5093",
      "latitude": "34.073600",
      "longitude": "-118.400400",
      "lata": "730",
      "region": "CA",
      "postal_code": "90209",
      "iso_country": "US",
      "capabilities": {
        "voice": true,
        "SMS": true,
        "MMS": true
      },
      "number": "+14243325093",
      "phone_number": "+14243325093",
      "number_type": "local",
      "addr_required": "none",
      "ratecenter": "Los Angeles",
      "distance": 0
    },
    {
      "source": 1,
      "friendly_name": "(424) 332-5173",
      "latitude": "34.073600",
      "longitude": "-118.400400",
      "lata": "730",
      "region": "CA",
      "postal_code": "90209",
      "iso_country": "US",
      "capabilities": {
        "voice": true,
        "SMS": true,
        "MMS": true
      },
      "number": "+14243325173",
      "phone_number": "+14243325173",
      "number_type": "local",
      "addr_required": "none",
      "ratecenter": "Los Angeles",
      "distance": 0
    },
    ...
```

You can use either the `number` or `phone_number` field from the desired tracking number to make the purchase.

If no numbers are found, you will receive an empty list of numbers. However, there will be suggestions for overlays if they are available/known. The following is the result of a search for area code 212 (New York, NY):

```json
{
  "numbers": [],
  "country": "US",
  "searchby": "area",
  "error": [],
  "format_style": "v2",
  "include_distance": false,
  "contains": "212",
  "areacode": "212",
  "status": "success",
  "overlays": [
    "646",
    "917"
  ]
}
```

If desired, a similar search can be repeated for area code 646 or 917.

### Purchasing Desired Numbers

Once you have a phone number you would like to purchase, you just need to POST a request to the numbers endpoint.

```bash
curl --request POST \
  --url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/numbers \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --header 'content-type: application/json' \
  --data '{"phone_number": "+14105551212","test": true}'
```

> NOTE: If you set `test`, the number will be added to your account. However, the number will only be a "test" number that can be used in other API calls. You will not be charged for the simulated purchase, and the number will not be able to send or receive phone calls or SMS. To purchase a live number, omit the `test` field.

If the purchase is successful (i.e. the number was still available and the account had sufficient funds), you will receive a reponse similar to the following:

```json
{
  "status": "success",
  "number": {
    "id": "TPNC3C4B23C348AEC2EE54EFD301979CD2EFB6E4F483E702C7FE233AC89EB9A978D",
    "filter_id": 67333,
    "name": null,
    "active": true,
    "status": "active",
    "account_id": "18600",
    "source": null,
    "number": "+14105551212",
    "call_setting": null,
    "country_code": "1",
    "next_billing_date": null,
    "purchased_time": "2016-08-09T16:09:32Z",
    "route_to": {
      "type": "receiving_number",
      "multi": true,
      "mode": "simultaneous",
      "dial": []
    },
    "split": [
      "1",
      "410",
      "555",
      "1212"
    ],
    "stats": {
      "since": 1470758972.4781718,
      "renewal_costs": "1.5",
      "calls": 0,
      "minutes": "0",
      "minute_costs": "0.0"
    },
    "formatted": "(410) 555-1212",
    "routing": "simultaneous",
    "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18600/numbers/TPNC3C4B23C348AEC2EE54EFD301979CD2EFB6E4F483E702C7FE233AC89EB9A978D.json"
  }
}
```

Within the returned number object, we will be using the id (TPN...) for the following requests.

```bash
export TPN_ID=tpn_id_from_response
```

## Setup Routing for Phone Numbers

In order to forward the newly purchased tracking number to a receiving number, we will update the tracking number using a PUT request.

```bash
curl --request POST \
  --url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/numbers/${TPN_ID}/update_number \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --header 'content-type: application/json' \
  --data '{"dial_route": "number","numbers": ["3105552222","4165553333"],"country_codes": ["US","CA"]}'
```

For the body of the request, you send an array of numbers without country codes and an array of country codes (ISO-2 alpha codes) that align with the numbers.

```json
{
    "dial_route": "number",
    "numbers": ["3105552222","4165553333"],
    "country_codes": ["US","CA"]
}
```

In the above request, we specified one US number and one CA number.

The response will be something like the following:

```json
{
  "status": "success",
  "number": {
    "id": "TPNC3C4B23C348AEC2EE54EFD301979CD2EFB6E4F483E702C7FE233AC89EB9A978D",
    "filter_id": 67333,
    "name": null,
    "active": true,
    "status": "active",
    "account_id": "18600",
    "source": null,
    "number": "+14102051558",
    "call_setting": {
      "id": "NCF5B75DB2863BDEA0F8D82E7B25F78F3AEA94ED6FAB1EDB7AF11E25FEA2FCEE80F",
      "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18600/call_settings/NCF5B75DB2863BDEA0F8D82E7B25F78F3AEA94ED6FAB1EDB7AF11E25FEA2FCEE80F",
      "name": "Account Level"
    },
    "country_code": "1",
    "next_billing_date": null,
    "purchased_time": "2016-08-09T16:09:32Z",
    "route_to": {
      "type": "receiving_number",
      "multi": true,
      "mode": "simultaneous",
      "dial": [
        {
          "id": "RPN34D8AC3F61E8848FEA641CEF711011AADF51559F51DBA0B436FA9716588BDE93",
          "filter_id": 40184,
          "name": null,
          "number": "+13105552222",
          "display_number": "(310) 555-2222",
          "account_id": 18600,
          "country_code": "1",
          "split": [
            "1",
            "310",
            "555",
            "2222"
          ],
          "formatted": "(310)-555-2222",
          "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18600/receiving_numbers/RPN34D8AC3F61E8848FEA641CEF711011AADF51559F51DBA0B436FA9716588BDE93"
        },
        {
          "id": "RPN34D8AC3F61E8848FEA641CEF711011AADF51559F51DBA0B4896CD430F1FC1D84",
          "filter_id": 40185,
          "name": null,
          "number": "+14165553333",
          "display_number": "(416) 555-3333",
          "account_id": 18600,
          "country_code": "1",
          "split": [
            "1",
            "416",
            "555",
            "3333"
          ],
          "formatted": "(416)-555-3333",
          "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18600/receiving_numbers/RPN34D8AC3F61E8848FEA641CEF711011AADF51559F51DBA0B4896CD430F1FC1D84"
        }
      ]
    },
    "split": [
      "1",
      "410",
      "205",
      "1558"
    ],
    "stats": {
      "since": 1470758972.4781718,
      "renewal_costs": "1.5",
      "calls": 0,
      "minutes": "0",
      "minute_costs": "0.0"
    },
    "formatted": "(410) 205-1558 (x67333)",
    "routing": "simultaneous",
    "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18600/numbers/TPNC3C4B23C348AEC2EE54EFD301979CD2EFB6E4F483E702C7FE233AC89EB9A978D.json"
  }
}
```

Within the returned number object, we will be using each id (RPN...) for the following requests.

```bash
export RPN_ID_US=first_rpn_id_from_response
export RPN_ID_CA=second_rpn_id_from_response
```

### Tracking Source

We will create a Google AdWords tracking source and add a tracking number to the new tracking source.

```bash
curl --request POST \
--url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/sources \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --header 'content-type: application/json' \
  --data '{"name": "Google Adwords","online": "1","referring_url": "", "landing_url": "gclid=.+", "position": "1"}'
```

The output will look something like the following:

```json
{
  "status": "success",
  "source": {
    "id": "TSOF6FC2D2594D22C52C7F22FA76C7AEAFA11100A5A494E400EAD97ABA6AD0276CD",
    "name": "Google Adwords",
    "account_id": 18649,
    "referring_url": "",
    "not_referrer_url": null,
    "landing_url": "gclid=.+",
    "not_landing_url": null,
    "position": 1,
    "online": true,
    "crm_tag": "",
    "geo_mode": "off",
    "geo_sources": "https://${CTM_API_HOST}/api/v1/accounts/18649/sources/TSOF6FC2D2594D22C52C7F22FA76C7AEAFA11100A5A494E400EAD97ABA6AD0276CD/geo_sources.json",
    "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18649/sources/TSOF6FC2D2594D22C52C7F22FA76C7AEAFA11100A5A494E400EAD97ABA6AD0276CD.json"
  }
}
```

Capture the TSO id from the response for the next request.

```bash
export TSO_ID=tso_id_from_reponse
```

```bash
curl --request POST \
  --url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/sources/${TSO_ID}/numbers/${TPN_ID}/add \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --header 'content-type: application/json'
```

The response will be similar to the following:

```json
{
  "status": "success",
  "source": {
    "id": "TSOF6FC2D2594D22C52C7F22FA76C7AEAFA11100A5A494E400EAD97ABA6AD0276CD",
    "name": "Google Adwords",
    "account_id": 18649,
    "referring_url": "",
    "not_referrer_url": null,
    "landing_url": "gclid=.+",
    "not_landing_url": null,
    "position": 1,
    "online": true,
    "crm_tag": "",
    "geo_mode": "off",
    "geo_sources": "https://${CTM_API_HOST}/api/v1/accounts/18649/sources/TSOF6FC2D2594D22C52C7F22FA76C7AEAFA11100A5A494E400EAD97ABA6AD0276CD/geo_sources.json",
    "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18649/sources/TSOF6FC2D2594D22C52C7F22FA76C7AEAFA11100A5A494E400EAD97ABA6AD0276CD.json"
  }
}
```

The tracking number is now assigned to the Google Adwords tracking source. You can see the tracking number settings which includes the tracking source information using the details for tracking number request:

```bash
curl --request GET \
  --url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/numbers/${TPN_ID} \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --header 'content-type: application/json'
```

In the response, you can see the source:

```json
{
  "id": "TPNC3C4B23C348AEC2EE54EFD301979CD2EDEC86735B9EBFF398FA80FD85CAAB7B3",
  "filter_id": 67334,
  "name": null,
  "active": true,
  "status": "active",
  "account_id": "18649",
  "source": {
    "id": "TSOF6FC2D2594D22C52C7F22FA76C7AEAFA11100A5A494E400EAD97ABA6AD0276CD",
    "name": "Google Adwords",
    "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18649/sources/TSOF6FC2D2594D22C52C7F22FA76C7AEAFA11100A5A494E400EAD97ABA6AD0276CD"
  },
  ...
```


### Schedules

We want to create two schedules. The first will be called Business Hours and will be for Monday through Friday, 9:00AM to 12:00PM, and 1:00PM to 5:00PM. The second will be called Weekend Hours and will be for Saturday and Sunday, 9:00AM to 12:00PM.

To create a schedule we will POST to the schedules endpoint. Note that start_time and end_time are measured in minutes past midnight; 540 corresponds to 9:00AM, 720 corresponds to 12:00PM, etc.

```bash
curl --request POST \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/schedules \
  --header 'content-type: application/json' \
  --data '{"schedule":{"name":"Business Hours", "times":[{"start_time":"540",
  "days":{"sun":false,"mon":true,"tue":true,"wed":true,"thu":true,"fri":true,"sat":false},
  "end_time":"720","position":"0"},{"start_time":"780",
  "days":{"sun":false,"mon":true,"tue":true,"wed":true,"thu":true,"fri":true,"sat":false},
  "end_time":"1020","position":"1"}],"timezone":"Pacific Time (US & Canada)"}}'
```

```bash
curl --request POST \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/schedules \
  --header 'content-type: application/json' \
  --data '{"schedule":{"name":"Weekend Hours","times":[{"position":0,"start_time":540,
  "start_day":"sun","end_time":720,"end_day":"sat","days":{"sun":true,"mon":false,
  "tue":false,"wed":false,"thu":false,"fri":false,"sat":true}}],
  "timezone":"Pacific Time (US & Canada)"}}'
```

The responses will be similar to the following:

```json
{
  "schedule": {
    "name": "Business Hours",
    "timezone": "Pacific Time (US & Canada)",
    "times": [
      {
        "start_time": "540",
        "days": {
          "sun": false,
          "mon": true,
          "tue": true,
          "wed": true,
          "thu": true,
          "fri": true,
          "sat": false
        },
        "end_time": "720",
        "position": "0"
      },
      {
        "start_time": "780",
        "days": {
          "sun": false,
          "mon": true,
          "tue": true,
          "wed": true,
          "thu": true,
          "fri": true,
          "sat": false
        },
        "end_time": "1020",
        "position": "1"
      }
    ]
  },
  "position": "1",
  "start": {
    "time": "780"
  },
  "end": {
    "time": "1020"
  }
}
```

We'll use the Business Hours schedule in the voice menu below, so capture the SCH id from the first schedule response.

```bash
export SCH_ID=sch_id_from_business_hours_above
```

### Voice Menus

We are going to create two voice menus. We will setup the leave a message (voicemail) voice menu first because we will be using it in the second voice menu. The second voice menu will be a typical "press 1, press 2" setup called "Business Hours Menu."

Let's create the voicemail menu now:
```json
curl --request POST \
  --url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/voice_menus \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --header 'content-type: application/json' \
  --data '{"voice_menu":{"name":"Voice Mail Box","play_message":"say:man:en:","default_number_id":"",
  "input_maxkeys":"1","input_timeout":"1","prompt_retries":"0","schedule_id":"","after_hours_action_label":"",
  "after_hours_action_id":"","after_hours_action_type":"","items":[{"id":"","keypress":"1",
  "voice_action_type":"message","default_action":"1",
  "play_message":"say:alice:en-US:Please leave a message after the beep",
  "play_beep":"1","recording":"1","timer":"120","transcribe":"0","tag_id":"voicemail","assign_agent_id":""}]}}'
```

The response will be similar to the following:

```json
{
  "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18776/voice_menus/VOM93F974B7707B24678A2C7EC91786D95775D605A0EE64DCDE",
  "id": "VOM93F974B7707B24678A2C7EC91786D95775D605A0EE64DCDE",
  "name": "Voice Mail Box",
  "play_message": "say:man:en:",
  "schedule_id": null,
  "default_number_id": null,
  "after_hours_action_type": "",
  "after_hours_action_id": null,
  "after_hours_action_label": "",
  "prompt_retries": 0,
  "input_maxkeys": 1,
  "input_timeout": 1,
  "items": [
    {
      "item": {
        "id": "VMI93F974B7707B246712BF010BB9EA0A87D1E8BFBC1C8D221477AFCFA1EF40A26C",
        "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18776/voice_menus/VOM93F974B7707B24678A2C7EC91786D95775D605A0EE64DCDE/voice_menu_items/VMI93F974B7707B246712BF010BB9EA0A87D1E8BFBC1C8D221477AFCFA1EF40A26C",
        "keypress": "1",
        "voice_action_type": "message",
        "tag_list": [
          "voicemail"
        ],
        "play_message": "say:alice:en-US:Please leave a message after the beep",
        "play_beep": true,
        "recording": true,
        "transcribe": false,
        "max_recording_time": null,
        "default_action": true,
        "sms_authorized": false
      }
    }
  ]
}
```

Within the returned voicemail voicemenu object, we will be using the id (VOM...) for the following requests.

```bash
export VOICEMAIL_VOM_ID=vom_id_from_response
```

Now let's create the main voice menu:

```bash
curl --request POST \
  --url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/voice_menus \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --header 'content-type: application/json' \
  --data '{"name":"Business Hours Menu",
  "play_message":"say:alice:en-US:Press 1 for sales, 2 for support, or 3 to leave a message",
  "default_number_id":"","input_maxkeys":"1","input_timeout":"7","prompt_retries":"5",
  "schedule_id":"'${SCH_ID}'","after_hours_action_label":"voice_menu",
  "after_hours_action_id":"'${VOICEMAIL_VOM_ID}'","after_hours_action_type":"VoiceMenu",
  "items":[{"keypress":"1","voice_action_type":"dial","default_action":"0",
  "dial_number_id":"'${RPN_ID_US}'","play_message":"say:alice:en-US:",
  "whisper_message":"say:alice:en-US:","tag_id":"","assign_agent_id":""},
  {"keypress":"2","voice_action_type":"dial","default_action":"0",
  "dial_number_id":"'${RPN_ID_CA}'","play_message":"say:alice:en-US:",
  "whisper_message":"say:alice:en-US:","tag_id":"","assign_agent_id":""},
  {"keypress":"3","voice_action_type":"menu","default_action":"1",
  "next_voice_menu_id":"'${VOICEMAIL_VOM_ID}'",
  "whisper_message":"say:alice:en-US:","tag_id":"","assign_agent_id":""}]}'
```

The response will be similar to the following:

```json
{
  "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18776/voice_menus/VOM93F974B7707B24678A2C7EC91786D957DEADF8980ED4CE9E",
  "id": "VOM93F974B7707B24678A2C7EC91786D957DEADF8980ED4CE9E",
  "name": "Business Hours Menu",
  "play_message": "say:alice:en-US:Press 1 for sales, 2 for support, or 3 to leave a message",
  "schedule_id": "SCHC25850ECC8B5AB2021E084F8290B4B50BC4BC3214F7EEB45",
  "default_number_id": null,
  "after_hours_action_type": "VoiceMenu",
  "after_hours_action_id": "VOM93F974B7707B24678A2C7EC91786D9579801CAAE2832BAAE",
  "after_hours_action_label": "voice_menu",
  "prompt_retries": 5,
  "input_maxkeys": 1,
  "input_timeout": 7,
  "items": [
    {
      "item": {
        "id": "VMI93F974B7707B246712BF010BB9EA0A87E01E0FDBFA62A92E4B0DB0C34EB387C7",
        "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18776/voice_menus/VOM93F974B7707B24678A2C7EC91786D957DEADF8980ED4CE9E/voice_menu_items/VMI93F974B7707B246712BF010BB9EA0A87E01E0FDBFA62A92E4B0DB0C34EB387C7",
        "keypress": "1",
        "voice_action_type": "dial",
        "tag_list": [

        ],
        "dial_number_id": "RPN34D8AC3F61E8848FEA641CEF711011AA397FC5F6864F4C1644086B347746A83B",
        "play_message": "say:alice:en-US:",
        "whisper_message": "say:alice:en-US:",
        "default_action": false,
        "sms_authorized": false
      }
    },
    {
      "item": {
        "id": "VMI93F974B7707B246712BF010BB9EA0A87985125D40BCA89FA8350DB01D41C66DB",
        "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18776/voice_menus/VOM93F974B7707B24678A2C7EC91786D957DEADF8980ED4CE9E/voice_menu_items/VMI93F974B7707B246712BF010BB9EA0A87985125D40BCA89FA8350DB01D41C66DB",
        "keypress": "2",
        "voice_action_type": "dial",
        "tag_list": [

        ],
        "dial_number_id": "RPN34D8AC3F61E8848FEA641CEF711011AA397FC5F6864F4C169DACE2C2A21FBF50",
        "play_message": "say:alice:en-US:",
        "whisper_message": "say:alice:en-US:",
        "default_action": false,
        "sms_authorized": false
      }
    },
    {
      "item": {
        "id": "VMI93F974B7707B246712BF010BB9EA0A87D1B7E0AAFC4C9D168EAD14BB8433C361",
        "url": "https://api.calltrackingmetrics.com/api/v1/accounts/18776/voice_menus/VOM93F974B7707B24678A2C7EC91786D957DEADF8980ED4CE9E/voice_menu_items/VMI93F974B7707B246712BF010BB9EA0A87D1B7E0AAFC4C9D168EAD14BB8433C361",
        "keypress": "3",
        "voice_action_type": "menu",
        "tag_list": [

        ],
        "next_voice_menu_id": "VOM...",
        "whisper_message": "say:alice:en-US:",
        "default_action": true,
        "sms_authorized": false
      }
    }
  ]
}
```

Within the returned main voicemenu object, we will be using the id (VOM...) for the following requests.

```bash
export MAIN_VOM_ID=vom_id_from_response
```

## Change the Tracking Number Route

Now that we have the voice menus setup, we can route the tracking number to the primary voice menu.

```bash
curl --request PUT \
  --url https://${CTM_API_HOST}/api/v1/accounts/${ACCOUNT_ID}/numbers/${TPN_ID}/dial_routes \
  -u ${ACCESS_KEY}:${SECRET_KEY} \
  --header 'content-type: application/json' \
  --data '{"virtual_phone_number": {"dial_route":"voice_menu","voice_menu_id":"${MAIN_VOM_ID}"}}'
```

The request body just needs a `dial_route` of `voice_menu` and the `voice_menu_id`:

```json
{
  "virtual_phone_number": {
    "dial_route": "voice_menu",
    "voice_menu_id": "VOM..."
  }
}
```

The response should include a `route_to` key indicating the change:

```json
{
  "status": "success",
  "message": "Settings changed",
  ...
  "route_to": {
    "type": "voice_menu",
    "dial": {
      "id": "VOM...",
      "name": "Business Hours Menu"
    }
    ...
  }
}
```

## Wrap Up

At this point, if you purchased a real tracking number (not a test number), then if you call the tracking number, you should be connected to a voice menu. Keypress 1 would send you to the first receiving number, keypress 2 would send you to the second receiving number, and keypress 3 (or no keypress) will take you to the voicemail menu. Also, if you call after business hours, you would be directed to the voicemail menu directly.
