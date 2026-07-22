[🇰🇷 한국어](README.ko.md) | [🇯🇵 日本語](README.ja.md)

# Hybrid LLM Architecture

**Local model for personality, cloud model for facts — a privacy-first pattern for AI character systems.**

> From the [NVatar](https://github.com/nskit-io/nvatar) project — an AI avatar chat system running entirely on local hardware.

---

## Why This Exists

LLM routing exists (RouteLLM, FrugalGPT, Semantic Router). But existing work optimizes for **cost** or **difficulty** — this pattern instead splits by **function**:

| Layer | Where | Why |
|-------|-------|-----|
| Personality & Conversation | **Local** (Gemma 26B) | Privacy, latency, character consistency |
| Facts & Search | **Cloud** (Claude via CSW) | Accuracy, tool access, web search |

This is that architecture.

## The Problem

Local LLMs are great at personality but **hallucinate facts**. Cloud LLMs are accurate but **expensive for casual chat** and expose private conversations to third parties.

Running everything locally: fast + private, but wrong about facts.
Running everything in the cloud: accurate, but slow + expensive + privacy risk.

**Solution: Split by function, not by difficulty.**

## Architecture

```
User Message
    │
    ▼
Context Router (Gemma, local)
    │
    ├─ T1-T4, T8-T10 ──→ Gemma (local)
    │   Casual chat, emotions,     Personality layer
    │   advice, opinions,          Low latency
    │   memories, greetings        Private
    │
    └─ T5-T7 ──────────→ CSW → Claude (cloud)
        Facts, news,               Knowledge layer
        recommendations            Accurate + web search
```

## 10 Conversation Types

| Code | Type | Route | Thinking |
|------|------|-------|----------|
| T1 | Casual chat | Gemma | OFF |
| T2 | Emotion sharing | Gemma | OFF |
| T3 | Advice seeking | Gemma | OFF |
| T4 | Deep discussion | Gemma | **ON** |
| T5 | Fact question | **Cloud** | - |
| T6 | News/current events | **Cloud** | - |
| T7 | Recommendations | **Cloud** | - |
| T8 | Memory reference | Gemma | OFF |
| T9 | Avatar questions | Gemma | OFF |
| T10 | Greetings/farewells | Gemma | OFF |

**Key insight:** 70-80% of character conversations are T1-T4 (personality layer). Cloud is only needed for the 20-30% that require factual accuracy.

## Speed Comparison

Local classification avoids a network round-trip to the cloud. Most conversations complete in under 2 seconds locally.

## The "Shall I Search?" Pattern

Instead of silently routing to cloud, the character **asks permission**:

```
User: "서울 날씨 어때?"  (How's Seoul weather?)
Avatar: "찾아볼까?"      (Want me to look it up?)
→ User confirms with natural affirmative responses
→ CSW WebSearch triggers
Avatar: "서울 지금 18도래! 오후에 비 온다는데 우산 챙겨~"
```

This maintains the **friend persona** even during factual lookups. The character doesn't suddenly become a search engine.

## Placeholder Auto-Resolution

Sometimes Gemma writes placeholders in its response:

```
"GDP는 [최신 GDP 수치]인데..."
```

Pattern-based detection of placeholder text → triggers CSW search → replaces inline. The user never sees the placeholder.

## Failure Handling

Known failure patterns are detected and gracefully handled. The conversation never breaks. The character stays in persona.

## Why Not Just Use a Bigger Cloud Model?

| Concern | Local-first Hybrid | Cloud-only |
|---------|-------------------|------------|
| Privacy | Conversations stay local | All data sent to cloud |
| Latency | Sub-second for most messages | 3-5 sec for everything |
| Cost | Cloud only for 20% of messages | Cloud for 100% |
| Character consistency | Local model = one personality | API changes can shift behavior |
| Offline capability | Chat works without internet | Nothing works offline |

## Practical Application

This pattern applies beyond NVatar:

- **Customer service bots**: Personality local, knowledge base in cloud
- **Game NPCs**: Character AI local, world facts in cloud
- **Companion apps**: Emotional support local, medical/legal info in cloud
- **Education**: Tutor personality local, curriculum facts in cloud

## Related Projects

Other open-source components from the NVatar AI avatar system:

- [**Try Live Demo**](https://nskit-io.github.io/nvatar-demo/) — Experience NVatar in your browser

- [**avatar-chat**](https://github.com/nskit-io/avatar-chat) — Prompt engineering patterns for Gemma character AI
- [**chat-like-human-memory**](https://github.com/nskit-io/chat-like-human-memory) — 9D emotion tracking + personality evolution + 3-tier memory
- [**vrm-studio**](https://github.com/nskit-io/vrm-studio) — 3D VRM avatar chat room with Three.js + WebSocket
- [**CSW**](https://github.com/nskit-io/csw) — Claude Subscription Worker powering the cloud layer
- [**portable-ai-companion**](https://github.com/nskit-io/portable-ai-companion) — Cross-app franchise architecture for avatar portability

## License

CC BY-NC-SA 4.0 — see [LICENSE](LICENSE)

---

## Support This Research

NVatar is an independent R&D project exploring the frontier of hybrid AI architectures. Built by a solo founder at Neoulsoft Inc. — independent R&D, no external funding yet.

If you find this work valuable:

**Donation & Investment Inquiry**
- Email: [nskit@nskit.io](mailto:nskit@nskit.io)
- Organization: [NSKit](https://nskit.io) by Neoulsoft Inc.

<p align="center">
  <a href="https://github.com/nskit-io">github.com/nskit-io</a>
</p>
