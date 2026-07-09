# SmartVibe Platform Overview

*A client-friendly guide to how the platform works*

---

## What Is SmartVibe?

SmartVibe is an **AI-powered music creation platform** that turns a simple idea into a finished song in about **4 minutes**.

Users describe what they want — a mood, genre, occasion, or message — and the platform creates a complete track with audio (and cover artwork when available). No musical skills are required.

The platform works on **two channels**:

| Channel | Best for |
|---------|----------|
| **Website** | Users who want a full studio-style experience with a personal song library |
| **WhatsApp** | Mobile-first users who prefer a conversational, guided experience (branded as **MTN MusicStudio**) |

Both channels share the same account, credits, and song history.

---

## How Users Get Started

### On the website

1. User visits the SmartVibe homepage.
2. They **sign up** with email and password.
3. New accounts receive **5 free credits** (1 credit = 1 song).
4. They describe their song and tap **Generate**.
5. The finished song appears in **Your generations**, where they can play and download it.

### On WhatsApp

1. User taps **Chat on WhatsApp** on the website.
2. They **verify their phone number** with a one-time code sent via WhatsApp.
3. Once verified, they open a chat with the SmartVibe business number.
4. A guided conversation walks them through choices (topic, style, recipient, occasion, language).
5. They confirm, and the song is created and **delivered directly in the chat** as audio.

> **Important:** WhatsApp creation requires a verified account linked to the same phone number. This keeps usage fair and tied to real users.

---

## Part 1: AI Music Generation — The User Experience

### What the user does

The user provides a **creative brief in plain language**. They never need to know chords, BPM, or production terms.

**Website example:**  
*"A dreamy synthwave track with retro drums and a nostalgic feel"*

**WhatsApp example (guided):**  
Birthday → Afrobeats → For my wife → Love → English → *"Afrobeats love song for my wife on her birthday"*

### What the user receives

- A **complete audio track** (MP3)
- **Cover artwork** (when available)
- A **title** and saved description in their personal library
- On WhatsApp: the cover image and audio file sent in the chat

### Step-by-step journey

```
Describe the song  →  Request accepted  →  Song is created  →  Listen & enjoy
     (30 seconds)        (immediate)         (~4 minutes)         (anytime)
```

| Step | What happens | What the user sees |
|------|--------------|-------------------|
| 1. Describe | User enters or selects their song idea | Prompt box (web) or guided menus (WhatsApp) |
| 2. Submit | Platform accepts the request | "Generating…" or "Create song" confirmation |
| 3. Create | AI builds the track | Countdown timer (web) or progress messages (WhatsApp) |
| 4. Deliver | Song is saved and shared | Player + download (web) or audio in chat (WhatsApp) |

### Progress updates on WhatsApp

During creation, users receive friendly status messages at key moments — for example around 1, 2, and 3 minutes — so they always know their song is on the way.

### Credits — simple and fair

| Rule | Detail |
|------|--------|
| Free start | **5 credits** on signup |
| Cost per song | **1 credit** |
| When charged | Only when the song **succeeds** |
| If it fails | Credit is **automatically refunded** |
| Remix (WhatsApp) | Returning users can tweak a previous song (funnier, different genre, shorter) — same 1-credit cost |

### Returning WhatsApp users

When a user comes back, they see a preview of their last song and can choose:

- **Remix it** — adjust the previous creation
- **Create new** — start a fresh song

---

## Part 2: The Smart Processing Layer — Built for High Traffic

This is the **core engine** that keeps SmartVibe stable even when many users request songs at the same time. Think of it as an intelligent **middle layer** between your users and the AI music provider — it manages demand, protects credits, and ensures no request is lost.

### The challenge

AI music creation depends on a **limited pool of generation capacity** (connections to the music provider). During peak times — promotions, viral moments, or busy evenings — not every song can start immediately.

Without a proper system, platforms either:

- **Crash or reject users** during busy periods, or
- **Lose requests** and charge users for songs that never arrive.

SmartVibe solves this with a **database-backed queue and processing system** designed for reliability at scale.

### How it works (in plain terms)

