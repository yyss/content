---
title: "Example and tutorial: Simple synth keyboard"
slug: Web/API/Web_Audio_API/Simple_synth
page-type: guide
---

{{DefaultAPISidebar("Web Audio API")}}

This article presents the code and working demo of a video keyboard you can play using the mouse. The keyboard allows you to switch among the standard waveforms as well as one custom waveform, and you can control the main gain using a volume slider beneath the keyboard. This example makes use of the following Web API interfaces: {{domxref("AudioContext")}}, {{domxref("OscillatorNode")}}, {{domxref("PeriodicWave")}}, and {{domxref("GainNode")}}.

Because {{domxref("OscillatorNode")}} is based on {{domxref("AudioScheduledSourceNode")}}, this is to some extent an example for that as well.

## The video keyboard

### HTML

There are three primary components to the display for our virtual keyboard. The first is the musical keyboard itself. We draw this in a pair of nested {{HTMLElement("div")}} elements so that we can make the keyboard horizontally scrollable if all the keys don't fit on the screen, without having them wrap around.

#### The keyboard

First, we create space to build the keyboard into. We will be programmatically constructing the keyboard, because doing so gives us the flexibility to configure each key as we determine the appropriate data for the corresponding note. In our case, we get each key's frequency from a table, but it could be calculated algorithmically as well.

```html
<div class="container">
  <div class="keyboard"></div>
</div>
```

The {{HTMLElement("div")}} named `"container"` is the scrollable box that lets the keyboard be scrolled horizontally if it's too wide for the available space. The keys themselves will be inserted into the block of class `"keyboard"`.

#### The settings bar

Beneath the keyboard, we'll put some controls for configuring the layer. For now, we will have two controls: one to set the main volume and another to select what periodic waveform to use when generating notes.

##### The volume control

First we create the `<div>` to contain the settings bar, so it can be styled as needed. Then we establish a box that will be presented on the left side of the bar and place a label and an {{HTMLElement("input")}} element of type `"range"`. The range element will typically be presented as a slider control; we configure it to allow any value between 0.0 and 1.0, stepping by 0.01 each position.

```html-nolint
<div class="settingsBar">
  <div class="left">
    <span>Volume: </span>
    <input
      type="range"
      min="0.0"
      max="1.0"
      step="0.01"
      value="0.5"
      list="volumes"
      name="volume" />
    <datalist id="volumes">
      <option value="0.0" label="Mute"></option>
      <option value="1.0" label="100%"></option>
    </datalist>
  </div>
```

