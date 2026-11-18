# Cagair architecture (C4 model)

> **Status: vision draft.** Nothing here is built yet. This describes where Cagair is heading, at the two highest levels of the [C4 model](https://c4model.com): **system context (C1)** and **containers (C2)**. Component and code levels are left out on purpose until there is code to describe.

## C1: System context

Who uses Cagair, and what it talks to.

```mermaid
flowchart TB
    wearer["👤 <b>Wearer</b><br/>Speaks to Cagair, hears its replies,<br/>controls the mode with a button"]
    bystander["👥 <b>People nearby</b><br/>Meeting or interview participants.<br/>Can see when Cagair is listening"]

    subgraph cagair_boundary ["Cagair"]
        cagair["🎙️ <b>Cagair</b><br/>Wearable plus self-hosted server.<br/>The ears and the voice of your assistant"]
    end

    assistant["🧠 <b>AI assistant</b><br/>Self-hosted. Hermes Agent and co,<br/>or the optional Cagair assistant"]
    models["⚙️ <b>Model runtime</b><br/>Where the models run:<br/>language, speech-to-text, text-to-speech"]
    tools["🗂️ <b>Your tools and data</b><br/>Notes, calendar, to-do list,<br/>whatever the assistant is allowed to use"]

    wearer -- "talks to, listens to" --> cagair
    bystander -. "is heard by, only when visible" .-> cagair
    cagair -- "sends what was said,<br/>gets a reply" --> assistant
    cagair -- "speech-to-text,<br/>text-to-speech" --> models
    assistant -- "thinks with" --> models
    assistant -- "acts on" --> tools

    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef system fill:#1168bd,stroke:#0b4884,color:#fff
    classDef external fill:#999,stroke:#6b6b6b,color:#fff
    class wearer,bystander person
    class cagair system
    class assistant,models,tools external
```

**Reading it:** Cagair is deliberately small. It owns the device and the voice pipeline, and treats the assistant and the models as things *you* already run, or choose to run. Nothing in this picture is a vendor cloud.

## C2: Containers

What Cagair is made of, and where each part runs.

```mermaid
flowchart TB
    wearer["👤 <b>Wearer</b>"]
    bystander["👥 <b>People nearby</b>"]

    subgraph cagair_boundary ["Cagair"]
        wearable["📟 <b>Wearable</b><br/><i>ESP32-S3 firmware</i><br/>Mics, speaker, button, LED.<br/>Captures and plays audio, shows the mode"]
        server["🔀 <b>Cagair server</b><br/><i>Self-hosted service</i><br/>Speaks the open device protocol.<br/>Turns speech into text and back,<br/>runs the modes"]
        own_assistant["🧠 <b>Cagair assistant</b><br/><i>Optional, self-hosted</i><br/>Cagair's own harness:<br/>counselor persona, memory, to-dos"]
    end

    third_party["🧠 <b>Third-party assistant</b><br/><i>Hermes Agent, OpenClaw, ...</i>"]
    models["⚙️ <b>Model runtime</b><br/><i>Often another machine, with a GPU</i><br/>LLM, speech-to-text, text-to-speech"]

    wearer -- "voice, button" --> wearable
    wearable -- "spoken reply, LED" --> wearer
    bystander -. "voice" .-> wearable

    wearable <-- "audio both ways, mode, events<br/><i>open protocol over Wi-Fi</i>" --> server
    server -- "text in, text out<br/><i>assistant API</i>" --> own_assistant
    server -. "or" .-> third_party
    server -- "speech-to-text, text-to-speech" --> models
    own_assistant -- "prompts" --> models
    third_party -- "prompts" --> models

    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef container fill:#438dd5,stroke:#2e6295,color:#fff
    classDef optional fill:#438dd5,stroke:#2e6295,color:#fff,stroke-dasharray: 5 5
    classDef external fill:#999,stroke:#6b6b6b,color:#fff
    class wearer,bystander person
    class wearable,server container
    class own_assistant optional
    class third_party,models external
```

| Container | Runs on | Responsibility |
| --- | --- | --- |
| **Wearable** | ESP32-S3, on you | Capture and play audio, show clearly when it is listening, recording or off. Stays dumb on purpose: no assistant logic lives here. |
| **Cagair server** | A machine you own (home server, NAS, laptop) | The only thing the wearable talks to. Speaks the open protocol, turns speech into text and the reply back into speech, and runs the modes (wake word handling, meeting recording, the "act, record or ignore" filter). |
| **Cagair assistant** *(optional)* | A machine you own, often the same as the server | Cagair's own harness: the counselor persona, memory, summaries and to-dos. Skip it and point the server at another assistant instead. |
| **Third-party assistant** *(external)* | Wherever you run it | Hermes Agent and similar. Swappable with the Cagair assistant because both sit behind the same assistant API. |
| **Model runtime** *(external)* | Often a separate machine with a GPU | Serves the models. Cagair does not ship models, it connects to ones you run. |

**Two seams make Cagair open:**

1. **Device protocol** (wearable ↔ server). Documented, so anyone can replace the Cagair server with their own stack.
2. **Assistant API** (server ↔ assistant). Text in, text out, so the brain can be swapped without touching the device.

## Open questions

These are the decisions this draft does not settle yet.

1. **Is the server separate from the assistant?** This draft splits them: the server owns voice and modes, the assistant owns thinking. The README currently calls the voice part "the harness"; the naming should be settled and the README aligned.
2. **Where do speech-to-text and text-to-speech run?** Drawn here as models in the model runtime, called by the server. They could instead run inside the server itself for a single-box setup.
3. **Which assistant API?** OpenAI-compatible chat completions is the common denominator today (Hermes Agent exposes it). Meeting mode may need more than chat, such as sending a long transcript and getting a summary and to-dos back.
4. **Where does the "act, record or ignore" filter live?** On the wearable (more private, less capable), in the server (drawn here), or in the assistant.
5. **Does the server keep data?** Recordings, transcripts and to-dos have to live somewhere. That may be a sixth container (a data store) owned by the server, or it may belong to the assistant.
6. **What happens away from home Wi-Fi?** A wearable leaves the house. Options include a phone hotspot, a phone app as relay, or a VPN back to your server. This could add a container.
7. **People nearby.** They are part of the system context. How the design respects them (visible LED, an audible cue, a meeting-mode consent step) is a principle, but not yet a design.