```
                    ┌─────────────────────────┐
   Users              │                         │
   (Web + WhatsApp) ─►│   Smart Request Queue   │  ← Every request is saved
                      │   (first in, first out) │
                      └───────────┬─────────────┘
                                  │
                      ┌───────────▼─────────────┐
                      │   Generation Slots      │  ← Limited pool of
                      │   (managed automatically)│    active connections
                      └───────────┬─────────────┘
                                  │
                      ┌───────────▼─────────────┐
                      │   AI Music Provider     │  ← Creates the actual song
                      └─────────────────────────┘
```

**1. Every request enters one shared queue**  
Whether from the website or WhatsApp, all song requests go into the same orderly queue. Nothing is dropped.

**2. Fair, automatic slot assignment**  
When a generation slot becomes free, the next waiting request is picked up automatically. Users are served in order.

**3. Transparent waiting during busy times**  
If all slots are busy, the user's request stays in line. They are told clearly:

- **WhatsApp:** *"Your song is in the queue. All generation slots are busy right now, but yours is lined up. We'll message you when creation begins."*
- **Website:** *"Waiting for an available account…"*

**4. Credits are protected**  
When a user submits a request, 1 credit is **held** (not spent yet). It is only charged when the song succeeds. If anything fails, the credit is **returned automatically**.

**5. Automatic monitoring**  
While a song is being created, the system checks progress every few seconds until it is done, fails, or times out.

**6. Safety net for stuck jobs**  
A background process runs on a schedule to:

- Detect jobs that have been stuck too long (e.g. over 10 minutes)
- Mark them as failed and **refund credits**
- Free up slots so the queue keeps moving

**7. Duplicate protection**  
WhatsApp messages are processed only once, so users are never charged twice for the same request — even if the network sends the same message multiple times.

### Why this matters for your business

| Benefit | What it means |
|---------|---------------|
| **Handles traffic spikes** | Launch campaigns or go viral without the platform breaking |
| **No lost requests** | Every valid request is queued and processed |
| **Fair user experience** | First come, first served; clear wait messages |
| **Credit integrity** | Users only pay for successful songs |
| **Self-healing** | Stuck jobs are recovered automatically; slots are freed for the next user |
| **Single pipeline** | Web and WhatsApp share one queue — consistent behavior everywhere |

### Scaling capacity

Peak throughput depends on how many **generation slots** (provider connections) are provisioned. The queue absorbs demand above that capacity — users wait with clear communication instead of being turned away.

---

## End-to-End User Flows (Summary)

### Website flow

```
Homepage → Sign in → Describe song → Generate → Wait (if busy) →
Countdown (~4 min) → Play & download from library
```

### WhatsApp flow

```
Verify phone on website → Open WhatsApp chat → Guided choices →
Confirm → Queue message (if busy) → Progress updates →
Audio (+ cover) delivered in chat
```

### What happens behind the scenes (both channels)

```
User submits request
       ↓
Enough credits?  →  No  →  Friendly "insufficient credits" message
       ↓ Yes
Request enters queue (1 credit held)
       ↓
Slot available?  →  No  →  User waits with clear notification
       ↓ Yes
AI creates song (~4 minutes)
       ↓
Success  →  Credit charged, song delivered
Failure  →  Credit refunded, user can retry
```

---

## Key Selling Points

1. **Dual-channel reach** — Web for power users; WhatsApp for mobile-first, conversational audiences (ideal for carrier partnerships like MTN).

2. **Guided WhatsApp journey** — Low friction for non-technical users; no typing long prompts required.

3. **Production-ready under load** — Queue-based processing layer built for real traffic, not just demos.

4. **Fair credit model** — Reserve on submit, charge on success, refund on failure.

5. **Engagement loops** — Remix and "create new" flows bring users back on WhatsApp.

6. **Unified platform** — One account, one credit balance, one song history across web and WhatsApp.

---

## Current Scope & Future Roadmap

**Available today**

- AI text-to-song (audio)
- Web app + WhatsApp integration
- Credit-based usage (5 free credits on signup)
- Queue system for high-traffic handling
- Phone verification for WhatsApp
- Remix for returning WhatsApp users

**Planned / not yet live**

- Self-serve payment and credit top-ups (credits are added manually by admins today)
- Video generation tiers
- Premium audio tiers in the UI
- Full credit purchase history in the web interface

---

## One-Line Summary

**SmartVibe turns anyone's idea into a personalized AI song in minutes — on the web or WhatsApp — with a queue-based processing engine that keeps the platform reliable even when demand is high.**
