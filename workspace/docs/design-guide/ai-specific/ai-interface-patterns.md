# AI Interface Design Patterns

Patterns for interfaces powered by LLMs, AI agents, and generative systems. This is a rapidly evolving area — these patterns represent established practices as of mid-2026.

## Core Principles for AI Interfaces

### 1. The AI Should Not Hide the Human
AI outputs should be clearly attributed. Users need to know when they're reading AI-generated content vs. human-written content vs. retrieved data.

### 2. The User Should Always Be in Control
Users should be able to:
- Modify AI outputs (edit, regenerate, provide feedback)
- Undo AI actions
- Switch to manual mode
- Understand why the AI produced a specific output

### 3. Set Expectations Honestly
- Don't claim AI is "intelligent" or "understands" — it processes and generates
- Show confidence levels when possible
- Be transparent about limitations

## Chat-Based AI Patterns

### Message Types
| Type | Visual Treatment | Example |
|------|-----------------|----------|
| User message | Aligned to the **end** of the reading direction, primary color bubble | User's question or prompt |
| AI response | Aligned to the **start**, neutral/surface bubble | AI's answer |
| System message | Centered, subtle, italic | "Conversation started", "Model changed" |
| Error | Left-aligned, danger color, icon | "Couldn't generate a response. [Retry]" |
| Source/reference | Inline cards or chips within AI response | Citations, data sources |

### Input Patterns
- **Text input with send button:** Standard chat input. Send button enables only when there's content.
- **Suggested prompts:** Show 3–5 suggested questions below the input when conversation is empty. Update contextually as the conversation progresses.
- **Attached context:** Allow file uploads, image uploads, or pasted content alongside the prompt.
- **Model/config controls:** Exposed as a settings panel or dropdown, not as the primary interface.

### Response Patterns
- **Streaming:** Show text appearing word by word (or token by token). This reduces perceived
  latency.

  > ⚠️ **Streaming is hostile to screen readers if you stream into a live region.** An
  > `aria-live` region re-announces on every mutation, so token-by-token output produces a
  > continuous, unusable stream of speech that the user cannot interrupt or navigate. Do **not**
  > put `aria-live="polite"` on the streaming message body.
  >
  > **Instead:** stream into a region that is *not* live, announce state transitions only
  > ("Generating response…", then "Response complete") via a separate small status region, and
  > let the user read the finished message by normal navigation. Offer a way to jump focus to
  > the completed response. Users who want live output can opt in — but it must be a choice,
  > not the default.

- **Stop generation button:** Allow users to stop an in-progress response. It must be keyboard
  reachable **while streaming** — if focus is trapped or the button only appears on hover, the
  user cannot stop anything.
- **Copy button:** On every AI response block.
- **Regenerate:** Allow re-generating the last response.
- **Thumbs up/down:** Simple feedback mechanism for response quality.

### Mixed-Initiative Interfaces
Beyond pure chat, AI interfaces increasingly mix AI and traditional UI:
- **AI fills a form, user reviews and edits** (not the user fills the form)
- **AI suggests actions, user approves/rejects** (action cards with approve/reject buttons)
- **AI highlights insights in existing data** (annotations on dashboards, documents)

### Chat layout is direction-dependent

The user-right / AI-left convention is an **LTR** convention. In Arabic and Hebrew it mirrors:
the user's message sits on the **left**, the AI's on the **right**. Build it with logical
properties (`margin-inline-start`, `align-items: flex-end`, `text-align: start`) so it
flips with `dir` instead of needing a second stylesheet.

Two further points for mixed-language chat, which is the normal case in Arabic-speaking
products:

- **Set `dir="auto"` on each message bubble individually**, not once on the container. A
  single conversation routinely contains Arabic and English messages, and the container's
  direction will be wrong for half of them.
- **Code blocks and URLs stay LTR** even inside an Arabic message — wrap them with
  `dir="ltr"` or they render scrambled.

→ [../accessibility/rtl-i18n.md](../accessibility/rtl-i18n.md)

→ See also: `trust-transparency.md`

## References
- Anthropic: Claude artifact and workbench design patterns (docs.anthropic.com)
- Vitaly Friedman: "Design Patterns For AI Chat Interfaces" (Jul 2025)
- Microsoft HAX Toolkit: human-AI interaction guidelines (microsoft.com/haxtoolkit)
- NN/g: AI and UX research (nngroup.com)
