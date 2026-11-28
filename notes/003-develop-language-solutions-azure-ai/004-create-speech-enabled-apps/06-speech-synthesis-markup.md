# 06 Speech Synthesis Markup Language

[Use SSML](https://learn.microsoft.com/en-us/training/modules/create-speech-enabled-apps/6-speech-synthesis-markup)

SSML is XML that controls how the text is spoken. Plain `speak_text_async` cannot do these:

- `mstts:express-as` sets a style such as cheerful or excited
- `break` inserts a pause
- `phoneme` forces a pronunciation, such as SQL as "sequel"
- prosody changes pitch and rate
- `say-as` reads a string as a date, time, or phone number
- audio inserts a recording

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
  <voice name="en-US-AriaNeural">
    <mstts:express-as style="cheerful">I say tomato</mstts:express-as>
  </voice>
  <voice name="en-US-GuyNeural">
    I say <phoneme alphabet="sapi" ph="t ao m ae t ow">tomato</phoneme>.
    <break strength="weak"/>Lets call the whole thing off!
  </voice>
</speak>
```

Send it with `speak_ssml_async`, not `speak_text_async`:

```python
result = synthesizer.speak_ssml_async(ssml).get()
```
