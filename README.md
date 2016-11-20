# Viber Bot API
Use this library to communicate with the Viber API to develop a bot for [Viber](https://developers.viber.com/).
Please visit [Getting Started](https://developers.viber.com/customer/en/portal/articles/2567874-getting-started?b_id=15145) guide for more information about Viber API.

## License
This library is released under the terms of the MIT license. See [License](LICENSE.md) for more information.

## Library Prerequisites
* Node >= 5.0.0
* Winston >= 2.0.0
* [Viber Auth Token](https://developers.viber.com/customer/en/portal/articles/2554141-create-a-public-account?b_id=15145).
* Certification - You'll need a trusted (ca.pem) certificate, not self-signed. You can find one at [Let's Encrypt](https://letsencrypt.org/) or buy one.

## Let's get started!
### Installing
Creating a basic Viber bot is simple:

1. Import `viber.api` library to your project
2. Create a Public Account and use the API key from https://developers.viber.com/
3. Configure your bot as described in the documentation below
4. Start your web server
5. Call set_webhook(url) with your webserver url

## A simple Echo Bot:
### Firstly, let's *import and configure* our bot:
```python
from viber.api.api import Api
from viber.api.bot_configuration import BotConfiguration

bot_configuration = BotConfiguration(
	name='PythonSampleBot',
	avatar='http://site.com/avatar.jpg',
	auth_token='451798a889a17401-865360a5474b3993-8fe73f00b019c611'
)
viber = Api(bot_configuration)
```

### create an https server
next thing you should do is starting a https server.
and yes, as we said in the [Library Prerequisites](#Library Prerequisites) it has to be https server.
create a server however you like, for example with Flask:

```python
from flask import Flask, request, Response

app = Flask(__name__)

@app.route('/incoming', methods=['POST'])
def incoming():
	logger.debug("received request. post data: {0}".format(request.get_data()))
	# handle the request here
	return Response(status=200)
	
context = ('server.crt', 'server.key')
app.run(host='0.0.0.0', port=443, debug=True, ssl_context=context)

```

### Setting a webhook
After the server is up and kickin' you can set a webook.
Viber will push messages sent to this URL. Webserver should be internet-facing.

```python
viber.set_webhook('https://mybotwebserver.com/incoming')
```

### Logging
This library uses the standard python logger.
if you want to see its logs you can configure the logger

```python
logger = logging.getLogger()
logger.setLevel(logging.DEBUG)
handler = logging.StreamHandler()
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)
```

### Do you supply a basic types of messages ?
Well, funny you ask. Yes we do.
All the Message types are located in 'viber.api.messages' package.
Here's some examples

```python
from viber.api.messages.text_message import TextMessage
from viber.api.messages.contact_message import ContactMessage
from viber.api.messages.data_types.contact import Contact
from viber.api.messages.picture_message import PictureMessage
from viber.api.messages.video_message import VideoMessage

# creation of text message
text_message = TextMessage(text="sample text message!")

# creation of contact message
contact = Contact(name="viber user", phone_number="0123456789")
contact_message = ContactMessage(contact=contact)

# creation of picture message
picture_message = PictureMessage(text="my son started to eat all alone!", media="http://site.com/son.jpg")

# creation of video message
video_message = VideoMessage(media="http://mediaserver.com/video.mp4", size=4324)
```

Have you noticed how we created the TextMessage? There's a all bunch of message types you should get familiar with, [here's a list of them](https://developers.viber.com/customer/en/portal/articles/2632255-send-message?b_id=15145).
Every one of them is already modeled:

* [Text Message](#TextMessage)
* [Url Message](#UrlMessage)
* [Contact Message](#ContactMessage)
* [Picture Message](#PictureMessage)
* [Video Message](#VideoMessage)
* [Location Message](#LocationMessage)
* [Sticker Message](#StickerMessage)

Creating them is easy! Every message object has it's own unique constructor corresponding to it's API implementation, click on them to see it!
Check out the full API documentation for more advanced uses.

### Let's add it all up and reply with a message!
```python
from flask import Flask, request, Response
from viber.api.api import Api
from viber.api.bot_configuration import BotConfiguration
from viber.api.messages import VideoMessage
from viber.api.messages.text_message import TextMessage
import logging

from viber.api.viber_requests import ViberConversationStartedRequest
from viber.api.viber_requests import ViberFailedRequest
from viber.api.viber_requests import ViberMessageRequest
from viber.api.viber_requests import ViberSubscribedRequest
from viber.api.viber_requests import ViberUnsubscribedRequest

app = Flask(__name__)
viber = Api(BotConfiguration(
	name='PythonSampleBot',
	avatar='http://site.com/avatar.jpg',
	auth_token='451798a889a17401-865360a5474b3993-8fe73f00b019c611'
))


@app.route('/incoming', methods=['POST'])
def incoming():
	logger.debug("received request. post data: {0}".format(request.get_data()))
	# every viber message is signed, you can verify the signature using this method
	if not viber.verify_signature(request.get_data(), request.headers.get('X-Viber-Content-Signature')):
		return Response(status=403)
	
	# this library supplies a simple way to receive a request object
	viber_request = viber.parse_request(request.get_data())

	if isinstance(viber_request, ViberMessageRequest):
		message = viber_request.get_message()
		# lets echo back 
		viber.send_messages(viber_request.get_sender().get_id(), [
			message
		])
	elif isinstance(viber_request, ViberSubscribedRequest):
		viber.send_messages(viber_request.get_user().get_id(), [
			TextMessage(text="thanks for subscribing!")
		])
	elif isinstance(viber_request, ViberFailedRequest):
		logger.warn("client failed receiving message. failure: {0}".format(viber_request))

	return Response(status=200)

if __name__ == "__main__":
	context = ('server.crt', 'server.key')
	app.run(host='0.0.0.0', port=443, debug=True, ssl_context=context)

```

As you can see there's a bunch of Request types [here's a list of them](# RequestTypes).

## API
### Api
`from viber.api.api import Api`

* [Api](#Api)
    * [init(bot_configuration)](#new-Api())
    * [.set_webhook(url, webhook_events)](#set_webhook) ⇒ `None`
    * [.unset_webhook()](#unset_webhook) ⇒ `None`
    * [.get_account_info()](#get_account_info) ⇒ `object`
    * [.verify_signature(request_data, signature)](#verify_signature) ⇒ `boolean`
    * [.parse_request(request_data)](#parse_request) ⇒ `ViberRequest`
    * [.send_messages(to, messages)](#send_messages) ⇒ `list of message tokens sent`

<a name="new-Api()"></a>
### new Api()

| Param | Type | Description |
| --- | --- | --- |
| bot_configuration | `object` | [BotConfiguration](#BotConfiguration) |

<a name="set_webhook"></a>
### Api.set_webhook(url)
| Param | Type | Description |
| --- | --- | --- |
| url | `string` | Your webserver url |
| webhook_events | `list` | optional list of subscribed events |

Returns `None`. **Example**
```python
viber.set_webhook('https://mywebserver.com/incoming')
```

<a name="unset_webhook"></a>
### Api.unset_webhook()
Returns `None`. **Example**
```python
viber.unset_webhook()
```

<a name="
">get_account_info</a>
### Api.get_account_info()
Returns an `object` ([Formmated from the following JSON](https://developers.viber.com/customer/en/portal/articles/2541122-get-account-info?b_id=15145)). **Example**
```python
account_info = viber.get_account_info()
```

<a name="verify_signature"></a>
### Api.verify_signature(request_data, signature)
| Param | Type | Description |
| --- | --- | --- |
| request_data | `string` | the post data from request |
| signature | `string` | sent as header `X-Viber-Content-Signature` |


Returns a `boolean` suggesting if the signature is valid. **Example**
```python
if not viber.verify_signature(request.get_data(), request.headers.get('X-Viber-Content-Signature')):
	return Response(status=403)
```

<a name="parse_request"></a>
### Api.parse_request(request_data)
| Param | Type | Description |
| --- | --- | --- |
| request_data | `string` | the post data from request |

Returns a `ViberRequest` object. **Example**

There's a list of [ViberRequest objects](#ViberRequest)

```python
viber_request = viber.parse_request(request.get_data())
```

<a name="send_messages"></a>
### Api.send_messages(to, messages)
| Param | Type | Description |
| --- | --- | --- |
| to | `string` | receiver viberId |
| messages | `list` | list of `Message` objects |

Returns `list` of message tokens of the messages sent. **Example**
```python
tokens = viber.send_messages(to=viber_request.get_sender().get_id(),
			     messages=[TextMessage(text="sample message")])
```


<a name="ViberRequest"></a>
### Request object
Members:

| Param | Type | Notes |
| --- | --- | --- |
| event_type | `string` | according to [EventTypes](#EventTypes) |
| timestamp | `long` | Epoch of request time |

* [ViberRequest](#ViberRequest)
    * .get_event_type() ⇒ `string`
    * .get_timestamp() ⇒ `long`
    
All of the Request objects listed below are [listed in Viber developers site](https://developers.viber.com/customer/en/portal/articles/2541267-callbacks?b_id=15145)
#### ViberConversationStartedRequest object
inherits from [ViberRequest](#ViberRequest)

Members:

| Param | Type | Notes |
| --- | --- | --- |
| event_type | `string` | always equals to the value of EventType.CONVERSATION_STARTED |
| message_token | `string` |  |
| type | `string` |  |
| context | `string` |  |
| user | `UserProfile` | the user started the conversation [UserProfile](#UserProfile) |

* [ViberRequest](#ViberRequest)
    * get_message_token() ⇒ `string`
    * get_type() ⇒ `string`
    * get_context() ⇒ `string`
    * get_user() ⇒ `UserProfile`
    

<a name="UserProfile"></a>
### UserProfile object
Members:

| Param | Type | Notes |
| --- | --- | --- |
| id | `string` | --- |
| name | `string` | --- |
| avatar | `string` | Avatar URL |
| country | `string` | **currently set in CONVERSATION_STARTED event only** |
| language | `string` | **currently set in CONVERSATION_STARTED event only** |

<a name="MessageObject"></a>
### Message Object
```javascript
const TextMessage     = require('@viber/viber-bot').Message.Text;
const UrlMessage      = require('@viber/viber-bot').Message.Url;
const ContactMessage  = require('@viber/viber-bot').Message.Contact;
const PictureMessage  = require('@viber/viber-bot').Message.Picture;
const VideoMessage    = require('@viber/viber-bot').Message.Video;
const LocationMessage = require('@viber/viber-bot').Message.Location;
const StickerMessage  = require('@viber/viber-bot').Message.Sticker;
```

**Common Members for `Message` interface**:

| Param | Type | Description |
| --- | --- | --- |
| timestamp | `long` | Epoch time |
| token | `long` | Sequential message token |
| trackingData | `JSON` | JSON Tracking Data from Viber Client |

**Common Constructor Arguments `Message` interface**:

| Param | Type | Description |
| --- | --- | --- |
| optionalKeyboard | `JSON` | [Writing Custom Keyboards](https://developers.viber.com/customer/en/portal/articles/2567880-keyboards?b_id=15145) |
| optionalTrackingData | `JSON` | Data to be saved on Viber Client device, and sent back each time message is recived |

<a name="TextMessage"></a>
#### TextMessage object
| Member | Type
| --- | --- |
| text | `string` |
```javascript
const message = new TextMessage(text, [optionalKeyboard], [optionalTrackingData]);
console.log(message.text);
```

<a name="UrlMessage"></a>
#### UrlMessage object
| Member | Type
| --- | --- |
| url | `string` |
```javascript
const message = new UrlMessage(url, [optionalKeyboard], [optionalTrackingData]);
console.log(message.url);
```

<a name="ContactMessage"></a>
#### ContactMessage object
| Member | Type
| --- | --- |
| contactName | `string` |
| contactPhoneNumber | `string` |
```javascript
const message = new ContactMessage(contactName, contactPhoneNumber, [optionalKeyboard], [optionalTrackingData]);
console.log(`${message.contactName}, ${message.contactPhoneNumber}`);
```

<a name="PictureMessage"></a>
#### PictureMessage object
| Member | Type
| --- | --- |
| url | `string` |
| text | `string` |
| thumbnail | `string` |
```javascript
const message = new ContactMessage(url, [optionalText], [optionalThumbnail], [optionalKeyboard], [optionalTrackingData]);
console.log(`${message.url}, ${message.text}, ${message.thumbnail}`);
```

<a name="VideoMessage"></a>
#### VideoMessage object
| Member | Type
| --- | --- |
| url | `string` |
| size | `int` |
| thumbnail | `string` |
| duration | `int` |
```javascript
const message = new VideoMessage(url, size, [optionalThumbnail], [optionalDuration], [optionalKeyboard], [optionalTrackingData]);
console.log(`${message.url}, ${message.size}, ${message.thumbnail}, ${message.duration}`);
```

<a name="LocationMessage"></a>
#### LocationMessage object
| Member | Type
| --- | --- |
| latitude | `float` |
| longitude | `float` |
```javascript
const message = new LocationMessage(latitude, longitude, [optionalKeyboard], [optionalTrackingData]);
console.log(`${message.latitude}, ${message.longitude}`);
```

<a name="StickerMessage"></a>
#### StickerMessage object
| Member | Type
| --- | --- |
| stickerId | `int` |
```javascript
const message = new StickerMessage(stickerId, [optionalKeyboard], [optionalTrackingData]);
console.log(message.stickerId);
```

## Useful links:
* Writing a custom keyboard JSON [described here](https://developers.viber.com/customer/en/portal/articles/2567880-keyboards?b_id=15145).
* [Forbidden file formats list](https://developers.viber.com/customer/en/portal/articles/2541358-forbidden-file-formats?b_id=15145).
* List of [Error Codes](https://developers.viber.com/customer/en/portal/articles/2541337-error-codes?b_id=15145).
* List of [Events and Callbacks](https://developers.viber.com/customer/en/portal/articles/2541267-callbacks?b_id=15145).
