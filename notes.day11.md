# Interacting with the simulations

We've already looked at using dat.GUI to interact with our simulations. Today we'll look at using our mobile devices to add more interactive control. We'll start with a [simple simulation controlling fractional brownian noise](https://github.com/cs-imgd-420x-20a/fbm) to play with.

## Mobile Apps for OSC
Open Sound Control is a simple network protocol that is commonly used in the digital arts. It uses UDP as the underlying transport layer; to integrate it with our web app we'll need a simple node.js server to convert the OSC messages into WebSocket messages. Here are a few apps that output OSC messages:

1. [TouchOSC](https://hexler.net/products/touchosc) - free for android, $5 for iOS. The most popular OSC app for mobile devices. 
2. [oscHook](https://play.google.com/store/apps/details?id=com.hollyhook.oscHook&hl=en_US) - free, android only.
3. [mrmr](https://apps.apple.com/us/app/mrmr-osc-controller/id294296343) - free, ios only

There are lots of other OSC apps out there to consider, so feel free to browse other apps that look interesting.

## Listening for OSC in Node.js

Here's a [nice library to use for generating and receiving OSC messages](https://github.com/russellmcc/node-osc-min). You can create a simple node.js server that listens for OSC essages with the following (after running `npm i osc-min` to install):

```js
const udp = require('dgram');
const osc = require('osc-min')

const sock = udp.createSocket("udp4", function(msg, rinfo) {
  try {
    return console.log( osc.fromBuffer( msg ) )
  } catch (error) {
    return console.log( "installnvalid OSC packet", error )
  }
})

sock.bind(8000)
```

Start the server using node.js, and then make sure that the OSC app on your phone is pointed at the IP address of your computer and the correct port (8000). Both of these are critical! You will also *need to ensure that port 8000 is open on your computer*. You should be able to see messages in your terminal console (wherever you launched the node server from) after getting this setup correctly.

You could write your own server to take these messages and convert them to WebSocket messages from this point, but I have a library that mostly abstracts this away.
