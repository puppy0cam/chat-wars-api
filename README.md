
# Chat Wars: The API

***This documentation is not completely up to date, and it is recommended that you visit [The official documentation here](https://chatwars.github.io/chatwars-api-docs/).***
You can still use it for learning the very basics, but beyond authentication, please use the [official documentation](https://chatwars.github.io/chatwars-api-docs/)

Within the world of Chat Wars, you can find an API. This API is unlike any other, because of the simple fact that that you cannot simply just make an HTTP request to do things with it. The API is run on a system known as RabbitMQ, but the members of the Developer's Castle refer to it as RabshitMQ.
There are two version for the API. one is for [@chtwrsbot (EU)](http://t.me/chtwrsbot), the other is for [@ChatWarsBot (CW3)](http://t.me/ChatWarsBot).
## Version History
### Version 0.3
The initial release of the API
* `createAuthCode` method added
* `grantToken` method added
* `authorizePayment` method added
* `pay` method added
* `payout` method added
* `getInfo` method added
* `deals` queue added
* `offers` queue added
#### Version 0.3.1
* Modified result code from `OK` to `Ok` _1_
### Version 0.4
* `requestProfile` method added
* `requestStock` method added
* `authAdditionalOperation` method added
* `grantAdditionalOperation` method added
* `sex_digest` queue added _2_
### Version 0.5
* `yellow_pages` queue added _2_
* more data added to `requestProfile` method
### Version 0.6
* `wantToBuy` method added
* `InvalidToken` response fixed
## Using the API
Rather than the HTTP protocol that follows a REST patttern, the Chat Wars API takes a different approach.
### How you would expect it to work
You send a request, and the API responds to that specific request
### How it actually works
You send a request, and the API send the data. You have no way to know what request the server is responding to unless it is specifically stated as such.

---
### Requesting Access
To access the Chat Wars API, you must first obtain a login from the feedback bot.
* To obtain access to [@chtwrsbot (EU)](http://t.me/chtwrsbot)'s API, You must request a login from [@ChatWarsFeedbackBot](http://t.me/ChatWarsFeedbackBot)
* To obtain access to [@ChatWarsBot (CW3)](http://t.me/ChatWarsBot)'s API, You must request a login from [@ChatWarsRuFeedbackBot](http://t.me/ChatWarsRuFeedbackBot)

When requesting access initially, please follow the following format:
```plaintext
#api
name of application: my amazing application
purpose: Make the players have an amazing experience

Requested queues:
sex_digest
deals
offers
yellow_pages

Default permissions:
 - request valuables transfer from you
 - transfer valuables to you
 - issue a wtb/wts/rm commands on your behalf
 - read your profile information
 - read your stock info

Do you want pouch transactions enabled: yes
```
If you do not receive a response from feedback within the next 24 hours, please do join the [Developer's Castle](https://t.me/cwapi) group and give [@I_Rony](tg://user?id=89886125) a ping complaining with the following message: `OI RUBBISH BIN COULD I GET A RESPONSE?`

---
To request modification to your application, send the relevant feedback bot a using the following format:
```plaintext
#api
Request of modification to my app:
name of application: my amazing application

Queues:
remove yellow_pages
add deals

Default Permissions:
remove - request valuables transfer from you
remove - transfer valuables to you
add - read your profile information
add - read your stock info

Access to pouch transactions: removed
```
---
### Connecting to the API
to access the API for [@chtwrsbot (EU)](http://t.me/chtwrsbot), your hostname will be `api.chatwars.me`
to access the API for [@ChatWarsBot (CW3)](http://t.me/ChatWarsBot), your hostname will be `api.chtwrs.com`

Use your AMQP client of choice to connect to the rabbitMQ instance, if your client accepts a connection string, you can use a connection URL of `amqps://app_name:password@hostname:5673/`, while replacing the relevant parts to fit which API you are connecting to as well as your app credentials.
If your AMQP library does not support this string, this is the data you will need to input
```
login: app_name
password: app_password
hostname: either api.chatwars.me or api.chtwrs.com
port: 5673
protocol: amqps
```
### Using the API
When you try to consume or publish you must always prefix the queue/exchange/routing key with your application login followed by an underscore. When you are connected to the API, you should never have to bind any queues on your own. simply consume and publish to the relevant places.
#### Methods
To use a method in the API, you must consume from your input queue that is automatically received with your application. 
##### receiving responses
you can access responses to requests by consuming from the `app_name_i` queue, or under the special case that [@I_Rony](tg://user?id=89886125) requests for you to test an API update, you must consume from `app_name_ti`.
When you receive a response, the response is automatically deleted from your queue, unless you consume with the `noAck` flag set to `false`. In which case, you must manually `ack` the message received from the API.
The API can currently contain a timestamp of the response in the message properties for the scenario where you need the timestamp because of your consumption process going offline.
##### sending requests
You can send a request to the API by publshing your request to the `app_name_ex` exchange with the `app_name_o` routing key, or under the special case that [@I_Rony](tg://user?id=89886125) requests for you to test an API update, you must use the `app_name_to` routing key instead.

---
#### Queues
In a REST scenario, these would be provided as read up to request, however this is not REST, and that comes with a few advantages in this case. AMQP is an event based system designed for asynchronous processing, and this means that rather than specifically request the data, you can receive it as it happens.
##### using a queue
To use a queue, you must consume from `app_name_queue`. for example, to consume from the deals queue with an app called `my_awesome_app`, you must consume from `my_awesome_app_deals`.
When you receive a message from a queue, the message is automatically deleted from your queue, unless you consume with the `noAck` flag set to `false`. In which case, you must manually `ack` the message received from the API.
The API can currently contain a timestamp of the event in the message properties for the scenario where you need the timestamp because of your consumption process going offline.
## References
1. case of response code was modified to make it less likely to be misinterpreted.
2. These queues send data on an interval of around every 5 minutes
