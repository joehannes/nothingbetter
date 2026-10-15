# Design/04 · Voice UX & Conversation Design

> Status: Draft v1 · Pairs with [AI/04 dials](../03-ai/04-conversation-engine-and-adaptive-dials.md) and [AI/08 voice](../03-ai/08-voice-and-speech.md).

## 1. Multimodal rules

1. **Speak the headline, show the details.** Voice says one sentence; the screen shows the cards.
2. **Every spoken question has chips on screen** so users can answer by voice or tap.
3. **Live captions** of the user's speech (partial transcript) and the assistant's speech (synced highlight).
4. **Barge-in is welcome.** The assistant stops mid-sentence when the user talks.
5. **Confirm sensitive actions on screen** (remember an allergy, pay).
6. **Graceful misunderstanding:** "Sorry, did you mean the Thai one or the tacos?" — offer the two most likely options as chips.

## 2. Writing for the Host

| Do | Don't |
|---|---|
| "Three picks, here by 12:35." | "Great question! I'd be absolutely delighted to help you find some amazing lunch options today!" |
| "Mia's pasta is sesame-safe — the kitchen confirmed." | "Don't worry, I've made sure nothing will hurt Mia!" (overpromising) |
| "This one's richer — about twice the sodium of the bowl." | "This is unhealthy." |
| "Traditionally, ginger is used to settle the stomach." | "Ginger will cure your nausea." |
| "Iftar is at 7:12 pm — I'll aim to arrive by 7:00." | "Since you're Muslim…" |
| "Two couriers, arriving within 8 minutes of each other. One courier would be $6 more but all at once." | Hiding courier count or fees |

Length limits per lane are enforced in prompts and checked in evals.

## 3. Conversation patterns

### Opening (by context)

- Workday lunch: "Lunch · 40 min until your 12:45. What sounds good?" + chips (*Usual*, *Light*, *Protein*, *Surprise me*).
- Weeknight household: "Dinner for the four of you?" + seats.
- Calendar occasion: "Anniversary tonight 🥂 — recreate something or try something new?"
- Late night: "Late-night bite. Something comforting or something light?"

### Clarifying (one at a time, highest information gain)

"Spicy or mild today?" · "Same kitchen for everyone, or mix and match?" · "Budget around $60 still right?"

### Proposing

"Here are three ways to do it: one kitchen, simplest; best for everyone; or the budget pick." (cards)

### Confirming

"So: ropa vieja, quinoa bowl, kids pasta (sesame-safe), margherita, and empanadas to share — $72.40, one courier, 7:20–7:30. Check with the kitchens?"

### Waiting (kitchen checks)

"Asking the kitchens… Luna said yes — ready by 7:15. Waiting on Bella Pizza." (spoken only if the user is in voice mode; otherwise chips update silently)

### Recovery

"Bella Pizza can't do the margherita tonight. Their pepperoni or the margherita from Forno 5 minutes away — same price?"

## 4. Voice-specific UI

- Hold-to-talk: orb grows, waveform ring, haptic tick at start/stop (mobile).
- Hands-free: persistent pill "Listening… tap to stop"; auto-stop after 8 s silence.
- Speaking: orb pulses; "Skip" button; captions highlight current sentence.
- Errors: "I lost you for a second — say that again?" with retry chip.

## 5. Accessibility of voice

- Screen readers: voice mode never conflicts with VoiceOver/TalkBack (announce state changes via ARIA live regions; pause TTS when a screen reader is speaking where detectable).
- Speech-impaired users: full text path; adjustable endpointing patience ("Give me more time to finish").

## 6. Localization of tone

- Spanish (US/LatAm): warm, *tú* by default in casual contexts; dish names in original language with short explanation.
- French (Canada): *vous* by default, switch to *tu* if the user does; Québécois dish names respected.
- Code-switching mirrors the user.
