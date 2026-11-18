# Cagair

> *Cagair* (Scottish Gaelic): **whisper**.

**Cagair is an open-source ESP32-S3 wearable that listens to your voice and streams it to your own self-hosted AI assistant, a private counselor whispering at your side.**

A personal assistant built on **trust, inclusivity and openness**.

> **Status: early prototype.** There is no working firmware yet. This repository starts with the vision; the hardware is on a breadboard and the project is being built in public, one small step at a time. Expect things to change.

---

## Why

Voice assistants and AI wearables are useful, but most of them send what you say to someone else's cloud, on someone else's terms. Cagair is the opposite bet: a small device you can build, read and repair yourself, talking only to an assistant *you* run.

It is also a learning project. It is built openly, from the signal level up, so anyone can follow along, understand the trade-offs of embedded audio, and reuse what works.

## Principles

1. **Yours, end to end.** Audio goes from the device to a server you host. No vendor cloud is required, and none is built in.
2. **Consent is visible.** It is always obvious when Cagair is listening, recording or off. Off means off.
3. **Open by default.** Firmware, hardware choices, wiring and decisions are published, along with the mistakes.
4. **Assistant-agnostic.** Cagair is the ears and the voice, not the brain. It should work with whichever self-hosted assistant you choose.
5. **Optional harness.** Cagair will provide its own harness (the assistant) to get you started, but it will never be required: you can plug in another one, such as Hermes Agent. The device will also speak an open protocol, so you can replace the Cagair server with your own stack.
6. **Small, honest steps.** Each milestone is something that actually works on real hardware before the next one starts. The README only claims what exists.
7. **Inclusive.** Affordable, off-the-shelf parts, clear documentation, and a design that should work for many voices, accents and needs.

## Non-goals

To keep the vision sharp, Cagair is explicitly **not**:

- **A cloud service or app ecosystem.** There is no Cagair account, no hosted backend, no subscription.
- **An AI model.** Cagair does not train or ship a language model. Its harness connects to one you run.
- **A covert recorder.** It is not designed to record people without their knowledge. Recording states are visible by design.
- **A smartphone replacement.** No screen, no apps, no notifications feed.
- **A finished consumer product (yet).** A small crowdfunded run may come later, but only once the open design is proven.

## Modes

Four modes are planned for exploration. None of them is implemented yet.

| Mode | What it does |
| --- | --- |
| **Meeting / interview** | Records a conversation, then produces a transcript, a summary and a list of to-dos. |
| **Active assistant** | Always listening; a lightweight model decides whether to act, record or ignore, in place of a wake word. |
| **Wake word** | Listens only for a wake word, then takes a command. |
| **Off** | Microphones off. Nothing is captured or sent. |

The active assistant mode is the most ambitious and the most sensitive. It will only ship with the consent and visibility principles above fully in place.

## How it works (planned)

Cagair has three parts: the **wearable**, the **Cagair server** that runs on a machine you own, and an **optional harness**, the assistant that does the thinking.

```
  [ Cagair wearable ] --- Wi-Fi ---> [ Cagair server ] ---> [ Harness ] ---> [ Model ]
   mics, speaker,      (local        receives audio,       Cagair's own,    runs where
   button, LED          network)     runs the modes,       or Hermes        you choose
         ^                           speech-to-text,       and co
         |                           text-to-speech
         +------------ spoken reply ------+
```

- **The wearable** captures audio, streams it over your local Wi-Fi, and plays back the reply. Working away from home comes later, once it can be done securely.
- **The Cagair server** receives the audio stream and runs the modes. It turns speech into text, passes it to your harness, and turns the answer back into speech. It is where mode logic that is too heavy for the device lives, such as recording a meeting before it is summarized.
- **The harness** is the assistant: it calls the model, keeps memory and produces things like the summary and to-dos of the meeting mode. Cagair will offer its own, but you can use another one, such as Hermes Agent, instead.
- **Bring your own stack.** The protocol between the wearable and the server will be documented, so you can point the device at your own server or an existing voice pipeline too.

The protocol, the server and the harness are still being designed. The draft system architecture is in [docs/architecture.md](docs/architecture.md).

## Hardware (current prototype)

The first prototype is a breadboard build from off-the-shelf parts:

| Part | Role |
| --- | --- |
| ESP32-S3 dev board | Main microcontroller, Wi-Fi |
| 2x SPH0645LM4H | I2S MEMS microphones |
| MAX98357A | I2S class-D amplifier |
| Adafruit 1314 speaker (3", 4 Ω, 3 W) | Audio out. Too big for a wearable; to be replaced with a smaller one. |
| Breadboard and jumper wires | Prototyping |

A wearable form factor (enclosure, battery, smaller speaker) comes later.

## Roadmap

Small steps, each one working on real hardware before the next:

- [x] **0. Vision.** This README.
- [ ] **1. Hello, board.** Flash the ESP32-S3, blink an LED, read a button.
- [ ] **2. Hear.** Capture audio from the I2S microphones and inspect the signal.
- [ ] **3. Speak.** Play a tone, then a sound clip, through the amplifier and speaker.
- [ ] **4. Loopback.** Microphone to speaker on the device.
- [ ] **5. Stream.** Send audio over Wi-Fi to a first, minimal version of the Cagair server on your own machine.
- [ ] **6. Converse.** The server runs speech-to-text, your harness replies, text-to-speech plays it back.
- [ ] **7. Open protocol.** Document the device-to-server protocol so the server is truly optional and anyone can bring their own stack.
- [ ] **8. Modes.** Off, wake word, meeting / interview, then active assistant.
- [ ] **9. Wear it.** Smaller speaker, battery, enclosure.

## Inspiration

Cagair stands on the shoulders of other open projects. The closest reference is [xiaozhi-esp32](https://github.com/78/xiaozhi-esp32), an open-source ESP32 voice assistant, whose approach is a big help for learning the hardware and audio pipeline.

## Get involved

Cagair is built in public, and early input shapes it the most.

- **Wishlist.** What would you want a private, self-hosted voice wearable to do? Open an [issue](https://github.com/alphanovia/cagair/issues) and tell us. The wishlist will be gathered from what people ask for.
- **Follow along.** A build journal is planned to document each step, including what went wrong.
- **Contribute.** Ideas, questions, hardware tips and corrections are all welcome as issues or pull requests.

## License

[MIT](LICENSE)
