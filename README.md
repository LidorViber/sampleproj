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

### A simple Echo Bot:
Firstly, let's *import and configure* our bot:
```python
the code will be here
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

## API
### Viber Bot
`require('@viber/viber-bot').Bot`

An event emitter, emitting events [described here](#onEvent).

* [ViberBot](#ViberBot)
    * [new ViberBot()](#new-ViberBot())
    * [.getProfile()](#getProfile) ⇒ `promise.JSON`
    * [.setWebhook(url)](#setWebhook) ⇒ `promise.JSON`
    * [.sendMessage(userProfile, messages, [optionalTrackingData])](#sendMessage) ⇒ `promise.ARRAY`
    * [.on(handler)](#onEvent)
    * [.onTextMessage(regex, handler)](#onTextMessage) : `handler` = [`TextMessageHandlerCallback`](#TextMessageHandlerCallback)
    * [.onError(handler)](#onError) : `handler` = [`ErrorHandlerCallback`](#ErrorHandlerCallback)
    * [.onConversationStarted(userProfile, onFinish)](#onConversationStarted) : `onFinish` = [`ConversationStartedOnFinishCallback`](#ConversationStartedOnFinishCallback)
    * [.onSubscribe(handler)](#onSubscribe) : `handler` = [`ResponseHandlerCallback`](#ResponseHandlerCallback)
    * [.onUnsubscribe(handler)](#onUnsubscribe) : `handler` = [`ResponseHandlerCallback`](#ResponseHandlerCallback)
    * [.middleware()](#middleware)


### new ViberBot()

| Param | Type | Description |
| --- | --- | --- |
| logger | `object` | Winston logger |
| options.authToken | `string` | Viber Auth Token  |
| options.name | `string` | Your BOT Name |
| options.avatar | `string` | Avatar URL. **No more than 100kb.** |

<a name="onEvent"></a>
### bot.on(handler)
`require('@viber/viber-bot').Events`

| Param | Type |
| --- | --- |
| handler | `EventHandlerCallback` |
| message | [`Message Object`](#MessageObject) |
| response | [`Response Object`](#ResponseObject) |
| err | `Error Object` |

Subscribe to events:
* MESSAGE_RECEIVED (Callback:  `function (event, message, response) {}`)
* SUBSCRIBED (Callback:  `function (event, response) {}`)
* UNSUBSCRIBED (Callback:  `function (event, response) {}`)
* CONVERSATION_STARTED (Callback:  `function (event, response) {}`)
* ERROR (Callback:  `function (event, err) {}`)

**Example**
```js
bot.on(BotEvents.MESSAGE_RECEIVED, (event, message, response) => ... );
bot.on(BotEvents.CONVERSATION_STARTED, (event, message, response) => ... );
bot.on(BotEvents.ERROR, (event, err) => ... );
bot.on(BotEvents.UNSUBSCRIBED, (event, response) => ... );
bot.on(BotEvents.SUBSCRIBED, (event, response) =>
    response.send(`Thanks for subscribing, ${response.userProfile.name}`));
```

<a name="getProfile"></a>
### bot.getProfile()
Returns a `promise.JSON` ([With the following JSON](https://developers.viber.com/customer/en/portal/articles/2541122-get-account-info?b_id=15145)). **Example**
```js
bot.getProfile().then(response => console.log(`Public Account Named: ${response.name}`));
```

<a name="setWebhook"></a>
### bot.setWebhook(url)
| Param | Type | Description |
| --- | --- | --- |
| url | `string` | Trusted SSL Certificate |

Returns a `promise.JSON`. **Example**
```js
bot.setWebhook("https://my.bot/incoming").then(() => yourBot.doSomething()).catch(err => console.log(err));
```

<a name="sendMessage"></a>
### bot.sendMessage(userProfile, messages, [optionalTrackingData])
| Param | Type | Description |
| --- | --- | --- |
| userProfile | [`UserProfile`](#UserProfile) | `UserProfile` object |
| Messages | `object or array` | Can be `Message` object or array of `Message` objects |
| [optionalTrackingData] | `JSON` | Optional. JSON Object. Returned on every message sent by the client |

*Note*: When passing array of messages to sendMessage, messages will be sent by explicit order (the order which they were given to the sendMessage method). The library will also cancel all custom keyboards except the last one, sending only the last message keyboard.

Returns a `promise.ARRAY` array of message tokens. **Example**
```js
// single message
const TextMessage = require('@viber/viber-bot').Message.Text;
bot.sendMessage(userProfile, new TextMessage("Thanks for shopping with us"));

