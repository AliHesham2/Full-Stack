# Trust and Transparency in AI Interfaces

Users don't trust AI outputs by default. Trust must be designed in.

## Trust as a Design Problem

Trust in AI is not binary — it's earned incrementally and lost instantly. A useful mental model (not empirically validated, but useful for structuring decisions):

Trust ≈ (Competence + Transparency + Control) ÷ (Uncertainty + Past Failures)

You can't eliminate uncertainty or prevent all failures, but you can maximize the numerator.

## Transparency Mechanisms

### 1. Confidence Signaling
When the AI is uncertain, communicate it.
- **High confidence:** Present the answer directly
- **Medium confidence:** Present the answer with a note ("I'm not entirely sure about this")
- **Low confidence:** Present the answer with a strong caveat or suggest the user verify

> ⚠️ **Do not display a model's self-reported confidence as a number.** A percentage or a
> progress bar reads as a measurement, and LLM self-assessed confidence is not well calibrated
> — models are routinely confident and wrong, which is precisely the failure the user needs
> protection from. A confidence display that is itself unreliable *manufactures* trust rather
> than earning it, which inverts the purpose of this file.
>
> **Prefer signals grounded in something checkable:** whether a source was actually retrieved,
> whether sources agree, whether the answer falls inside the system's known scope, when the
> data was last updated. Hedging language is acceptable; a fabricated "87% confident" is not.
> When confidence is genuinely low, the strongest move is to say what would resolve it
> ("I couldn't find a source for this — check X").

### 2. Source Attribution
- Show where the AI got its information (citations, URLs, data sources)
- Distinguish between AI-generated text and retrieved factual content
- Allow users to click through to sources

### 3. Process Visibility
- Show what the AI is doing ("Searching...", "Analyzing document...", "Comparing options...")
- Explain why it gave a specific answer when asked
- Show the reasoning chain (especially for decision-support AI)

### 4. Limitation Disclosure
- State what the AI cannot do
- State known failure modes ("May produce inaccurate information about recent events")
- Don't overclaim ("AI-powered" ≠ "intelligent")

## Control Mechanisms

### User Control Checklist
- [ ] Can edit or modify AI output
- [ ] Can regenerate the response
- [ ] Can undo an AI action
- [ ] Can provide feedback (thumbs up/down, report)
- [ ] Can switch to manual mode
- [ ] Can adjust AI behavior (temperature, detail level, style)

### Error Handling in AI Interfaces

| Error Type | User-Facing Message | Recovery Action |
|---|---|---|
| Network/timeout | "Couldn't reach the server. Your message is saved. [Retry]" | Retry, input preserved |
| Content policy block | "I can't help with this request. Rephrasing it may help." | Allow rephrase, keep the text |
| Context too long | "This conversation is too long to continue. [Summarise it] or [Start a new one]" | Both options offered |
| Model error | "The model didn't return a response. [Try again]" | Retry button |

> **"Something went wrong" was in this table.** It is the exact string that
> [../foundations/content-and-microcopy.md](../foundations/content-and-microcopy.md) and
> [../interaction-design/feedback-systems.md](../interaction-design/feedback-systems.md) both
> name as the canonical bad error message — listing it here as a *recommendation* contradicted
> the rest of the folder. Even "the model didn't return a response" is thin; say what failed,
> and always say what was preserved.

## Failure Recovery

When AI fails, the recovery path matters more than the failure itself:

1. **Acknowledge the failure** clearly (don't show a generic spinner that never resolves)
2. **Explain what happened** in plain language
3. **Provide a recovery action** (retry, rephrase, switch mode)
4. **Preserve context** (don't lose the user's prompt or previous work)

## References
- NN/g articles on trust in digital products
- Microsoft HAX Toolkit (human-AI interaction guidelines): https://www.microsoft.com/en-us/haxtoolkit