We specify a default value of 0.5, and we provide a {{HTMLElement("datalist")}} element which is connected to the range using the [`list`](/en-US/docs/Web/HTML/Reference/Elements/input#list) attribute to find an option list whose ID matches; in this case, the data set is named `"volumes"`. This lets us provide a set of common values and special strings which the browser may optionally choose to display in some fashion; we provide names for the values 0.0 ("Mute") and 1.0 ("100%").

##### The waveform picker

On the right side of the settings bar, we place a label and a {{HTMLElement("select")}} element named `"waveform"` whose options correspond to the available waveforms.

```html-nolint
  <div class="right">
    <span>Current waveform: </span>
    <select name="waveform">
      <option value="sine">Sine</option>
      <option value="square" selected>Square</option>
      <option value="sawtooth">Sawtooth</option>
      <option value="triangle">Triangle</option>
      <option value="custom">Custom</option>
    </select>
  </div>
</div>
```

### CSS

```css
.container {
  overflow-x: scroll;
  overflow-y: hidden;
  width: 660px;
  height: 110px;
  white-space: nowrap;
  margin: 10px;
}

.keyboard {
  width: auto;
  padding: 0;
  margin: 0;
}

.key {
  cursor: pointer;
  font:
    16px "Open Sans",
    "Lucida Grande",
    "Arial",
    sans-serif;
  border: 1px solid black;
  border-radius: 5px;
  width: 20px;
  height: 80px;
  text-align: center;
  box-shadow: 2px 2px darkgray;
  display: inline-block;
  position: relative;
  margin-right: 3px;
  user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
}

.key div {
  position: absolute;
  bottom: 0;
  text-align: center;
  width: 100%;
  pointer-events: none;
}

.key div sub {
  font-size: 10px;
  pointer-events: none;
}

.key:hover {
  background-color: #eef;
}

.key:active,
.active {
  background-color: #000;
  color: #fff;
}

.octave {
  display: inline-block;
  padding: 0 6px 0 0;
}

.settingsBar {
  padding-top: 8px;
  font:
    14px "Open Sans",
    "Lucida Grande",
    "Arial",
    sans-serif;
  position: relative;
  vertical-align: middle;
  width: 100%;
  height: 30px;
}

.left {
  width: 50%;
  position: absolute;
  left: 0;
  display: table-cell;
  vertical-align: middle;
}

.left span,
.left input {
  vertical-align: middle;
}

.right {
  width: 50%;
  position: absolute;
  right: 0;
  display: table-cell;
  vertical-align: middle;
}

.right span {
  vertical-align: middle;
}

.right input {
  vertical-align: baseline;
}
```

### JavaScript

The JavaScript code begins by initializing a number of variables.

```js
const audioContext = new AudioContext();
const oscList = [];
let mainGainNode = null;
```

1. `audioContext` is created as an instance of {{domxref("AudioContext")}}.
2. `oscList` is set up to be ready to contain a list of all currently-playing oscillators. It starts off empty, since there are none playing yet.
3. `mainGainNode` is set to null; during the setup process, it will be configured to contain a {{domxref("GainNode")}} which all playing oscillators will connect to and play through to allow the overall volume to be controlled using a single slider control.

```js
const keyboard = document.querySelector(".keyboard");
const wavePicker = document.querySelector("select[name='waveform']");
const volumeControl = document.querySelector("input[name='volume']");
```

References to elements we'll need access to are obtained:

- `keyboard` is the container element into which the keys will be placed.
- `wavePicker` is the {{HTMLElement("select")}} element used to choose the waveform to use for the notes.
- `volumeControl` is the {{HTMLElement("input")}} element (of type `"range"`) used to control the main audio volume.

```js
let customWaveform = null;
let sineTerms = null;
let cosineTerms = null;
```

Finally, global variables that will be used when constructing waveforms are created:

- `customWaveform` will be set up as a {{domxref("PeriodicWave")}} describing the waveform to use when the user selects "Custom" from the waveform picker.
- `sineTerms` and `cosineTerms` will be used to store the data for generating the waveform; each will contain an array that's generated when the user chooses "Custom".

### Creating the note table

The `createNoteTable()` function builds the array `noteFreq` to contain an array of objects representing each octave. Each octave, in turn, has one named property for each note in that octave; the property's name is the note's name (such as "C#" to represent C-sharp), and the value is the frequency, in Hertz, of that note. We only hardcode one octave; each subsequent octave can be derived from the previous octave by doubling each note.

```js
function createNoteTable() {
  const noteFreq = [
    { A: 27.5, "A#": 29.13523509488062, B: 30.867706328507754 },
    {
      C: 32.70319566257483,
      "C#": 34.64782887210901,
      D: 36.70809598967595,
      "D#": 38.89087296526011,
      E: 41.20344461410874,
      F: 43.65352892912549,
      "F#": 46.2493028389543,
      G: 48.99942949771866,
      "G#": 51.91308719749314,
      A: 55,
      "A#": 58.27047018976124,
      B: 61.73541265701551,
    },
  ];
  for (let octave = 2; octave <= 7; octave++) {
    noteFreq.push(
      Object.fromEntries(
        Object.entries(noteFreq[octave - 1]).map(([key, freq]) => [
          key,
          freq * 2,
        ]),
      ),
    );
  }
  noteFreq.push({ C: 4186.009044809578 });
  return noteFreq;
}
```

In part, the resulting object looks like this:

<table class="standard-table">
  <tbody>
    <tr>
      <th scope="row">Octave</th>
      <td colspan="8">Notes</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th scope="row">0</th>
      <td>"A" ⇒ 27.5</td>
      <td>"A#" ⇒ 29.14</td>
      <td>"B" ⇒ 30.87</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th scope="row">1</th>
      <td>"C" ⇒ 32.70</td>
      <td>"C#" ⇒ 34.65</td>
      <td>"D" ⇒ 36.71</td>
      <td>"D#" ⇒ 38.89</td>
      <td>"E" ⇒ 41.20</td>
      <td>"F" ⇒ 43.65</td>
      <td>"F#" ⇒ 46.25</td>
      <td>"G" ⇒ 49</td>
      <td>"G#" ⇒ 51.9</td>
      <td>"A" ⇒ 55</td>
      <td>"A#" ⇒ 58.27</td>
      <td>"B" ⇒ 61.74</td>
    </tr>
    <tr>
      <th scope="row">2</th>
      <td colspan="12">. . .</td>
    </tr>
  </tbody>
</table>

With this table in place, we can find out the frequency for a given note in a particular octave quite easily. If we want the frequency for the note G# in octave 1, we use `noteFreq[1]["G#"]` and get the value 51.9 as a result.

> [!NOTE]
> The values in the example table above have been rounded to two decimal places.

### Building the keyboard

The `setup()` function is responsible for building the keyboard and preparing the app to play music.

```js
function setup() {
  const noteFreq = createNoteTable();

  volumeControl.addEventListener("change", changeVolume, false);

  mainGainNode = audioContext.createGain();
  mainGainNode.connect(audioContext.destination);
  mainGainNode.gain.value = volumeControl.value;

  // Create the keys; skip any that are sharp or flat; for
  // our purposes we don't need them. Each octave is inserted
  // into a <div> of class "octave".

  noteFreq.forEach((keys, idx) => {
    const keyList = Object.entries(keys);
    const octaveElem = document.createElement("div");
    octaveElem.className = "octave";

    keyList.forEach((key) => {
      if (key[0].length === 1) {
        octaveElem.appendChild(createKey(key[0], idx, key[1]));
      }
    });

    keyboard.appendChild(octaveElem);
  });

  document
    .querySelector("div[data-note='B'][data-octave='5']")
    .scrollIntoView(false);

  sineTerms = new Float32Array([0, 0, 1, 0, 1]);
  cosineTerms = new Float32Array(sineTerms.length);
  customWaveform = audioContext.createPeriodicWave(cosineTerms, sineTerms);

  for (let i = 0; i < 9; i++) {
    oscList[i] = {};
  }
}

setup();
```

1. The table which maps note names and octaves to their frequencies is created by calling `createNoteTable()`.
2. An event handler is established (by calling our old friend {{domxref("EventTarget.addEventListener", "addEventListener()")}} to handle {{domxref("HTMLElement/change_event", "change")}} events on the main gain control. This will update the main gain node's volume to the new value of the control.
3. Next, we iterate over each octave in the note frequencies table. For each octave, we use {{jsxref("Object.entries()")}} to get a list of the notes in that octave.
4. Create a {{HTMLElement("div")}} to contain that octave's notes (so we can have a small bit of space drawn between octaves), and set its class name to "octave"
5. For each key in the octave, we check to see if the note's name has more than one character. We skip these, because we're leaving out the sharp notes in this example. If the note's name is only one character, then we call `createKey()`, specifying the note string, octave, and frequency. The returned element is appended to the octave element created in step 4.
6. When each octave element has been built, it's appended to the keyboard.
7. Once the keyboard has been constructed, we scroll the note "B" in octave 5 into view; this has the effect of ensuring that middle-C is visible along with its surrounding keys.
8. Then a new custom waveform is built using {{domxref("BaseAudioContext.createPeriodicWave()")}}. This waveform will be used any time the user selects "Custom" from the waveform picker control.
9. Finally, the oscillator list is initialized to ensure that it's ready to receive information identifying which oscillators are associated with which keys.

#### Creating a key

The `createKey()` function is called once for each key that we want to present in the virtual keyboard. It creates the elements that comprise the key and its label, adds some data attributes to the element for later use, and assigns event handlers for the events we care about.

```js
function createKey(note, octave, freq) {
  const keyElement = document.createElement("div");
  const labelElement = document.createElement("div");

  keyElement.className = "key";
  keyElement.dataset["octave"] = octave;
  keyElement.dataset["note"] = note;
  keyElement.dataset["frequency"] = freq;
  labelElement.appendChild(document.createTextNode(note));
  labelElement.appendChild(document.createElement("sub")).textContent = octave;
  keyElement.appendChild(labelElement);

  keyElement.addEventListener("mousedown", notePressed, false);
  keyElement.addEventListener("mouseup", noteReleased, false);
  keyElement.addEventListener("mouseover", notePressed, false);
  keyElement.addEventListener("mouseleave", noteReleased, false);

  return keyElement;
}
```

After creating the elements that will represent the key and its label, we configure the key's element by setting its class to "key" (which establishes its appearance). Then we add [`data-*`](/en-US/docs/Web/HTML/Reference/Global_attributes/data-*) attributes which contain the key's octave (attribute `data-octave`), string representing the note to play (attribute `data-note`), and frequency (attribute `data-frequency`) in Hertz. This will let us easily fetch that information as needed when handling events.

### Making music

#### Playing a tone

The `playTone()` function's job is to play a tone at the given frequency. This will be used by the handler for events triggering keys on the keyboard to start playing the appropriate notes.

```js
function playTone(freq) {
  const osc = audioContext.createOscillator();
  osc.connect(mainGainNode);

  const type = wavePicker.options[wavePicker.selectedIndex].value;

  if (type === "custom") {
    osc.setPeriodicWave(customWaveform);
  } else {
    osc.type = type;
  }

  osc.frequency.value = freq;
  osc.start();

  return osc;
}
```

`playTone()` begins by creating a new {{domxref("OscillatorNode")}} by calling the {{domxref("BaseAudioContext.createOscillator()")}} method. We then connect it to the main gain node by calling the new oscillator's {{domxref("AudioNode/connect", "connect()")}} method;, which tells the oscillator where to send its output to. By doing this, changing the gain of the main gain node will affect the volume of all tones being generated.

Then we get the type of waveform to use by checking the value of the waveform picker control in the settings bar. If the user has it set to `"custom"`, we call {{domxref("OscillatorNode.setPeriodicWave()")}} to configure the oscillator to use our custom waveform. Doing this automatically sets the oscillator's {{domxref("OscillatorNode.type", "type")}} to `custom`. If any other waveform type is selected in the wave picker, we set the oscillator's type to the value of the picker; that value will be one of `sine`, `square`, `triangle`, and `sawtooth`.

The oscillator's frequency is set to the value specified in the `freq` parameter by setting the value of the {{domxref("OscillatorNode.frequency")}} {{domxref("AudioParam")}} object. Then, at last, the oscillator is started up so that it begins to produce sound by calling the oscillator's inherited {{domxref("AudioScheduledSourceNode.start()")}} method.

#### Playing a note

When the {{domxref("Element/mousedown_event", "mousedown")}} or {{domxref("Element/mouseover_event", "mouseover")}} event occurs on a key, we want to start playing the corresponding note. The `notePressed()` function is used as the event handler for these events.

```js
function notePressed(event) {
  if (event.buttons & 1) {
    const dataset = event.target.dataset;

    if (!dataset["pressed"] && dataset["octave"]) {
      const octave = Number(dataset["octave"]);
      oscList[octave][dataset["note"]] = playTone(dataset["frequency"]);
      dataset["pressed"] = "yes";
    }
  }
}
```

We start by checking whether the primary mouse button is pressed, for two reasons. First, we want to only allow the primary mouse button to trigger notes playing. Second, and more importantly, we are using this to handle {{domxref("Element/mouseover_event", "mouseover")}} for cases where the user is dragging from note to note, and we only want to start playing the note if the mouse is pressed when it enters the element.

If the mouse button is in fact down, we get the pressed key's [`dataset`](/en-US/docs/Web/API/HTMLElement/dataset) attribute; this makes it easy to access the custom data attributes on the element. We look for a `data-pressed` attribute; if there isn't one (which indicates that the note isn't already playing), we call `playTone()` to start playing the note, passing in the value of the element's `data-frequency` attribute. The returned oscillator is stored into `oscList` for future reference, and `data-pressed` is set to `yes` to indicate that the note is playing so we don't start it again next time this is called.

#### Stopping a tone

The `noteReleased()` function is the event handler called when the user releases the mouse button or moves the mouse out of the key that's currently playing.

```js
function noteReleased(event) {
  const dataset = event.target.dataset;

  if (dataset && dataset["pressed"]) {
    const octave = Number(dataset["octave"]);

    if (oscList[octave] && oscList[octave][dataset["note"]]) {
      oscList[octave][dataset["note"]].stop();
      delete oscList[octave][dataset["note"]];
      delete dataset["pressed"];
    }
  }
}
```

`noteReleased()` uses the `data-octave` and `data-note` custom attributes to look up the key's oscillator, then calls the oscillator's inherited {{domxref("AudioScheduledSourceNode.stop", "stop()")}} method to stop playing the note. Finally, the `oscList` entry for the note is cleared and the `data-pressed` attribute is removed from the key element (as identified by {{domxref("event.target")}}), to indicate that the note is not currently playing.

#### Changing the main volume

The volume slider in the settings bar provides an interface to change the gain value on the main gain node, thereby changing the loudness of all playing notes. The `changeVolume()` method is the handler for the {{domxref("HTMLElement/change_event", "change")}} event on the slider.

```js
function changeVolume(event) {
  mainGainNode.gain.value = volumeControl.value;
}
```

This sets the value of the main gain node's `gain` {{domxref("AudioParam")}} to the slider's new value.

#### Keyboard support

The code below adds [`keydown`](/en-US/docs/Web/API/Element/keydown_event) and [`keyup`](/en-US/docs/Web/API/Element/keyup_event) event listeners to handle keyboard input. The `keydown` event handler calls `notePressed()` to start playing the note corresponding to the key that was pressed, and the `keyup` event handler calls `noteReleased()` to stop playing the note corresponding to the key that was released.

```js
const synthKeys = document.querySelectorAll(".key");
// prettier-ignore
const keyCodes = [
  "Space",
  "ShiftLeft", "KeyZ", "KeyX", "KeyC", "KeyV", "KeyB", "KeyN", "KeyM", "Comma", "Period", "Slash", "ShiftRight",
  "KeyA", "KeyS", "KeyD", "KeyF", "KeyG", "KeyH", "KeyJ", "KeyK", "KeyL", "Semicolon", "Quote", "Enter",
  "Tab", "KeyQ", "KeyW", "KeyE", "KeyR", "KeyT", "KeyY", "KeyU", "KeyI", "KeyO", "KeyP", "BracketLeft", "BracketRight",
  "Digit1", "Digit2", "Digit3", "Digit4", "Digit5", "Digit6", "Digit7", "Digit8", "Digit9", "Digit0", "Minus", "Equal", "Backspace",
  "Escape",
];
function keyNote(event) {
  const elKey = synthKeys[keyCodes.indexOf(event.code)];
  if (elKey) {
    if (event.type === "keydown") {
      elKey.tabIndex = -1;
      elKey.focus();
      elKey.classList.add("active");
      notePressed({ buttons: 1, target: elKey });
    } else {
      elKey.classList.remove("active");
      noteReleased({ buttons: 1, target: elKey });
    }
    event.preventDefault();
  }
}
addEventListener("keydown", keyNote);
addEventListener("keyup", keyNote);
```

### Result

Put all together, the result is a simple but working point-and-click musical keyboard:

{{ EmbedLiveSample('The_video_keyboard', 680, 200) }}

## See also

- [Web Audio API](/en-US/docs/Web/API/Web_Audio_API)
- {{domxref("OscillatorNode")}}
- {{domxref("GainNode")}}
- {{domxref("AudioContext")}}
