# Optional owner-only concept image

Use only when an image tool, its use and the output location are authorized for the task. The image helps the owner understand a proposed screen. It is not an Implementer input, implementation specification, product asset or proof of a rendered interface. Its absence is not a design gate failure.

## Separate audience and records

Keep the generated image and its generation prompt, paths, links, image-specific feedback and review notes in a separate owner-facing record at the authorized owner-review location. That record goes only through the authorized owner-review route. It is not a section or attachment of `design.todo.md` and is not appended to the implementation-facing Designer Output.

The image record may identify its related screen/state and design version so Designer and Orchestrator can understand it. The dependency is one-way: the implementation specification and execution package must not link back to, embed, name, describe or disclose the existence of this preview. Do not put its path, an image identifier, generation instructions, "see the mockup" or a placeholder for it in the design TODO, software TODO, implementation-facing Output, brief or shared downstream attachment.

Retain private provenance in the owner-facing record rather than deleting history. Keep those records outside the Implementer's authorized read scope; do not forward their containing directory, a broad discovery request, a raw owner review or an unsanitized coordination/evidence record. Use an explicit allowlist of actual implementation inputs. Tool visibility does not make private owner-review material an authorized implementation input.

## Prepare and discuss the illustration

Use the actual design candidate and stated screen/state/viewport to create the optional illustration. Inspect the result with the available tool, identify unverified details or mismatches and label it as a concept for owner discussion. Do not claim accurate pixel layout, accessible contrast, working interactions or tested responsive behavior merely because the image looks convincing. Record actual creation/viewing evidence or NOT RUN; do not manufacture an image when no tool was used.

No image generation, external upload, paid action or image tool setup is authorized just by this procedure. If no image is requested/appropriate or a permitted tool is unavailable, continue the full text-specification route without inventing a visual artifact.

## Translate owner feedback into complete text

Convert any owner decision into exact written specification changes: elements, token values, measurements/relative constraints, responsive intervals, states, content and observable criteria. Ask only for consequential ambiguity that cannot be resolved from the actual decision and project sources. New business/API/security scope still goes through Planner/Orchestrator and the actual owner; the image does not authorize it.

Update the canonical design TODO/version and implementation-facing Output with those complete textual decisions. Cite the owner decision reference and affected requirement/design IDs without mentioning the preview or exposing its record. Never leave "match the picture" or image-derived gaps for Implementer to infer. Reconcile software tasks and actual design gate evidence when affected; changes do not inherit an old review automatically.

## Implementation handoff check

Designer and Orchestrator inspect the actual files being delivered. Confirm they contain no preview image, thumbnail, link/path, caption, existence flag, hidden metadata, generation prompt, indirect preview reference or dependency. Do not attach the private owner record through a source inventory or raw review report. Tester gate evidence forwarded to Implementer must also be a text-only, implementation-safe record.

The Implementer must be able to execute the complete approved specification without knowing that a concept image exists. Production assets genuinely required by the product, such as an approved logo, remain distinct from an owner-only concept image and need their own explicit asset specification/authorization. This procedure does not hide a required runtime asset or turn a concept image into one.
