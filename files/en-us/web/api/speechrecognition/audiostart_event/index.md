---
title: "SpeechRecognition: audiostart event"
short-title: audiostart
slug: Web/API/SpeechRecognition/audiostart_event
page-type: web-api-event
browser-compat: api.SpeechRecognition.audiostart_event
---

{{APIRef("Web Speech API")}}

The **`audiostart`** event of the [Web Speech API](/en-US/docs/Web/API/Web_Speech_API) is fired when the user agent has started to capture audio for speech recognition.

## Syntax

Use the event name in methods like {{domxref("EventTarget.addEventListener", "addEventListener()")}}, or set an event handler property.

```js-nolint
addEventListener("audiostart", (event) => { })

onaudiostart = (event) => { }
```

## Event type

A generic {{DOMxRef("Event")}} with no added properties.

## Examples

You can use the `audiostart` event in an [`addEventListener`](/en-US/docs/Web/API/EventTarget/addEventListener) method:

```js
const recognition = new (SpeechRecognition || webkitSpeechRecognition)();

recognition.addEventListener("audiostart", () => {
  console.log("Audio capturing started");
});
```

Or use the `onaudiostart` event handler property:

```js
recognition.onaudiostart = () => {
  console.log("Audio capturing started");
};
```

## Specifications

{{Specifications}}

## Browser compatibility

{{Compat}}

## See also

- [Web Speech API](/en-US/docs/Web/API/Web_Speech_API)
