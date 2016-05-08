# Bionico Dev Kit

_This is a work in progress, contributions are welcome. please PM me at
constantin [at] hackerloop.com for further explanations_

Bionico's dev kit makes it possible to quickly prototype and distribute
software that runs on any limb prosthesis.

This guide focuses on Bionico's case, the limb being the hand, and the
prosthesis hand being the one from openbionics. But the dev kit should
easily be extended to other limbs and other prosthesis models.

## Base principle

All features of the dev kit are accessible via its Websocket connection.
A Websocket is a standard for a persistent connection originally developped for the web,
allowing two programs to exchange messages.

The two main advantages being easiness of use and wide availability of
libraries in all languages.

The messages exchanged on the dev kit are in JSON format.

Developping an application or adding hardware components to the dev kit
implies to connect to its websocket and interact with the rest of the
system by sending and/or listening for events and actions.

![Main schema](http://i.imgur.com/SI4LcJ2.png)

## Setup the dev kit

What you need:

- Ada hand from [openbionics](http://www.openbionics.com/shop/ada)
- RaspberryPI or any other linux based ARM computer
- (Wifi dongle)[https://www.raspberrypi.org/products/usb-wifi-dongle/]

Behind the dev kit there is an open source project called [rotonde](https://github.com/HackerLoop/rotonde),
rotonde provides a middleware to handle the websocket message routing,
and a set of modules to easily prototype in any languages.

Rotonde also provides a [CLI](https://github.com/HackerLoop/rotonde-cli) (CLI == Command Line Interface),
to easily install modules on a linux based ARM hardware.

First follow the instructions [here](https://github.com/MHKit/BionicoDevKit-QuickStart/blob/master/RASPBIAN.md),
and then install the rotonde CLI by following [these instructions](https://github.com/HackerLoop/rotonde-cli/blob/master/README.md) (just keep in mind that the raspberrypi is now called `bionicodevkit.local` instead of `raspberrypi.local`).

_Work in progress, comming soon_

## First contact

This first contact aims to show the most basic usage to the Dek kit, no
programming skills required.

We are going to connect to the dev kit thanks to a simple chrome
extension.
The chrome extension makes it possible to connect to the websocket and
start exchanging messages with the dev kit.

Install this [chrome
extension](https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo?hl=en).

In the extension, connect to the dev kit by entering the following url:

```
ws://bionicodevkit.local:4224/
```

Click the `Open` button.

Copy-paste the following JSON message in the extension's console:

```js
{
  "type": "action",
  "payload": {
    "identifier": "HAND_FINGERS",
    "data": {
      "fingers": [
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1}
      ]
    }
  }
}
```

The hand will open its fingers.

What just happenned here is that we send an action called
`HAND_FINGERS`, this actions is then handled by the module running in
the openbionics firmware.

![HAND_FINGERS path](http://i.imgur.com/WyTvfwS.png) 

The `HAND_FINGERS` action provides a serie of positions and speeds
associated with each fingers (`position` and `speed` in the example).
for each fingers, from the thumb to pinky, you can define a position
from 0 to 1, and a speed from 0 to 1.

Try to modify the messages, and start experimenting with the hand's behaviour:

```js
{
  "type": "action",
  "payload": {
    "identifier": "HAND_FINGERS",
    "data": {
      "fingers": [
        {"position": 0, "speed": 1},
        {"position": 1, "speed": 1},
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1}
      ]
    }
  }
}
```

Congratulations, you now have control over the hand's fingers :)

## First code

The chrome extension is a convenient tool to test the available
features.

But the websocket can also be used by code, and has to advantage to be
language agnostic, which means you can use any language to interact with
it.

## Javascript

The Dev kit provides a library to make it easier to use with Javascript,
that's why it is the language used in this guide.

You are going to need [nodeJS installed](https://nodejs.org/en/download/) on you computer to test these
codes.

The idea of this first code is to control the hand with a myo armband.

Start by installing the library to ease the usage of the dev kit:

```sh
npm install --save HackerLoop/rotonde-client.js
```

Create a file called `index.js` and copy-paste the colling content:

```js

'use strict'

const _ = require('lodash');

const client =
require('rotonde-client/node/rotonde-client')('ws://bionicodevkit.local:4224');

client.onReady(() => {
  console.log('Connected');
});

client.connect();

```

You can now launch the previous script with:

```sh
node index.js
```

You should get an output as follows:

```sh
$ node index.js
Connected
```

We are now ready to start receiving myo events, they will contain the
gestures detected by the myo's muscular sensors.
The myo event related to poses is called `MYO_POSE_EDGE`.

![MYO_ARMBAD event](http://i.imgur.com/aVABc1o.png)

Thanks to the Javascript library, we can easily subscribe to an event:

```js

'use strict'

const _ = require('lodash');

const client =
require('rotonde-client/node/rotonde-client')('ws://bionicodevkit.local:4224');

// ===================
// Code to receive a MYO_POSE_EDGE event

client.eventHandlers.attach('MYO_POSE_EDGE', (event) => {
  console.log(event.identifier);
  
  // Insert the hand's control code here
});

// ===================

client.onReady(() => {
  console.log('Connected');
});

client.connect();

```

The `event` parameter for the `MYO_POSE_EDGE` is as follows:

```js

{
  "identifier": "MYO_POSE_EDGE",
  "data": {
    "pose": "", // Name for the pose
    "edge": "", // Indicates whether it is the start or end of the pose
  }
}

```

Let's start with something simple, if the pose is `wave_left`, we want
the hand to be closed, if the pose is `wave_right`, we want it to be
openned.

```js

client.eventHandlers.attach('MYO_POSE_EDGE', (event) => {
  console.log(event.identifier);
  
  // Insert the hand's control code here
  if (event.data.pose == 'wave_right') {
    client.sendAction('HAND_FINGERS', { // same structure as the data field sent through the chrome extension
      "fingers": [
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1},
        {"position": 0, "speed": 1}
      ]
    });
  } else {
    client.sendAction('HAND_FINGERS', { // same as above, but the positions are all set to 1
      "fingers": [
        {"position": 1, "speed": 1},
        {"position": 1, "speed": 1},
        {"position": 1, "speed": 1},
        {"position": 1, "speed": 1},
        {"position": 1, "speed": 1}
      ]
    });
  }
});

```

Of course this is a strict minimum, and it's not even practical for
Bionico, the myo armbad is not very suitable for amputees, but we could
easily add another interface by plugging another type of device, like
the [myoware](https://www.adafruit.com/product/2699).
All you need is expose its features as events or actions.
