
# Description

There are two targets in total, one is the [Intel official map](https://www.ingress.com/intel) and the other is the API used by the game

This article only do the relevant API appreciation, please do not do things that violate the [TOS](https://www.ingress.com/terms)

Already packaged as a library

`pip3 install ingressAPI`

`from ingressAPI import IntelMap`

# Intel Map

## Get API

1. After logging into your Google account, open the intel interface, Chrome open the Network tab
2. Refresh the page to get all xhr requests
! [chrome](https://ww1.sinaimg.cn/large/006tNbRwgw1farvi9y1ymj30qq0tujvn.jpg)
3. These `get*` requests are the ingress API, don't rush, look at them one by one

## Caution

1. HTTP Request Headers have a `x-csrftoken`, please turn to the various white hat tutorials for specific definitions. The value of this parameter is the same as the cookie's csrftoken! [csrftoken-same-cookie](https://ww4.sinaimg.cn/large/006tNbRwgw1farvqqqf7mj30r20vi12i.jpg)
2. HTTP Request Headers have a bunch of parameters starting with `:`, ignore them and it doesn't matter!
3. ingress all APIs are POST JSON data interaction, so HTTP Method and Conten-Type to determine, the following is not repeated
4. throughout the Intel Map API parameters v equivalent to Session, as to how to get, please see the Intel home page source code in the login state at the end, the name of the js contains the v value, that is, the following chart to code the part! [v](http://ww3.sinaimg.cn/large/006tNbRwgw1farw1l250nj31kw0atdkd.jpg)

## getGameScore

Obviously, this API is used to get the score of the whole game

	API URL: https://www.ingress.com/r/getGameScore
	API Request: v,
	API Response: result: [Enl, Res],

**example**:


![getGameScore](https://ww3.sinaimg.cn/large/006tNbRwgw1farw9fbte8j31fc06sgnj.jpg)
![getGameScore](https://ww3.sinaimg.cn/large/006tNbRwgw1farw6qgzgwj30ac05wq37.jpg)


## getRegionScoreDetails

This API is used to get the current score of that area

	API URL: https://www.ingress.com/r/getRegionScoreDetails
	API Request: latE6, lngE6, v,
	API Response: result: {gameScore: [Res, Enl], regionName, regionVertices: [[Res, Enl], ...], scoreHistory: [[id, Enl, Res], ...], timeToEndOfBaseCycleMs, topAgents:[{nick, team}]}

	Specific latE6,lngE6 see below getEntities API



**example**:
![code](http://ww1.sinaimg.cn/large/006tNbRwgw1fasr0bg6ehj31kw0aiwhd.jpg)
![getRegionScoreDetails](https://ww2.sinaimg.cn/large/006tNbRwgw1fasqwpmodij30jy1csaf2.jpg)
![getRegionScoreDetails](https://ww4.sinaimg.cn/large/006tNbRwgw1fasqwqamvwj30k00pywgy.jpg)


## getEntities

This is used to get the Entity, i.e. link, field, portal

	API URL: https://www.ingress.com/r/getEntities
	API Request: tileKeys: [tilename1, tilename2, ...], v,
	API Response: result: {map: {tilename1: {gameEntities: [[guid, time, [type, faction, guid1, Lng1, Lat1, ...]], ...]}, ...}}

Let's explain what each variable means

tileKeys is a string that represents a rectangle, an advanced MBR (minimum cell matrix) with controllable parameters, which is a query parameter for a typical 2D map, in the specific format `zoom_x_y_minlevel_maxlevel_health`, where zoom indicates the level at which the area is zoomed in (which happens to be the cookie in ingress.intelmap.zoom value, point once + or - button, map zoom, value ± 1, on the intel, this worth range is 3-21, the maximum, the less portal displayed, the smaller the range); x, y indicates the current zoom area number, you can refer to [tilenames](http://wiki. openstreetmap.org/wiki/Slippy_map_tilenames)

The last three are simple, minimum, corresponding to Intel on the choice of Level and Health of the three values, that is, minimum po level, maximum po level, maximum health percentage

This is the conversion involving latitude and longitude in tilename and the actual latitude and longitude, refer to [github-iitc](https://github.com/iitc-project/ingress-intel-total-conversion/blob/ 7dc38a89e708318eb94c201d9cc6f2b5e158ab36/code/map_data_calc_tools.js#L159)

The following code performs warp-tile interpolation
![lat-lng](https://ww4.sinaimg.cn/large/006tNbRwgw1farzd3oygpj31kw0q7jx0.jpg)

In addition, the return values of Lng_x_, Lat_x_, are 6 decimal places, and time is the Unix timestamp determined to the millisecond

guid is the identity of each entity, all users, items, links, portals, ... all have a unique identity guid
type is the type of this entity, e: lin (directly followed by 6 elements, two groups of guid latitude and longitude), r: field (followed by a list, three groups of guid latitude and longitude), p: portal (a group of guid latitude and longitude, followed by the level, energy (0.0-100.0), number of feet, po map, po name, [], false, false, null (these 4 do not quite understand what is), update timestamp)
faction stands for camp, E: Enl, R: Res

**example**

![code](https://ww4.sinaimg.cn/large/006tNbRwgw1farzwpfnizj31g20aowh0.jpg)
Too long, post a part
![example](https://ww2.sinaimg.cn/large/006tNbRwgw1farzyfwdnwj31kw1astif.jpg)


## getPortalDetails

Used to get the details of the portal

	API URL: https://www.ingress.com/r/getPortalDetail
	API Request: guid, v
	API Response: result: [type, faction, lng, lat, level, energy, resonator_nums, picurl, name, [], false, false, null, updatetime, [[agentname, modtype, modlevel, {a: value, b: value}],...], [['agentname', resonator_level, energy],...], belongs, ["", "", []]]

The meaning is clear, it is the details of the portal

{a: value, b: value}

The effect of the mod is indicated by the following diagram, which is easy to understand

**example**

![code](http://ww1.sinaimg.cn/large/006tNbRwgw1fas0j4xa0aj31h80amq5m.jpg)
![example1](https://ww1.sinaimg.cn/large/006tNbRwgw1fas0ggoxy3j31fk1c4jz2.jpg)
![example2](https://ww1.sinaimg.cn/large/006tNbRwgw1fas0gkdjpej314c1ccgp5.jpg)
![example3](https://ww3.sinaimg.cn/large/006tNbRwgw1fas0gn6bk8j30hi07m74b.jpg)

## getArtifactPortals

The guess is to see if a new portal is generated

	API URL: https://www.ingress.com/r/getArtifactPortals
	API Request: v
	APi Resonse: Idem

## getPlexts

Used to obtain comm information

	API URL: https://www.ingress.com/r/getPlexts
	API Request: ascendingTimestampOrder, maxLatE6, maxLngE6, minLatE6 , minLngE6, maxTimestampMs, minTimestampMs, tab, v
	API Response: result: [[guid, updatetime, {plext: {categories, plextType, team, text, markup: [#]}}],...]
	According to different categories (1=all, 2=faction), different plextType, markup's list is different, explain the content of #
	SYSTEM_BROADCAST:
		[PLATER: {plain, team}], [TEXT: {plain}], [PORTAL: {name, address, latE6, lngE6, name, plain, team}], [TEXT: {plain}], [TEXT: {plain}], [TEXT: {plain}]
		The last three are the end of the message, such as '-' '28' 'Mus'
		There may also be 2 endings, such as [TEXT] [PORTAL]
		There may also be no message tail, such as the destroyed behavior
	PLAYER_GENERATED:
		[SECURE: {plain}]
		[SENDER: {plain, team}], [TEXT: {plain}] [AT_PLAYER: {plain, team}]

	Obviously [SECURE] is optional
	Each @ will cause [AT_PLAYER] to appear and fill in the text with [TEXT], so the list is not long (I guessed, 2333)

To explain E6, obviously this is 4 points that define a rectangular area, then the captured messages come from this area

ascendingTimestampOrder(Bool) indicates whether the output is in accordance with time asc

![code](https://ww2.sinaimg.cn/large/006tNbRwgw1fas1zzzm12j31kw0hv43f.jpg)
![example](https://ww2.sinaimg.cn/large/006tNbRwgw1fas1yjmqelj31kw0nhq7w.jpg)

## https://www.ingress.com/r/sendPlext

Send a message

	API URL:
	API Request: latE6, lngE6, message, tab, v,
	API Response: result

This API, see above to receive messages on it, note the value of tab is all or faction

## redeemReward

Using passcode

	API URL: https://www.ingress.com/r/redeemReward
	API Request: passcode, v,
	API Response:
		1. error


## sendInviteEmail

Invitation

	API URL: https://www.ingress.com/r/sendInviteEmail
	API Request: inviteeEmailAddress, v
	API Response: result

## End of Part I

[Code](https://github.com/lc4t/ingress-api)

# Game API(iOS)

## Get API

1. Use Charles as a relay, iPhone forward to Charles, then to ss
2. Install HTTPS certificate

## Cautions

1. In the HTTP Request Headers, there is also X-XsrfToken
2. and fixed values User-Agent: Nemesis (gzip), Accept-Encoding: gzip, Accept-Language: the language you set yourself
3. also need Authorization in the HTTP Request Headers
4. Identification.
5. 
6. no special instructions, are POST requests
7. API domain names used: (gstatic)|(upsight-api)|(appspot.com)|(google-analytics.com)
8. google-analytics a moment on a, are GET requests, in the course of the game of this package lazy to write

## Google stats

During this period the User-Agent is Ingress/1.110.0 CFNetwork/808.2.16 Darwin/16.3.0

### Start

When opening the game, the first request is sent to https://www.googleadservices.com/pagead/conversion/999636601/的

Tell us about the parameters

	appversion: 1.110.0
	auto: 1
	bundleid: com.google.ingress
	lat: 1
	muid: n4nISlWfVzY2pH_42u0NMw
	osversion: 10.2
	sdkversion: ct-sdk-i-v3.1.1
	timestamp: 1457613595.596337
	usage_tracking_enabled: 1

Only timestamp is variable

### Config

This is followed by the POST request profile, https://bootstrap.upsight-api.com/config/v1/f1f4c1b2962446b38f72d5a456aac73c/

Note that starting here, the Headers carry an X-US-Ref-Id (which is required when requesting from upsight-api.com, several other API stations use X-XsrfToken), and each time you start the APP value is not allowed

There are many elements to this request.
```json
{
	"sdk.version": "4.0.2",
	"device.manufacturer": "Apple",
	"screen.width": 375,
	"app.token": "f1f4c1b2962446b38f72d5a456aac73c",
	"app.version": "1.110.0",
	"device.limit_ad_tracking": true,
	"screen.height": 667,
	"bundle.hash": *,
	"ids.idfv": *,
	"sdk.build": "+release.91a51d8",
	"request_ts": 1481902230,
	"device.carrier": "中国联通",
	"device.hardware": "iPhone8,1",
	"locale": "zh_CN",
	"app.bundleid": "com.google.ingress",
	"sid": "622357641234265637",
	"location.tz": "+0800",
	"device.connection": "Wifi",
	"device.os_version": "10.2",
	"device.type": "phone",
	"identifiers": "pub",
	"opt_out": false,
	"device.jailbroken": false,
	"screen.scale": 2,
	"device.os": "ios",
	"sessions": [{
		"session_num": 5558,
		"install_ts": 1457609995,
		"session_start": 1481901886,
		"events": [{
			"ts": 1481902230,
			"seq_id": 252628,
			"type": "upsight.config.expired",
			"user_attributes": {
				"days_inactive": -1,
				"hashed_user_id": *,
				"player_approx_lat": 30.76,
				"player_approx_lng": 103.93,
				"language": "zh_CN",
				"faction": "ALIENS",
				"distance_to_portal": -1,
				"agent_level": 13
			}
		}],
		"past_session_time": 2873649
	}]
}
```

You can see here is actually uploading the session backup, device information, APP information

will return a configuration

```json
{
	"errobj": null,
	"response": [{
		"content": {
			"configurationList": [{
				"configuration": {
					"queues": [{
						"max_age": 120,
						"count_network_fail_retries": false,
						"name": "batch",
						"retry_interval": 60,
						"host": "batch.upsight-api.com",
						"max_retry_count": 3,
						"url_fmt": "{protocol}://{host}/batch/{version}/{app_token}/",
						"protocol": "https",
						"max_size": 50
					}, {
						"max_age": 0,
						"count_network_fail_retries": true,
						"name": "immediate",
						"retry_interval": 5,
						"host": "single.upsight-api.com",
						"max_retry_count": 1,
						"url_fmt": "{protocol}://{host}/sdk/{version}/events/{app_token}/",
						"protocol": "https",
						"max_size": 1
					}, {
						"max_age": 0,
						"count_network_fail_retries": true,
						"name": "config",
						"retry_interval": 15,
						"host": "bootstrap.upsight-api.com",
						"max_retry_count": 2,
						"url_fmt": "{protocol}://{host}/config/{version}/{app_token}/",
						"protocol": "https",
						"max_size": 1
					}],
					"identifiers": [{
						"ids": ["ids.idfv", "ids.android_id"],
						"name": "pub"
					}, {
						"ids": ["ids.idfa", "ids.aid"],
						"name": "ad"
					}],
					"route_filters": [{
						"filter": "*",
						"queues": ["batch", "trash"]
					}, {
						"filter": "upsight.*",
						"queues": ["batch", "trash"]
					}, {
						"filter": "upsight.milestone",
						"queues": ["immediate", "batch", "trash"]
					}, {
						"filter": "upsight.uxm.enumerate",
						"queues": ["immediate", "batch", "trash"]
					}, {
						"filter": "upsight.content.unrendered",
						"queues": ["immediate", "batch", "trash"]
					}, {
						"filter": "upsight.campaign.*",
						"queues": ["immediate", "batch", "trash"]
					}, {
						"filter": "upsight.content.*",
						"queues": ["immediate", "batch", "trash"]
					}, {
						"filter": "upsight.comm.register",
						"queues": ["immediate", "batch", "trash"]
					}, {
						"filter": "upsight.comm.unregister",
						"queues": ["immediate", "batch", "trash"]
					}, {
						"filter": "upsight.session.start",
						"queues": ["immediate", "batch", "trash"]
					}, {
						"filter": "pub.RequestContentSize.*",
						"queues": ["trash"]
					}, {
						"filter": "pub.RequestDuration.*",
						"queues": ["trash"]
					}, {
						"filter": "pub.RequestPayloadSize.*",
						"queues": ["trash"]
					}, {
						"filter": "upsight.config.expired",
						"queues": ["config", "trash"]
					}, {
						"filter": "upsight.monetization.iap",
						"queues": ["immediate", "batch", "trash"]
					}, {
						"filter": "upsight.data_collection",
						"queues": ["immediate", "batch", "trash"]
					}],
					"identifier_filters": [{
						"filter": "*",
						"identifiers": "pub"
					}]
				},
				"type": "upsight.configuration.dispatcher"
			}, {
				"configuration": {
					"retryMultiplier": 120,
					"requestInterval": 3600,
					"retryPowerBase": 1.61,
					"retryPowerExponentMax": 10
				},
				"type": "upsight.configuration.configurationManager"
			}, {
				"configuration": {
					"push_token_ttl": 604800,
					"auto_register": true
				},
				"type": "upsight.configuration.push"
			}]
		},
		"type": "upsight.configuration"
	}],
	"error": null
}
```

I won't go into details here



### batch

This is POST to https://batch.upsight-api.com/batch/v1/f1f4c1b2962446b38f72d5a456aac73c/

```json
{
	"sdk.version": "4.0.2",
	"device.manufacturer": "Apple",
	"screen.width": 375,
	"app.token": "*",
	"app.version": "1.110.0",
	"device.limit_ad_tracking": true,
	"screen.height": 667,
	"bundle.hash": "*",
	"ids.idfv": "*",
	"sdk.build": "+release.91a51d8",
	"request_ts": 1481902232,
	"device.carrier": "中国联通",
	"device.hardware": "iPhone8,1",
	"locale": "zh_CN",
	"app.bundleid": "com.google.ingress",
	"sid": *,
	"location.tz": "+0800",
	"device.connection": "Wifi",
	"device.os_version": "10.2",
	"device.type": "phone",
	"identifiers": "pub",
	"opt_out": false,
	"device.jailbroken": false,
	"screen.scale": 2,
	"device.os": "ios",
	"sessions": [{
		"session_num": 5558,
		"install_ts": 1457609995,
		"session_start": 1481901886,
		"events": [{
			"ts": 1481902146,
			"seq_id": 252627,
			"type": "upsight.session.pause",
			"user_attributes": {
				"days_inactive": -1,
				"hashed_user_id": "*",
				"player_approx_lat": *,
				"player_approx_lng": *,
				"language": "zh_CN",
				"faction": "ALIENS",
				"distance_to_portal": -1,
				"agent_level": 13
			}
		}, {
			"ts": 1481902230,
			"seq_id": 252629,
			"type": "upsight.session.resume",
			"user_attributes": {
				"days_inactive": -1,
				"hashed_user_id": "*",
				"player_approx_lat": *,
				"player_approx_lng": *,
				"language": "zh_CN",
				"faction": "ALIENS",
				"distance_to_portal": -1,
				"agent_level": 13
			}
		}, {
			"ts": 1481902230,
			"pub_data": {
				"ui_type": "FOREGROUND",
				"GL_MAX_TEXTURE_SIZE": 4096
			},
			"seq_id": 252630,
			"type": "pub.Stats.GL_MAX_TEXTURE_SIZE",
			"user_attributes": {
				"days_inactive": -1,
				"hashed_user_id": "*",
				"player_approx_lat": *,
				"player_approx_lng": *,
				"language": "zh_CN",
				"faction": "ALIENS",
				"distance_to_portal": -1,
				"agent_level": 13
			}
		}],
		"past_session_time": 2873649
	}]
}
```
The difference with the above is the event, if you are interested in diff

What to return.

```json
{
	"errobj": null,
	"error": null,
	"response": []
}
```

### OAuth2 Login

Here is the OAuth2 authentication, not the focus, are omitted

The last API is POST to https://www.googleapis.com/oauth2/v4/token, **form** contains

	client_id
	code
	grant_type
	redirect_uri
	verifier

then still returns a JSON that

```json
{
	"access_token": *,
	"token_type": "Bearer",
	"expires_in": 3600,
	"refresh_token": *,
	"id_token": *
}
```

**Authorizatoin is required from here on** (when requesting from appspot)

The format is `Bearer access_token`

#### collect

After that, ingress will send a statistic message, GET passing a reference to `https://ssl.google-analytics.com/collect`

Here the User-Agent is set to `GoogleAnalytics/3.10 (iPhone; U; CPU iPhone OS 10.2 like Mac OS X; zh-hans-cn-cn)`

Here are the parameters

	an: niantic-ingress
	av: 1.110.0
	a: 1360117771
	tid: UA-30116200-4
	cd: NiOSAuthenticationInfoViewController
	t: screenview
	cid: *
	ul: zh-hans-cn
	aid: com.google.ingress
	ds: app
	_u: *
	sr: 375x667
	v: 1
	_crc: 0
	_v: mi3.1.0
	sf: 100
	ht: 1481901425098
	qt: 155026
	z: *

### hanshake

URL: https://m-dot-betaspike.appspot.com/handshake

Here is still a POST form submission, but the content is JSON, suspected that the wrong settings

```json
{
"nemesisSoftwareVersion": "2016-11-18T13:44:21Z+c90631584ea7-i+opt",
"deviceSoftwareVersion": "10.2",
"a": *,
"reason": "SUP"
}
```

What is returned is important
First is a crack string `)]}'`
Then there is a huge JSON that returns.

```json
{
    "result": {
        "playerEntity": [user_guid, 1481899737145, {
            "playerPersonal": {
                "ap": "14031060",
                "energy": 10255,
                "allowNicknameEdit": false,
                "allowFactionChoice": false,
                "verifiedLevel": 13,
                "mediaHighWaterMarks": {
                    "General": 1809,
                    "RESISTANCE": 1328,
                    "ALIENS": 1327
                },
                "energyState": "XM_OK",
                "notificationSettings": {
                    "shouldSendEmail": false,
                    "maySendPromoEmail": false,
                    "shouldPushNotifyForAtPlayer": true,
                    "shouldPushNotifyForPortalAttacks": true,
                    "shouldPushNotifyForInvitesAndFactionInfo": true,
                    "shouldPushNotifyForNewsOfTheDay": true,
                    "shouldPushNotifyForEventsAndOpportunities": true,
                    "locale": "zh-Hans-CN"
                },
                "profileSettings": {
                    "areMetricsPublic": false
                },
                "verificationState": "UNVERIFIED",
                "extraXmTankEnergy": 0
            },
            "controllingTeam": {
                "team": "RESISTANCE"
            },
            "avatar": {
                "foreground": {
                    "layerId": "foreground4",
                    "tintColor": 16249235,
                    "imageUrl": "http://lh3.googleusercontent.com/COPtJLf8PEP0dgeehOg425-ASLkkQtrphoViG6Hy19mbHtyBDvanqWB4a6jPCaHhYpLYNaw9Ot9-0cfYm0Qc"
                },
                "background": {
                    "layerId": "background3",
                    "tintColor": 481625,
                    "imageUrl": "http://lh3.googleusercontent.com/WorWGSonh3XJrC7RWpVuBkTKcHoF2QtsolzqhmNUrZcaHKmfRgpTO_5QmpYTbBvkLkbTlF5LRW9gS_xDmTdY"
                }
            }
        }],
        "nickname": *,
        "xsrfToken": *,
        "storage": {
            "tutorial_complete_scanner_intro": "true:delim:1481895477261:delim:true",
            "game_intro_has_played": "true:delim:1457410095779:delim:true",
            "mission_complete_7": "ENDED_BY_NAGGING:delim:1457876587760:delim:true",
            "mission_complete_6": "ENDED_BY_NAGGING:delim:1457876587759:delim:true",
            "mission_complete_5": "ENDED_BY_NAGGING:delim:1457876587758:delim:true",
            "mission_complete_4": "SUCCESS:delim:1457876587753:delim:true",
            "mission_complete_3": "ENDED_BY_NAGGING:delim:1457876587757:delim:true",
            "mission_complete_0": "ENDED_BY_NAGGING:delim:1457876587754:delim:true",
            "mission_complete_1": "ENDED_BY_NAGGING:delim:1457876587755:delim:true",
            "mission_complete_2": "ENDED_BY_NAGGING:delim:1457876587756:delim:true",
            "all_missions_complete_announcement_made": "true:delim:1457410237730:delim:true",
            "tutorial_complete_hack": "true:delim:1457410237722:delim:true",
            "training_portal_photo_url": "http://lh6.ggpht.com/xZU07oPfUpRDnz_2og2L1E3O4T74iMUyCVtB5lL6m16lh0PnZPtBMf8XjkpYbq8Veby0eHNhCGNRwQrYNg:delim:1457439507178:delim:true",
            "training_portal_lng_degrees": *,
            "training_portal_lat_degrees": *,
            "training_portal_title": "本科9栋:delim:1457439507178:delim:true",
            "tutorial_complete_xm": "true:delim:1457412143790:delim:true",
            "tutorial_complete_xmp": "true:delim:1457412131239:delim:true",
            "tutorial_complete_profile_link": "true:delim:1457422332944:delim:true",
            "tutorial_complete_deploy": "true:delim:1457488741871:delim:true"
        },
        "pregameStatus": {
            "action": "NO_ACTIONS_REQUIRED",
            "dialogText": ""
        },
        "initialKnobs": {
            "bundleMap": {
                "ClientFeatureKnobs": {
                    "enableEmbeddedYouTubePlayback": true,
                    "showGlobalMap": true,
                    "logSkipRegex": "\\b\\B",
                    "enableParticleFilter": true,
                    "enableGAViolationReporting": false,
                    "portalKeyCardRefreshIntervalSecs": 5,
                    "portalInfoRefreshIntervalSecs": 2,
                    "enableInviteNag": true,
                    "inviteNagDelayDays": 14,
                    "enableVerificationNag": false,
                    "verificationNagDelayDays": 3,
                    "refreshTimeMs": "-1",
                    "playerProfileCacheExpirationSecs": "60",
                    "enableDelayGpsPause": true,
                    "enableAdvancedAnalytics": true,
                    "playerVerificationEnabled": 0,
                    "enableExtraNativeRenderedText": -1,
                    "enableShareMission": 10740,
                    "profileUrlFormat": "http://plus.google.com/%s",
                    "profileUrlFormatApp": "gplus://plus.google.com/%s",
                    "factionChoiceBeforeSpaceToFace": true,
                    "minGlobBundleSizeXm": 1500,
                    "enableMissions": 10610,
                    "enableBadAnalyticsTrackingEventLogging": 10570,
                    "enableIOSPushNotifications": 10640,
                    "iOSPortalDiscoveryProcessIntervalMS": "200000",
                    "enableMissionsConnectivityRecovery": 10610,
                    "enableMissionsStatePolling": 10610,
                    "enableHackLongPressIndicator": 10612,
                    "enableIOSNativeYoutubePlayer": 10620,
                    "enableIOSPortalDetailPanel": 10630,
                    "enableIOSMultiPhotoViewer": 10640,
                    "enableIOSPhotoSubmission": 10640,
                    "enableIOSGridPhotoViewer": 10650,
                    "enableCommunityTab": 10740,
                    "enableBoostRechargeV2": 10760,
                    "enableSetLocale": 10771,
                    "disableNewPortalSubmissions": -1,
                    "enableLocalLinkChecks": 10790,
                    "enableStore": 10851,
                    "enableRecycleConfirmationDialog": 0,
                    "enableGlyphCommandChannel": 10900,
                    "enableGlyphRedoButton": 10910,
                    "upsightLocationDecimalPlacesToRound": 2,
                    "enableExtendedFireMenu": 11010
                },
                "SmartNotificationsClientKnobs": {
                    "enableSmartNotifications": 10670,
                    "maximumCachedHandshakeResultAgeMs": "43200000",
                    "enableR": 10691,
                    "enableRIOS": 10700,
                    "playerRProbabilityE6": 1000000,
                    "rLPlayerMs": "864000000",
                    "rMinPollingDelayMs": "900000",
                    "maximumPortalCacheSize": 5000,
                    "portalCacheEntryExpirationTimeMs": "1209600000",
                    "nearbyPortalDistanceM": 200,
                    "loiteringMs": "300000",
                    "noLongerNearbyDistanceM": 100,
                    "rNearbyPortalDistanceGrowthRate": 50,
                    "rMaxGrownNearbyPortalDistanceM": 1000,
                    "rWaitPeriodsMs": ["86400000", "172800000", "259200000", "432000000"],
                    "rIOSAlwaysLocationAccessMinIntervalMs": "5256000000",
                    "amSessionLengthMs": "1800000",
                    "enableAm": 10700,
                    "playerAmProbabilityE6": 1000000,
                    "amNearbyPortalDistanceM": 37,
                    "amNoLongerNearbyDistanceM": 40,
                    "amAndroidLocationRequestPriority": 102,
                    "amAndroidLocationRequestIntervalMs": "30000",
                    "amAndroidLocationRequestFastestIntervalMs": "5000",
                    "amAndroidLocationRequestSmallestDisplacementM": 0
                },
                "TranslationClientKnobs": {
                    "enableCloudTranslations": 10860,
                    "translationUrls": {
                        "zh_TW.json": "http://storage.googleapis.com/ingress-translations/2016.11.04/zh_TW.json",
                        "ru": "http://storage.googleapis.com/ingress-translations/2016.11.04/ru.json",
                        "fr": "http://storage.googleapis.com/ingress-translations/2016.11.04/fr.json",
                        "en": "http://storage.googleapis.com/ingress-translations/2016.11.04/en.json",
                        "nl": "http://storage.googleapis.com/ingress-translations/2016.11.04/nl.json",
                        "pt": "http://storage.googleapis.com/ingress-translations/2016.11.04/pt.json",
                        "no": "http://storage.googleapis.com/ingress-translations/2016.11.04/no.json",
                        "de": "http://storage.googleapis.com/ingress-translations/2016.11.04/de.json",
                        "ko": "http://storage.googleapis.com/ingress-translations/2016.11.04/ko.json",
                        "it": "http://storage.googleapis.com/ingress-translations/2016.11.04/it.json",
                        "sv": "http://storage.googleapis.com/ingress-translations/2016.11.04/sv.json",
                        "pl": "http://storage.googleapis.com/ingress-translations/2016.11.04/pl.json",
                        "cs": "http://storage.googleapis.com/ingress-translations/2016.11.04/cs.json",
                        "zh_CN.json": "http://storage.googleapis.com/ingress-translations/2016.11.04/zh_CN.json",
                        "ja": "http://storage.googleapis.com/ingress-translations/2016.11.04/ja.json",
                        "es": "http://storage.googleapis.com/ingress-translations/2016.11.04/es.json"
                    }
                },
                "GlyphCommandClientKnobs": {
                    "commands": {
                        "ikj": "GLYPH_ACCEPT_MORE_KEYS",
                        "jkgh": "GLYPH_ACCEPT_COMPLEX_HACK",
                        "hkg": "GLYPH_ACCEPT_LESS_KEYS",
                        "ji": "GLYPH_ACCEPT_SIMPLE_HACK"
                    },
                    "hints": ["GLYPH_HINT_LESS_KEYS", "GLYPH_HINT_MORE_KEYS", "GLYPH_HINT_SIMPLE_HACK"]
                },
                "StagingKnobs": {
                    "playerStoreProbabilityE6": 1000000,
                    "oldAnalyticsProbabilityE6": 1000000,
                    "newAnalyticsProbabilityE6": 1000000
                },
                "InventoryKnobs": {
                    "maxInventoryItems": 2000,
                    "resourceLimits": {
                        "KEY_CAPSULE": 5
                    },
                    "cmuRestockItemId": "ingress.cmu.large",
                    "itemRestockResourceToId": {},
                    "powerupRestockToId": {
                        "LOOK": "ingress.beacon.look",
                        "RES": "ingress.beacon.res",
                        "NIA": "ingress.beacon.nia",
                        "ENL": "ingress.beacon.enl",
                        "FRACK": "ingress.fracker",
                        "MEET": "ingress.beacon.meet"
                    }
                },
                "PortalKnobs": {
                    "resonatorLimits": {
                        "bands": [{
                            "remaining": 1,
                            "applicableLevels": [8]
                        }, {
                            "remaining": 4,
                            "applicableLevels": [2]
                        }, {
                            "remaining": 8,
                            "applicableLevels": [1]
                        }, {
                            "remaining": 2,
                            "applicableLevels": [5]
                        }, {
                            "remaining": 4,
                            "applicableLevels": [3]
                        }, {
                            "remaining": 2,
                            "applicableLevels": [6]
                        }, {
                            "remaining": 4,
                            "applicableLevels": [4]
                        }, {
                            "remaining": 1,
                            "applicableLevels": [7]
                        }]
                    },
                    "maxModsPerPlayer": 2,
                    "radiusParamsSet": [],
                    "allowRegionSpecificSubmissions": false,
                    "minLevelForSubmission": 2
                },
                "recycleKnobs": {
                    "recycleValuesMap": {
                        "BOOSTED_POWER_CUBE": [160, 160, 160, 160, 160, 160, 160, 160],
                        "MULTIHACK": [20, 40, 60, 80, 100, 120, 140, 160],
                        "EMP_BURSTER": [20, 40, 60, 80, 100, 120, 140, 160],
                        "INTEREST_CAPSULE": [20, 40, 60, 80, 100, 120, 140, 160],
                        "TURRET": [20, 40, 60, 80, 100, 120, 140, 160],
                        "MEDIA": [20, 40, 60, 80, 100, 120, 140, 160],
                        "EXTRA_SHIELD": [20, 40, 60, 80, 100, 120, 140, 160],
                        "ULTRA_STRIKE": [20, 40, 60, 80, 100, 120, 140, 160],
                        "POWER_CUBE": [1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000],
                        "FLIP_CARD": [20, 40, 60, 80, 100, 120, 140, 160],
                        "ULTRA_LINK_AMP": [20, 40, 60, 80, 100, 120, 140, 160],
                        "CAPSULE": [20, 40, 60, 80, 100, 120, 140, 160],
                        "HEATSINK": [20, 40, 60, 80, 100, 120, 140, 160],
                        "RES_SHIELD": [20, 40, 60, 80, 100, 120, 140, 160],
                        "PORTAL_POWERUP": [20, 40, 60, 80, 100, 120, 140, 160],
                        "LINK_AMPLIFIER": [20, 40, 60, 80, 100, 120, 140, 160],
                        "EMITTER_A": [20, 40, 60, 80, 100, 120, 140, 160],
                        "PORTAL_LINK_KEY": [500, 500, 500, 500, 500, 500, 500, 500],
                        "KEY_CAPSULE": [20, 40, 60, 80, 100, 120, 140, 160],
                        "FORCE_AMP": [20, 40, 60, 80, 100, 120, 140, 160]
                    }
                },
                "WeaponRangeKnobs": {
                    "xmpDamageRangeMap": {
                        "1": 42,
                        "3": 58,
                        "2": 48,
                        "5": 90,
                        "4": 72,
                        "7": 138,
                        "6": 112,
                        "8": 168
                    },
                    "ultraStrikeDamageRangeMap": {
                        "1": 10,
                        "3": 16,
                        "2": 13,
                        "5": 21,
                        "4": 18,
                        "7": 27,
                        "6": 24,
                        "8": 30
                    },
                    "maxPowerBoostPercentage": 20,
                    "powerBoostPeriodHardS": 0.6,
                    "powerBoostPeriodEasyS": 1.5,
                    "powerBoostWindowHardS": 0.06,
                    "powerBoostWindowEasyS": 0.1,
                    "powerBoostPowAnimation": 1.0,
                    "powerBoostPowScore": 1.0,
                    "maxFireRateSeconds": 1.5
                },
                "XmCostKnobs": {
                    "xmpFiringCostByLevel": [50, 100, 150, 200, 250, 300, 350, 400],
                    "shieldDeployCostByLevel": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                    "linkAmplifierDeployCostByLevel": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                    "forceAmplifierDeployCostByLevel": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                    "heatsinkDeployCostByLevel": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                    "multihackDeployCostByLevel": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                    "turretDeployCostByLevel": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                    "resonatorDeployCostByLevel": [50, 100, 150, 200, 250, 300, 350, 400],
                    "resonatorUpgradeCostByLevel": [50, 100, 150, 200, 250, 300, 350, 400],
                    "portalHackFriendlyCostByLevel": [50, 100, 150, 200, 250, 300, 350, 400],
                    "portalHackNeutralCostByLevel": [50, 100, 150, 200, 250, 300, 350, 400],
                    "portalHackEnemyCostByLevel": [50, 100, 150, 200, 250, 300, 350, 400],
                    "flipCardCostByLevel": [1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000],
                    "portalModByLevel": {
                        "MULTIHACK": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                        "HEATSINK": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                        "TURRET": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                        "EXTRA_SHIELD": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                        "FORCE_AMP": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                        "ULTRA_LINK_AMP": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                        "RES_SHIELD": [200, 400, 600, 800, 1000, 1200, 1400, 1600],
                        "LINK_AMPLIFIER": [200, 400, 600, 800, 1000, 1200, 1400, 1600]
                    },
                    "powerupByLevel": {
                        "PORTAL_POWERUP": [200, 400, 600, 800, 1000, 1200, 1400, 1600]
                    },
                    "boostRechargeResonatorsCost": 15000
                },
                "ScannerKnobs": {
                    "updateIntervalMs": 30000,
                    "updateDistanceM": 10,
                    "maxUpdateIntervalMs": 30000,
                    "minUpdateIntervalMs": 5000,
                    "updateIntervalThrottlingTriggerVelocityMps": 3.0,
                    "updateIntervalThrottlingRate": 0.65,
                    "rangeM": 300,
                    "missionsUpdateIntervalMs": 60000,
                    "missionsUpdateDistanceM": 500,
                    "maxLinkAutoCheckRangeM": 200000,
                    "reportRequestPerformancePercentE6": 0,
                    "crittercismSendPerfDataProbabilityE6": 0,
                    "crittercismSendLogcatProbabilityE6": 0
                },
                "PortalModSharedKnobs": {
                    "diminishingValues": {
                        "FORCE_AMPLIFIER": [1000, 250, 125, 125],
                        "ATTACK_FREQUENCY": [1000, 250, 125, 125],
                        "LINK_RANGE_MULTIPLIER": [1000, 250, 125, 125],
                        "TEMP_LINK_RANGE_MULTIPLIER_SORTED": [1000, 250, 125, 125],
                        "TEMP_ATTACK_FREQUENCY_SORTED": [1000, 250, 125, 125],
                        "BURNOUT_INSULATION": [1000, 500, 500, 500],
                        "HACK_SPEED": [1000, 500, 500, 500],
                        "TEMP_FORCE_AMPLIFIER_SORTED": [1000, 250, 125, 125]
                    },
                    "directValues": {
                        "OUTGOING_LINKS_BONUS": [8, 8, 8, 8]
                    }
                },
                "NearbyPortalKnobs": {
                    "repopulateDistanceMeters": 500.0,
                    "repopulateTimeMilliseconds": "30000"
                },
                "PlayerAnnounceSharedKnobs": {
                    "featureActivationDate": "1377241200000"
                },
                "PlayerVerificationSharedKnobs": {
                    "unverifiedXmCapacity": 3000,
                    "unverifiedInventoryCapacity": 100,
                    "unverifiedApCapacity": 19999,
                    "enableVerificationFeature": false,
                    "enableVerifiedGameActions": false,
                    "numPinCodeDigits": 4
                },
                "PlayerLevelKnobs": {
                    "levelUpRequirements": {
                        "11": {
                            "apRequired": "6000000",
                            "achievementsRequired": {
                                "SILVER": 6,
                                "GOLD": 4
                            }
                        },
                        "10": {
                            "apRequired": "4000000",
                            "achievementsRequired": {
                                "SILVER": 5,
                                "GOLD": 2
                            }
                        },
                        "13": {
                            "apRequired": "12000000",
                            "achievementsRequired": {
                                "PLATINUM": 1,
                                "GOLD": 7
                            }
                        },
                        "12": {
                            "apRequired": "8400000",
                            "achievementsRequired": {
                                "SILVER": 7,
                                "GOLD": 6
                            }
                        },
                        "15": {
                            "apRequired": "24000000",
                            "achievementsRequired": {
                                "PLATINUM": 3
                            }
                        },
                        "14": {
                            "apRequired": "17000000",
                            "achievementsRequired": {
                                "PLATINUM": 2
                            }
                        },
                        "16": {
                            "apRequired": "40000000",
                            "achievementsRequired": {
                                "PLATINUM": 4,
                                "BLACK": 2
                            }
                        },
                        "1": {
                            "apRequired": "0",
                            "achievementsRequired": {}
                        },
                        "3": {
                            "apRequired": "20000",
                            "achievementsRequired": {}
                        },
                        "2": {
                            "apRequired": "2500",
                            "achievementsRequired": {}
                        },
                        "5": {
                            "apRequired": "150000",
                            "achievementsRequired": {}
                        },
                        "4": {
                            "apRequired": "70000",
                            "achievementsRequired": {}
                        },
                        "7": {
                            "apRequired": "600000",
                            "achievementsRequired": {}
                        },
                        "6": {
                            "apRequired": "300000",
                            "achievementsRequired": {}
                        },
                        "9": {
                            "apRequired": "2400000",
                            "achievementsRequired": {
                                "SILVER": 4,
                                "GOLD": 1
                            }
                        },
                        "8": {
                            "apRequired": "1200000",
                            "achievementsRequired": {}
                        }
                    },
                    "maxPlayerLevel": 16,
                    "xmCapacityByLevel": {
                        "11": 14500,
                        "10": 13000,
                        "13": 17500,
                        "12": 16000,
                        "15": 20500,
                        "14": 19000,
                        "16": 22000,
                        "1": 3000,
                        "3": 5000,
                        "2": 4000,
                        "5": 7000,
                        "4": 6000,
                        "7": 9000,
                        "6": 8000,
                        "9": 11500,
                        "8": 10000
                    },
                    "boostedPowerCubeCapacityByLevel": {
                        "11": 40800,
                        "10": 38400,
                        "13": 45600,
                        "12": 43200,
                        "15": 50400,
                        "14": 48000,
                        "16": 52800,
                        "1": 18000,
                        "3": 22500,
                        "2": 20250,
                        "5": 27000,
                        "4": 24750,
                        "7": 31500,
                        "6": 29250,
                        "9": 36000,
                        "8": 33750
                    }
                },
                "MapCompositionRootKnobs": {
                    "mapAreas": [{
                        "description": "S Korea",
                        "epoch": 1,
                        "mapProvider": "OSM",
                        "boundingRects": [{
                            "north": 37.69547616496338,
                            "south": 34.83562779241368,
                            "east": 131.16790087890627,
                            "west": 124.44700830078114
                        }, {
                            "north": 37.69547616496338,
                            "south": 33.025620228813466,
                            "east": 128.76189501953127,
                            "west": 124.44700830078114
                        }, {
                            "north": 37.84094374831599,
                            "south": 37.561299473030594,
                            "east": 126.86126025390627,
                            "west": 126.18010156249989
                        }, {
                            "north": 38.358197906100926,
                            "south": 37.62589818166485,
                            "east": 129.34966357421877,
                            "west": 126.65800683593739
                        }, {
                            "north": 38.634814820135226,
                            "south": 38.358197906100926,
                            "east": 128.55040820312502,
                            "west": 128.14116113281239
                        }]
                    }, {
                        "description": "World",
                        "epoch": -1,
                        "mapProvider": "GMM",
                        "mapProviderName": "",
                        "boundingRects": [{
                            "north": 90.0,
                            "south": -90.0,
                            "east": 180.0,
                            "west": -180.0
                        }]
                    }],
                    "mapProviders": [{
                        "name": "GMM",
                        "baseUrl": "http://mobilemaps.clients.google.com/glm/mmap",
                        "queryFormat": ""
                    }, {
                        "name": "OSM",
                        "baseUrl": "http://maps.nianticlabs.com",
                        "queryFormat": "ntp-maptiles/epoch-%04d/%s.gmm"
                    }]
                }
            },
            "syncTimestamp": "1481903753065"
        }
    }
}
```

Here's something you can do a tutorial, basically, the effect of various items, upgrade requirements are written very clearly in it, other students are welcome to do a statistic

For example, the ultraStrikeDamageRangeMap illustrates the US action distance for each level

Here is probably for hot updates, put the settings on the server side (running not my traffic anyway, gg)



### setLocale

Then it's POST to https://m-dot-betaspike.appspot.com/rpc/emptyBasket/setLocale

Direct form content is simple and violent

	{"params":["zh-Hans-CN"]}

The return is also an empty{}

### globalRegionMap

This time it is GET to https://m-dot-betaspike.appspot.com/globalRegionMap to get the map of the spinning earth at the beginning of the game

Response is a 256*128 black, green and red chart


### batch Second time

Another device information upload is triggered, except request_ts is updated with a little difference in events

```json
"pub_data": {
	"type": "upsight.session.pause",
	"ui_type": "FOREGROUND"
},
...,
"type": "pub.StartupWheelsOfAdaSubActivity",
```



This was all before clicking the `I got it` button on the request


## Game API

### getGameScore

Apparently it is used to get the score of the game

	API URL: https://m-dot-betaspike.appspot.com/rpc/playerUndecorated/getGameScore
	API Request: params: []
	API Response:

```json
{
	"result": {
		"alienScore": "569243223",
		"resistanceScore": "723155462"
	},
	"gameBasket": {
		"gameEntities": [],
		"inventory": [],
		"deletedEntityGuids": []
	}
}
```

I'm too lazy to explain, just post the returned package

## getObjectsInCells

Used to obtain map elements

	API URL： https://m-dot-betaspike.appspot.com/rpc/gameplay/getObjectsInCells
	API Request: params: {clientBasket: {clientBlob}, knobSyncTimestamp, energyGlobGuids: [], playerLocation: value, dates: [value], cellsAsHex: [...], cells}

Here clientBasket is the verification information, the whole process will not change, probably to reverse the algorithm to know

playerLocation is a string like 'hex,hex', latitude and longitude (keep 6 decimal places and remove the decimal point) are converted to hexadecimal and then concatenated with spaces

dates is a list of 20 zeros (not sure)

cellsAsHex is the token of the cell slice, a list of 20 16-bit hex values


	API Response:

```json
{
	"result": "1481906071300",
	"gameBasket": {
		"gameEntities": [
			[guid, time, {
				"capturedRegion": {
					"vertexA": {
						"guid": guid,
						"location": {
							"latE6": *,
							"lngE6": *
						}
					},
					"vertexB": {
						...
					},
					"vertexC": {
						...
					}
				},
				"entityScore": {
					"entityScore": "8"
				},
				"creator": {
					"creatorGuid": user_guid,
					"creationTime": time
				},
				"controllingTeam": {
					"team": "RESISTANCE"
				}
			}],
			...,
			[guid, time, {
				"photoStreamInfo": {
					"coverPhoto": {
						"portalImageGuid": *,
						"imageUrl": *,
						"attributionMarkup": ["PLAYER", {
							"plain": agent_name,
							"guid": guid,
							"team": "ALIENS"
						}],
						"voteCount": 0
					},
					"photoCount": 1
				},
				"locationE6": {
					"latE6": *,
					"lngE6": *
				},
				"discoverer": {
					"playerMarkupArgSet": {
						"plain": agent_name,
						"guid": *,
						"team": "ALIENS"
					}
				},
				"resonatorArray": {
					"resonators": [{
						"level": 3,
						"distanceToPortal": 28,
						"ownerGuid": agent_guid,
						"energyTotal": 1327,
						"slot": 0,
						"id": uuid
					}, ...]
				},
				"descriptiveText": {
					"map": {
						"TITLE": *,
						"ADDRESS": *
					}
				},
				"turret": {},
				"portalV2": {
					"linkedEdges": [],
					"linkedModArray": [{
						"installingUser": agent_guid,
						"stats": {
							"MITIGATION": "30",
							"REMOVAL_STICKINESS": "0"
						},
						"rarity": "COMMON",
						"displayName": "Portal Shield",
						"type": "RES_SHIELD"
					}, ...],
					"missionStartPoint": false
				},
				"controllingTeam": {
					"team": "RESISTANCE"
				},
				"captured": {
					"capturingPlayerId": agent_guid
				},
				"defaultActionRange": {}
			}],
			...,

			[guide, time, {
				"edge": {
					"originPortalGuid": guid,
					"destinationPortalGuid": guid,
					"originPortalLocation": {
						"latE6": *,
						"lngE6": *
					},
					"destinationPortalLocation": {
						"latE6": *,
						"lngE6": *
					}
				},
				"creator": {
					"creatorGuid": *,
					"creationTime": *
				},
				"controllingTeam": {
					"team": "RESISTANCE"
				}
			}],
			...,
			[guid, time, {
				"imageByUrl": {
					"imageUrl": *
				},
				"photoStreamInfo": {
					"coverPhoto": {
						"portalImageGuid": guid,
						"imageUrl": *,
						"attributionMarkup": ["PLAYER", {
							"plain": agent_name,
							"guid": guid,
							"team": "ALIENS"
						}],
						"voteCount": 5
					},
					"photoCount": 1
				},
				"locationE6": {
					"latE6": *,
					"lngE6": *
				},
				"discoverer": {
					"playerMarkupArgSet": {
						"plain": agent_name,
						"guid": *,
						"team": "ALIENS"
					}
				},
				"resonatorArray": {
					"resonators": [...]
				},
				"descriptiveText": {
					"map": {
						"TITLE": *,
						"DESCRIPTION": *,
						"ADDRESS": *
					}
				},
				"turret": {},
				"portalV2": {
					"linkedEdges": [{
						"edgeGuid": guid,
						"otherPortalGuid": guid",
						"isOrigin": false
					}, ...],
					"linkedModArray": [..., null],
					"missionStartPoint": false
				},
				"controllingTeam": {
					"team": "RESISTANCE"
				},
				"captured": {
					"capturingPlayerId": agent_guid
				},
				"defaultActionRange": {}
			}],
			...,
			[guid, time, {
				"timedPowerupSet": {
					"powerUpItems": []
				},
				"imageByUrl": {
					"imageUrl": *
				},
				"photoStreamInfo": {
					"coverPhoto": {
						"portalImageGuid": guid,
						"imageUrl": *,
						"attributionMarkup": ["PLAYER", {
							"plain": agent_user,
							"guid": guid,
							"team": "ALIENS"
						}],
						"voteCount": 2
					},
					"photoCount": 1
				},
				"locationE6": {
					"latE6": *,
					"lngE6": *
				},
				"discoverer": {
					"playerMarkupArgSet": {
						"plain": agent_user,
						"guid": guid,
						"team": "ALIENS"
					}
				},
				"resonatorArray": {
					"resonators": [...]
				},
				"descriptiveText": {
					"map": {
						"ADDRESS": *,
						"TITLE": (
					}
				},
				"turret": {},
				"portalV2": {
					"linkedEdges": [...],
					"linkedModArray": [null, null, null, null],
					"missionStartPoint": false
				},
				"controllingTeam": {
					"team": "RESISTANCE"
				},
				"captured": {
					"capturingPlayerId": agent_guid
				},
				"defaultActionRange": {}
			}],
			...,
		}],
		"apGains": [],
		"inventory": [],
		"deletedEntityGuids": [],
		"energyGlobGuids": [guid, ...],
		"energyGlobTimestamp": "1481906071300",
		"refreshEntityGuids": [guid, ...]
	}
}
```

Very clear. No more.

### getCurrentMission

	API URL: https://m-dot-betaspike.appspot.com/rpc/gameplay/getCurrentMission
	API Request: params: {clientBasket: {clientBlob}, knobSyncTimestamp, energyGlobGuids: [], playerLocation: value}
	API Response: Task and gameBasket, here because I do not have the current task, omitted


### getNewsOfTheDay


	API URL:https://m-dot-betaspike.appspot.com/rpc/playerUndecorated/getNewsOfTheDay
	API Request: params: [value]

This value is a string that doesn't create any ghosts

	API Response:  gameBasket: {gameEntities: [], inventory: [], deletedEntityGuids: [] }


This API has to be tested when there is an event

### getPaginatedPlexts

Get comm messages


	API URL: https://m-dot-betaspike.appspot.com/rpc/gameplay/getPaginatedPlexts

	API Request:
```json
{
	"params": {
		"clientBasket": {
			"clientBlob": *
		},
		"knobSyncTimestamp": *,
		"energyGlobGuids": [],
		"playerLocation": value,
		"categories": -1,
		"ascendingTimestampOrder": false,
		"desiredNumItems": 100,
		"maxTimestampMs": -1,
		"minTimestampMs": 1481900888021,
		"cellsAsHex": [...],
		"factionOnly": false
	}
}
```

The parameters for this can be found in the Intel Map

	API Response:

```json
{
	"result": [
		[guid, time, {
			"locationE6": {
				"latE6": *,
				"lngE6": *
			},
			"plext": {
				"text": *,
				"team": "ALIENS",
				"markup": [
					["SENDER", {
						"plain": agent_name,
						"guid": *,
						"team": "ALIENS"
					}],
					["TEXT", {
						"plain": ""
					}],
					["AT_PLAYER", {
						"plain": *,
						"guid": guid,
						"team": "ALIENS"
					}],
					["TEXT", {
						"plain": *
					}]
				],
				"plextType": "PLAYER_GENERATED",
				"categories": 1
			}
		}]
	],
	"gameBasket": {
		"gameEntities": [],
		"playerEntity": [...],
		"apGains": [],
		"inventory": [],
		"deletedEntityGuids": [],
		"serverBlob": *
	}
}
```

Omitted some duplicates, here you can also refer to the Intel Map API


### getInventory

	API URL: https://m-dot-betaspike.appspot.com/rpc/playerUndecorated/getInventory
	API Request:
```json
{
	"params": {
		"lastQueryTimestamp": timestamp
	}
}
```

	API Response:(Return only updated items)

```json
{
	"result": "1481911112158",
	"gameBasket": {
		"gameEntities": [],
		"inventory": [
		["56b24e20197240d2a2330e7fb60fa6a1.5", 1473951636802, {
						"inInventory": {
							"playerId": *,
							"acquisitionTimestampMs": *
						},
						"modResource": {
							"displayName": "Heat Sink",
							"stats": {
								"REMOVAL_STICKINESS": "0",
								"HACK_SPEED": "200000"
							},
							"rarity": "COMMON",
							"resourceType": "HEATSINK"
						},
						"displayName": {
							"displayName": "Heat Sink",
							"displayDescription": "Mod that reduces cooldown time between Portal hacks."
						}
				}],...,
		],
		"deletedEntityGuids": []
	}
}
```
The inventory can see what the updated items are, after the forced synchronization, this inside is everything, very large. (After all, each item a list ...)

### registerForApn

	API URL: https://m-dot-betaspike.appspot.com/rpc/emptyBasket/registerForApn
	API Request: params: [key, "com.google.ingress", "2048"]
	API Response: {}

Used to register token


### unregisterForApn

	API URL: https://m-dot-betaspike.appspot.com/rpc/emptyBasket/unregisterForApn
	API Request:
	API Response: {}



### getInviteInfo

	API URL: https://m-dot-betaspike.appspot.com/rpc/playerUndecorated/getInviteInfo
	API Request: params: []
	API Response:

```json
{
	"result": {
		"numAvailableInvites": 298,
		"inviteeToStatusMap": {
			email: status,
			...
		},
		"apGainOnInviteAccepted": "3000"
	},
	"gameBasket": {
		"gameEntities": [],
		"inventory": [
		],
		"deletedEntityGuids": []
	}
}
```



This API is good, you can see all the people you invite and their status, even ACCEPTED_ANOTHER_PLAYERS_INVITE

### putBulkPlayerStorage

	API URL:https://m-dot-betaspike.appspot.com/rpc/playerUndecorated/putBulkPlayerStorage
	API Request: prams: [{tutorial_complete_scanner_intro: value}]

This value is a string in the format `true:delim:1481912306360:delim:true`

	API Response:
```json
{
	"result": "SUCCESS",
	"gameBasket": {
		"gameEntities": [],
		"inventory": [],
		"deletedEntityGuids": []
	}
}
```


### getPlayerProfile2

This is used to get information about yourself

	API URL: https://m-dot-betaspike.appspot.com/rpc/playerUndecorated/getPlayerProfile2
	API Request: params: [agent_name]
	API Response: result: {team, avatar: {foregroundLayer: {id, imageURL, layer}, backgroundLayer: {id, imageUrl, layer}, avatarColorForeground, avatarColorBackground}, metrics: [{metricName, metricCategory, formattedValueAllTime, formattedValue30Days, formattedValue7Days}, ...], highlightedAchievements: [title, description, group, tiers: [{value, badgeImageUrl, locked}, ...], timestampAwarded], firstAchievementContinuationToken, ap, verifiedLeve, achievementCounts: {SILVER, GOLD, BLACK, PLATINUM, BRONZE}, gPlusId, highlightedMissionBadges: [{missionGuid, missionTitle, missionDescription, imageUrl}, ...], firstMissionBadgeContinuationToken}, gameBasket: {gameEntities, inventory, deletedEntityGuids}

All are self-annotated

Metrics are statistics, such as Unique Portals Visite

highlightedAchievements is the tile information

highlightedMissionBadges is the mission information


### getPaginatedMissionBadges

	API URL: https://m-dot-betaspike.appspot.com/rpc/playerUndecorated/getPaginatedMissionBadges
	API Request: params: [{continuationToken, nickname}]

The continuationToken is the firstMissionBadgeContinuationToken returned by the API above or the continuationToken returned by yourself

	API Response:
```json
{
	"result": {
		"missionBadges": [{
			"missionGuid": guid,
			"missionTitle": *,
			"missionDescription": *,
			"imageUrl": *
		}, ...],
		"continuationToken": *
	},
	"gameBasket": {
		"gameEntities": [],
		"inventory": [],
		"deletedEntityGuids": []
	}
}
```

This API is used to get task icons and details

### getNearbyMissions

	API URL: https://m-dot-betaspike.appspot.com/rpc/gameplay/getNearbyMissions
	API Request:
```json
{
	"params": {
		"clientBasket": {
			"clientBlob": *
		},
		"knobSyncTimestamp": *,
		"energyGlobGuids": [],
		"location": value,
		"continuationToken": value
	}
}
```

The value here is also a comma separated by two hexes

A continuationToken of null means the first time the task is viewed, after that the value is the continuationToken returned by the API itself

	API Response:
```json
{
	"result": {
		"missionSnippets": [{
			"header": {
				"guid": guid,
				"title": *,
				"logoUrl": *,
				"startLocation": value,
				"badgeUrl": *,
				"stats": {
					"ratingE6": "0",
					"medianCompletionTimeMs": "0",
					"numUniqueCompletedPlayers": "0"
				},
				"status": "NOT_STARTED",
				"authorNickname": agent_name,
				"authorTeam": "ALIENS"
			},
			"version": "2"
		},...],
		"continuationToken": value
	},
	"gameBasket": {
		"gameEntities": [],
		"playerEntity": [guid, 1481912305897, {
			"avatar": {
				"foreground": {
					"layerId": "foreground4",
					"tintColor": 16249235,
					"imageUrl": *
				},
				"background": {
					"layerId": "background3",
					"tintColor": 481625,
					"imageUrl": *
				}
			},
			"playerPersonal": {
				"ap": "14031060",
				"energy": 17500,
				"allowNicknameEdit": false,
				"allowFactionChoice": false,
				"verifiedLevel": 13,
				"mediaHighWaterMarks": {
					"General": 1809,
					"RESISTANCE": 1328,
					"ALIENS": 1327
				},
				"energyState": "XM_OK",
				"notificationSettings": {
					"shouldSendEmail": false,
					"maySendPromoEmail": false,
					"shouldPushNotifyForAtPlayer": true,
					"shouldPushNotifyForPortalAttacks": true,
					"shouldPushNotifyForInvitesAndFactionInfo": true,
					"shouldPushNotifyForNewsOfTheDay": true,
					"shouldPushNotifyForEventsAndOpportunities": true,
					"locale": "zh-Hans-CN"
				},
				"profileSettings": {
					"areMetricsPublic": false
				},
				"verificationState": "UNVERIFIED",
				"extraXmTankEnergy": 0
			},
			"controllingTeam": {
				"team": "RESISTANCE"
			}
		}],
		"apGains": [],
		"inventory": [],
		"deletedEntityGuids": [],
		"serverBlob": *,
		"refreshEntityGuids": [guid, ...]
	}
}
```

### getNickNamesFromPlayerIds

	API URL: https://m-dot-betaspike.appspot.com/rpc/playerUndecorated/getNickNamesFromPlayerIds
	API Request: params: [agent_guid]
	API Response: result: [agent_name], gameBasket: {gameEntities: , inventory: , deletedEntityGuids: }


### collectItemsOrGlyphsFromPortal


	API URL: https://m-dot-betaspike.appspot.com/rpc/gameplay/collectItemsOrGlyphsFromPortal

#### Without Glyphs


	API Request:
```json
{
	"params": {
		"glyphGameRequested": false,
		"spewInfluenceGlyphSequence": null,
		"userInputGlyphSequence": null,
		"clientBasket": {
			"clientBlob": *
		},
		"knobSyncTimestamp": *,
		"energyGlobGuids": [],
		"portalGuid": guid,
		"location": value
	}
}
```

	API Response:
```json
{
	"result": {
		"items": {
			"addedGuids": [guid, ...],
			"baseHackMultiplier": "1"
		}
	},
	"gameBasket": {
		"playerDamages": [],
		"gameEntities": [],
		"playerEntity": [guid, 1481913704637, {
			"playerPersonal": {
				...,
				"energyState": "XM_OK",
				"notificationSettings": ...,
				"profileSettings": {
					"areMetricsPublic": false
				},
				"verificationState": "UNVERIFIED",
				"extraXmTankEnergy": 0
			},
			"controllingTeam": {
				"team": "RESISTANCE"
			},
			"avatar": {
				...
			}
		}],
		"apGains": [],
		"inventory": [
			[guid, 1481913704513, {
				"modResource": {
					"displayName": "Portal Shield",
					"stats": {
						"REMOVAL_STICKINESS": "0",
						"MITIGATION": "30"
					},
					"rarity": "COMMON",
					"resourceType": "RES_SHIELD"
				},
				"displayName": {
					"displayName": "Portal Shield",
					"displayDescription": "Mod which shields Portal from attacks."
				},
				"inInventory": {
					"playerId": agent_guid,
					"acquisitionTimestampMs": *
				}
			}],
			...
		],
		"deletedEntityGuids": [],
		"serverBlob": *,
		"refreshEntityGuids": [guid]
	}
}
```

#### With Glyphs

	API Request:
```json
{
	"params": {
		"glyphGameRequested": true,
		"spewInfluenceGlyphSequence": null,
		"userInputGlyphSequence": null,
		"clientBasket": {
			"clientBlob": *
		},
		"knobSyncTimestamp": 1481912295011,
		"energyGlobGuids": [],
		"portalGuid": guid,
		"location": value
	}
}
```

	API Response:

```json
{
	"result": {
		"glyphs": {
			"glyphSequence": [{
				"glyphOrder": "ejgahic"
			}, {
				"glyphOrder": "ga"
			}, {
				"glyphOrder": "jkgh"
			}, {
				"glyphOrder": "ejka"
			}, {
				"glyphOrder": "gkhijd"
			}],
			"isPrompted": false,
			"maxInputTimeMs": "15000",
			"timeToDemoMs": "500"
		}
	},
	"gameBasket": {
		*
	}
}
```

### collectItemsFromPortalWithGlyphResponse

	API URL: https://m-dot-betaspike.appspot.com/rpc/gameplay/collectItemsFromPortalWithGlyphResponse
	API Request:
```json
{
	"params": {
		"glyphGameRequested": false,
		"spewInfluenceGlyphSequence": {
			"inputTimeMs": 1233,
			"bypassed": false,
			"glyphSequence": [{
				"glyphOrder": "hgkj"
			}, {
				"glyphOrder": "gkh"
			}]
		},
		"userInputGlyphSequence": {
			"inputTimeMs": 8683,
			"bypassed": false,
			"glyphSequence": [{
				"glyphOrder": "fgah"
			}, {
				"glyphOrder": "egjihc"
			}, {
				"glyphOrder": "egahc"
			}, {
				"glyphOrder": "ega"
			}, {
				"glyphOrder": "efabhkjd"
			}]
		},
		"clientBasket": {
			"clientBlob": *
		},
		"knobSyncTimestamp": 1481912295011,
		"energyGlobGuids": [],
		"portalGuid": "3e2bcc15c58d486fae24e2ade2bf7327.16",
		"location": "01d553fe,0631e166"
	}
}
```

	API Response:

```json
{
	"result": {
		"addedGuids": ["a4f754d8e8f0448c88a6fe349f303ebc.5", "c07c542b6e0c4fea9b66b66fcc110c37.5", "c7995bba88c4442eac1ac2c8008d419c.5", "3264136a3f414efba8e88a401786bc25.5", "4d928a839c3546658abc025fb8170981.5", "65916b9be8944fe986abe47433ad4c75.5", "0e9b7f761b8f41d8b149447dfd10ca0d.5", "995fddb948bf41caab25ad1d0496e57a.5", "36ecac7a73ba43bfb21f6d4dbdd84a1f.5", "afe79b3bd18941b995557eb0d0bcef41.5"],
		"glyphResponse": {
			"glyphResponses": [true, true, true, true, true],
			"powerBoostInMillion": 1630000,
			"timeBonusBoostInMillion": 421133,
			"displayNames": ["PURSUE", "CONFLICT", "WAR", "ADVANCE", "CHAOS"],
			"bonusGuids": ["a4f754d8e8f0448c88a6fe349f303ebc.5", "c07c542b6e0c4fea9b66b66fcc110c37.5", "3264136a3f414efba8e88a401786bc25.5", "4d928a839c3546658abc025fb8170981.5", "65916b9be8944fe986abe47433ad4c75.5", "afe79b3bd18941b995557eb0d0bcef41.5"]
		},
		"baseHackMultiplier": "1"
	},
	"gameBasket": {
		"playerDamages": [],
		"gameEntities": [],
		"playerEntity": [*, 1481914255972, {
			"avatar": {
				*
			},
			"playerPersonal": {
				"ap": "14031565",
				"energy": 17100,
				"allowNicknameEdit": false,
				"allowFactionChoice": false,
				"verifiedLevel": 13,
				"mediaHighWaterMarks": {
					"General": 1809,
					"RESISTANCE": 1328,
					"ALIENS": 1327
				},
				"energyState": "XM_OK",
				"notificationSettings": {
					"shouldSendEmail": false,
					"maySendPromoEmail": false,
					"shouldPushNotifyForAtPlayer": true,
					"shouldPushNotifyForPortalAttacks": true,
					"shouldPushNotifyForInvitesAndFactionInfo": true,
					"shouldPushNotifyForNewsOfTheDay": true,
					"shouldPushNotifyForEventsAndOpportunities": true,
					"locale": "zh-Hans-CN"
				},
				"profileSettings": {
					"areMetricsPublic": false
				},
				"verificationState": "UNVERIFIED",
				"extraXmTankEnergy": 0
			},
			"controllingTeam": {
				"team": "RESISTANCE"
			}
		}],
		"apGains": [{
			"apTrigger": "GLYPH_HACK",
			"apGainAmount": "405"
		}],
		"inventory": [
			["a4f754d8e8f0448c88a6fe349f303ebc.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "EMITTER_A",
					"level": 8
				},
				"displayName": {
					"displayName": "Resonator",
					"displayDescription": "XM object used to power-up a Portal and align it to a Faction."
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255794"
				},
				"accessLevel": {
					"requiredLevel": 8,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 8
					}
				}
			}],
			["c07c542b6e0c4fea9b66b66fcc110c37.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "EMP_BURSTER",
					"level": 8
				},
				"displayName": {
					"displayName": "Xmp Burster",
					"displayDescription": "Exotic Matter Pulse weapons which can destroy enemy Resonators and Mods and neutralize enemy Portals."
				},
				"empWeapon": {
					"level": 8,
					"ammo": 1
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255779"
				},
				"accessLevel": {
					"requiredLevel": 8,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 8
					}
				}
			}],
			["c7995bba88c4442eac1ac2c8008d419c.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "EMITTER_A",
					"level": 7
				},
				"displayName": {
					"displayName": "Resonator",
					"displayDescription": "XM object used to power-up a Portal and align it to a Faction."
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255777"
				},
				"accessLevel": {
					"requiredLevel": 7,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 7
					}
				}
			}],
			["3264136a3f414efba8e88a401786bc25.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "EMP_BURSTER",
					"level": 7
				},
				"displayName": {
					"displayName": "Xmp Burster",
					"displayDescription": "Exotic Matter Pulse weapons which can destroy enemy Resonators and Mods and neutralize enemy Portals."
				},
				"empWeapon": {
					"level": 7,
					"ammo": 1
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255794"
				},
				"accessLevel": {
					"requiredLevel": 7,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 7
					}
				}
			}],
			["4d928a839c3546658abc025fb8170981.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "EMITTER_A",
					"level": 8
				},
				"displayName": {
					"displayName": "Resonator",
					"displayDescription": "XM object used to power-up a Portal and align it to a Faction."
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255779"
				},
				"accessLevel": {
					"requiredLevel": 8,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 8
					}
				}
			}],
			["65916b9be8944fe986abe47433ad4c75.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "EMITTER_A",
					"level": 8
				},
				"displayName": {
					"displayName": "Resonator",
					"displayDescription": "XM object used to power-up a Portal and align it to a Faction."
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255780"
				},
				"accessLevel": {
					"requiredLevel": 8,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 8
					}
				}
			}],
			["0e9b7f761b8f41d8b149447dfd10ca0d.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "EMITTER_A",
					"level": 8
				},
				"displayName": {
					"displayName": "Resonator",
					"displayDescription": "XM object used to power-up a Portal and align it to a Faction."
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255778"
				},
				"accessLevel": {
					"requiredLevel": 8,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 8
					}
				}
			}],
			["995fddb948bf41caab25ad1d0496e57a.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "POWER_CUBE",
					"level": 8
				},
				"displayName": {
					"displayName": "Power Cube",
					"displayDescription": "Store of XM which can be used to recharge Scanner."
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255778"
				},
				"accessLevel": {
					"requiredLevel": 8,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 8
					}
				},
				"powerCube": {
					"energy": 8000
				}
			}],
			["36ecac7a73ba43bfb21f6d4dbdd84a1f.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "EMITTER_A",
					"level": 8
				},
				"displayName": {
					"displayName": "Resonator",
					"displayDescription": "XM object used to power-up a Portal and align it to a Faction."
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255778"
				},
				"accessLevel": {
					"requiredLevel": 8,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 8
					}
				}
			}],
			["afe79b3bd18941b995557eb0d0bcef41.5", 1481914255823, {
				"resourceWithLevels": {
					"resourceType": "POWER_CUBE",
					"level": 8
				},
				"displayName": {
					"displayName": "Power Cube",
					"displayDescription": "Store of XM which can be used to recharge Scanner."
				},
				"inInventory": {
					"playerId": *,
					"acquisitionTimestampMs": "1481914255779"
				},
				"accessLevel": {
					"requiredLevel": 8,
					"failure": {
						"isAllowed": false,
						"requiredLevel": 8
					}
				},
				"powerCube": {
					"energy": 8000
				}
			}]
		],
		"deletedEntityGuids": [],
		"serverBlob": *,
		"refreshEntityGuids": [guid]
	}
}
```

All the parameters submitted and obtained are listed here, indicating which position is represented by the letters of the drawing.

From top to bottom, left to right, in order

	       a
	f            b
	    g     h
	       k
	    j     i
	e            c
	       d

That is, the ALL command is abcd(bottom) dfa clockwise from the topmost vertex; the Strong command is ghij clockwise from the top left corner; the center is k

# Concluding remarks

	Used two nights to count the API.
	Originally intended to also write a Game API library, but because it can not generate client-side blob and does not make sense to use later, Intel Map has been OK.
	In the course of the game, batch will be triggered when moving
	Get map updates are timed polling: getInventory (after all, you can also passcode to get items, all have to keep getting), getPaginatedPlexts, getObjectsInCells