// multiple messages
const UrlMessage  = require('@viber/viber-bot').Message.Url;
bot.sendMessage(userProfile, [
    new TextMessage("Here's the product you've requested:"),
    new UrlMessage("http://my.ecommerce.site/product1"),
    new TextMessage("Shipping time: 1-3 business days")
]);
```

<a name="middleware"></a>
### bot.middleware()
Returns a middleware to use with `http/https`. **Example**
```js
const https = require('https');
https.createServer({ key: ... , cert: ... , ca: ... }, bot.middleware()).listen(8080);
```

<a name="onTextMessage"></a>
### bot.onTextMessage(regex, handler)
| Param | Type |
| --- | --- |
| regex | `regular expression` |
| handler | [`TextMessageHandlerCallback`](#TextMessageHandlerCallback) |

<a name="TextMessageHandlerCallback"></a>
##### TextMessageHandlerCallback: `function (message, response) {}`
**Example**
```js
bot.onTextMessage(/^hi|hello$/i, (message, response) =>
    response.send(new TextMessage(`Hi there ${response.userProfile.name}. I am ${bot.name}`)));
```

<a name="onError"></a>
### bot.onError(handler)
| Param | Type |
| --- | --- |
| handler | [`ErrorHandlerCallback`](#ErrorHandlerCallback) |

<a name="ErrorHandlerCallback"></a>
##### ErrorHandlerCallback: `function (event, err) {}`
**Example**
```js
bot.onError((event, err) => logger.error(err));
```

<a name="onConversationStarted"></a>
### bot.onConversationStarted(userProfile, onFinish)
| Param | Type |
| --- | --- |
| userProfile | [`UserProfile`](#UserProfile) |
| onFinish | [`ConversationStartedOnFinishCallback`](#ConversationStartedOnFinishCallback) |

*Note*: onConversationStarted event happens once when A Viber client opens the conversation with your bot for the first time. Callback accepts `null` and [`MessageObject`](#MessageObject) only. Otherwise, an exception is thrown.

<a name="ConversationStartedOnFinishCallback"></a>
##### ConversationStartedOnFinishCallback: `function (responseMessage) {}`
**Example**
```js
bot.onConversationStarted((userProfile, onFinish) =>
	onFinish(new TextMessage(`Hi, ${userProfile.name}! Nice to meet you.`)));
```

<a name="onSubscribe"></a><a name="onUnsubscribe"></a>
### bot.onSubscribe(handler) & bot.onUnsubscribe(handler)
| Param | Type |
| --- | --- |
| handler | [`ResponseHandlerCallback`](#ResponseHandlerCallback) |

<a name="ResponseHandlerCallback"></a>
##### ResponseHandlerCallback: `function (event, response) {}`
**Example**
```js
bot.onSubscribe((event, response) => console.log(`Subscribed: ${response.userProfile.name}`));
bot.onUnsubscribe((event, response) => console.log(`Unsubscribed: ${response.userProfile.name}`));
```

<a name="ResponseObject"></a>
### Response object
Members:

| Param | Type | Notes |
| --- | --- | --- |
| userProfile | [`UserProfile`](#UserProfile) | --- |

* [Response](#Response)
    * [.send(messages, [optionalTrackingData])](#sendMessage) ⇒ `promise.JSON`

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
