---
layout: post
title:  "Promisifying Post Messages"
date:   2017-05-17
---

Your boss comes to you, and tells you that he wants you to build something that communicates with a web page inside an iframe.  
“Sounds simple!” – you reply.  
You read the details, and you find out that the web page you need to communicate with is on another origin and uses  [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)  for communication.  
“Ok.. PostMessage should be easy. You just emit an event on the iframe window, and receive another event as response”. What could possibly go wrong?

So, your start implementing you solution with postMessages and it seems something like this:

```javascript
window.addEventListener("message", (event) => {
    var apples = JSON.parse(event.data);
    alert(apples);
});
 
frame.contentWindow.postMessage(JSON.stringify({
    iCanHaz: "apples"
}), "*");
```

And it works fine.. until you have to do two simultaneous requests. The problem is that the event the window receives upon an incoming postMessage is always the same event – “message”, so you have to find a way to tell different responses.

The first, and most trivial solution, is to check the structure of the response, and according to the structure call different callbacks, or calling every callback with every response and filtering inside the callback.

So the implementation of the above suggestion would be something like this (disclaimer: this code is for demonstrational purposes only, in the real world I would write it completely differently)(note: notice that the postMessages are sent to different iFrames):

```javascript
callbacks = [];
 
window.addEventListener("message", (event) => {
    callbacks.forEach(callback => callback(event));
});
 
callbacks.push((event) => {
    const data = JSON.parse(event.data);
    if("apples" in data) {
        alert(`Apples: ${data.apples}`);
    }
});
 
callbacks.push((event) => {
    const data = JSON.parse(event.data);
    if("oranges" in data) {
        alert(`Oranges: ${data.oranges}`);
    }
});
 
frame.contentWindow.postMessage(JSON.stringify({
    iCanHaz: "apples"
}), "*");
 
anotherFrame.contentWindow.postMessage(JSON.stringify({
    iCanHaz: "oranges"
}), "*");
```

This code would work, but it’s completely uInscalable. What if someone would add a postMessage that would expect a similar pattern as another call?

Luckily there is a solution 🙂

If, for every frame your code wants to communicate with, you open another  **empty**  iframe without any src attribute (which would be in the same origin as you) you can programmatically open the real iframe inside that iframe, and then listen to the “message” event inside the intermediate iframe – you could be able to recognize which callback to call for every response.

It’s a bit hard to understand from reading, so let’s write some code:

```javascript
class PostMessenger {
    constructor(url) {
        const parentIFrame = this._createIFrame();
        parentIFrame.contentWindow.addEventListener("message", event => this._handleIncomingMessage(event));
        parentIFrame.contentDocument.body.innerHTML = `<iframe src="${url}" id="inner_frame"></iframe>`;
        this._innerWindow = parentIFrame.contentDocument.getElementById("inner_frame").contentWindow;
    }
 
    sendMessage(message) {
        return new Promise(resolve => {
            this._promiseResolver = resolve;
            this._innerWindow.postMessage(message, "*");
        });
    }
 
    _handleIncomingMessage(event) {
        this._promiseResolver(event);
    }
 
    _createIFrame() {
        return $("<iframe></iframe>").appendTo("body")[0];
    }
}
```

And now, we can just use this class like this:

```javascript
const iframeMessenger = new PostMessenger("http://localhost:8000/1");
const message = JSON.stringify({type: "fruits"})
iframeMessenger.sendMessage(message).then(event => {
    const data = JSON.parse(event.data);
    console.log(`I got some fruits: ${data}`);
});
 
const anotherIframeMessenger = new PostMessenger("http://localhost:8000/2");
const message = JSON.stringify({type: "vegetables"})
anotherIframeMessenger.sendMessage(message).then(event => {
    const data = JSON.parse(event.data);
    console.log(`I got some vegetables: ${data}`);
});
```