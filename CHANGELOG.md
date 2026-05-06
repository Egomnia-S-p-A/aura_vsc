# Changelog

## [0.1.18] - 2026-04-30

Iteration consolidating four UX and architecture changes around the chat composer and the agent registry. The chat toolbar now ends with a gear-icon Settings menu that gathers three per-chat controls in one place: the approval mode (Ask vs Auto, replacing the standalone "Approvazioni" pill that previously cluttered the toolbar), an Egomnia Compressor toggle that promotes the user-local `~/.aura/skills/Compressor` skill out of the skill picker and into a single switch (the skill is hidden from the picker so it cannot be selected twice, and when on it claims one of the two pinned-skill slots with priority over manual selections), and a new Terminal Execution toggle that gates the `run_terminal_command` tool at the extension layer. When terminal execution is off (the **default** for new chats — explicit user opt-in is required to let the agent run shell commands), the tool returns `{ status: "execution_disabled", suggested_command: ..., explanation: ... }` instead of spawning a process; the agent prompt has a dedicated section instructing the model to surface the command as a fenced ```bash code block with a one-line explanation rather than retrying, and a new orange "Comando suggerito" terminal card with the command + hint is appended to chat as a visual mirror. The proposed-terminal markdown chip in agent replies was redesigned from a heavy two-row card with a "COMANDO SUGGERITO" header into an inline single-line command bar (left accent stripe, `$` prefix in accent color, monospace command, `Copia` + `Esegui` ghost buttons floated top-right). When the terminal toggle is off the `Esegui` button is hidden entirely and the chip carries a tooltip explaining the user's own choice — eliminating the previous contradictory CTA where a green Run button was visible while execution was disabled. The terminal card itself was redesigned in Claude-Code style: by default cards are now collapsed with a one-line `$ <command>` summary plus a chevron, expanding on click to reveal the full body (badges, output streams, exit code, Stop button); the expanded state is persisted per session in webview state and the Stop and chevron buttons stop click propagation so the header-wide click-to-toggle never fights interactive children. Backend-side, the `aura_pdf` and `aura_docx` document agents were removed from the registry and their five tools (`extract_pdf_text`, `extract_pdf_metadata`, `extract_docx_text`, `extract_docx_structure`, `convert_document`) were promoted to the baseline tool set in `AgentDescription` so every agent can read PDF/DOCX directly without a routing handoff; `main_agent_prompt.txt` gained a "DOCUMENT TOOLS" section codifying the PDF/DOCX flow (always start with `extract_pdf_text`, surface `empty_text_layer: true` instead of fabricating content, reject PDF as a `convert_document` target, surface `pandoc_not_installed` cleanly), and the orphan `agent_pdf.txt` / `agent_docx.txt` prompt files were deleted.

## [0.1.17] - 2026-04-30

RAG embeddings provider now follows the LLM the user picks in chat instead of being pinned globally in `.env`. Selecting an Ollama LLM from the model dropdown automatically routes RAG embeddings through the same Ollama host (`bge-m3:latest` by default, `OLLAMA_BASE_URL`); selecting any other provider (OpenAI, Anthropic, generic OpenAI-compatible) keeps RAG on OpenAI embeddings. The new `services/rag_embeddings.py::resolve_rag_embeddings(api_key)` helper is the single source of truth — `BaseRAGManager` no longer reads the global `RAG_EMBEDDING_PROVIDER` env on its own. The env var still works as an explicit override (`ollama` or `openai`) for staging environments that want to pin a provider regardless of api_key, but the per-job dispatch is the default. The Chroma `storage_setting` already includes `(provider, model, base_url)`, so switching provider mid-conversation triggers an automatic index rebuild — no manual cache wipe needed. Avoids the previous footgun where an Ollama-only setup on internal Egomnia hosts would still call out to OpenAI for embeddings.

## [0.1.16] - 2026-04-29

Context-window progress bar in the chat composer now grows monotonically across turns instead of rewinding whenever a new turn happens to be smaller than the previous one. The underlying server-side metric `context.used_tokens` previously reflected only the current turn's tokens, which on providers that don't stream a cumulative `input_tokens` value (Anthropic, several OpenAI-compatible backends) made the bar look stuck or oscillating. It now reflects the cumulative conversation context: live updates during streaming compute `committed.total_tokens + running.turn.total_tokens`, finalization snapshots use the post-commit `committed.total_tokens`, and on backend restart the rehydrate path reconstructs the same cumulative value by summing all jobs' `total_tokens` for the history. Manual context compaction continues to override the value to a smaller post-compact size, so the bar correctly drops after a `/compact`. The server-side database column `agent_jobs.context_used_tokens` is intentionally left storing the per-turn value (no migration needed); cumulative reasoning is centralized in the metrics aggregator only.

## [0.1.15] - 2026-04-29

Approval visibility fix when the Aura chat view is hosted in a non-default container — typically when a developer drags Aura into the Primary Side Bar to live next to (or in place of) the Explorer file tree. In that layout the previous reveal command (`workbench.view.extension.egomnia-aura-sidebar`) targeted an empty container and silently failed, so approval cards were appended to the chat but the view was never brought to the front, making mid-job approvals look broken. The picker now uses the per-view focus command `egomnia-aura.chatView.focus` (which VS Code auto-registers and which works regardless of the hosting container), with the original container command kept as a fallback for older builds, and an explicit `WebviewView.show(true)` call as a final layer. Whenever an approval card is appended, the chat is also force-scrolled to the bottom so the freshly added card never lands below the fold in long conversations. If after the reveal attempt the webview is still not visible (sidebar collapsed, container swapped away), Aura now surfaces a non-modal information toast with a "Show" button that re-runs the reveal flow, so the user is never silently stranded waiting for an invisible approval. Internal `.aura/` metadata writes (workdir id, codebase RAG cache) no longer auto-open in the editor: they were stealing focus during bootstrap or mid-job without any user value, so the auto-open is now skipped for any path that contains a `/.aura/` segment while every real user-facing file write keeps opening as before. The Aura activity bar icon also got a small but meaningful fix: it was previously a colored PNG, which VS Code could not theme automatically, so the Egomnia "e" looked out of place next to the monochrome Files / Search / Source Control icons and worse, when a developer dragged the chat into the Primary Side Bar (next to the Explorer) or the Secondary Side Bar, the colored "e" would float on the activity bar disconnected from any content. The icon is now served from a new `resources/icon-mono.svg` that uses `fill="currentColor"`, so VS Code tints it correctly on light and dark themes and it integrates visually with the rest of the activity bar; the colored marketplace icon stays unchanged. When Aura suggests a shell command in chat as a fenced code block (`\`\`\`bash`, `sh`, `zsh`, `console`, `terminal`, `cmd`, `powershell`, etc.) instead of running it directly, the block is now decorated with a small terminal-icon header labeled "Suggested command" and integrates Copy and Run buttons. Clicking Run hands the command to the same pipeline used by the agent's `run_terminal_command` tool: it spawns under the workspace scope, honors the active approval mode (auto skips the prompt, ask requires confirmation), and streams its output into a real terminal card in chat — exactly as if the agent had executed it. Multi-line snippets keep the Copy affordance but disable Run (the underlying tool refuses multi-line on purpose), so users get a clear hint to paste them in a real terminal instead.

## [0.1.14] - 2026-04-29

UX polish round on the chat composer and toolbar. The “Aura sta elaborando…” / “Aura sta lavorando…” split was unified into a single coherent label across thinking message, progress notification, and job placeholder (it/en/es). The New Chat header button got a chat-bubble icon (with a small inner cross) replacing the bare `+`, making its meaning more immediate. The message composer now persists the in-progress draft across page changes and webview reloads via `vscode.setState` with a 250 ms debounce, restoring the typed text on rehydrate so users no longer lose what they were writing when navigating away. The agent dropdown menu now exposes a per-agent info icon: clicking the inline `i` expands the item showing the agent description, helping users understand the difference between `aura_doc`, `aura_mvp`, `aura_pdf`, etc., directly from the picker. The same info-icon pattern was extended to the skills dropdown: each user skill loaded from `~/.aura/skills` shows a curated, localized one-sentence summary (it/en/es) maintained in `vscode_extension/src/skills/skillSummaries.ts` for the 12 commonly-installed skills (Angular, Compressor, Frontend Pro, Git, Nest, NextJS, React Native, Self Improve, Shadcn, Skill Creator, Spring Boot, UI-UX Pro), with automatic fallback to the first sentence of the SKILL.md frontmatter for custom skills — no ellipsis, no truncation, the sentence always renders in full and matches the active extension language. To make this work even when the picker is at the 2-skills cap, skills are no longer hard-disabled at the DOM level: selection is gated in JS via `aria-disabled`, so the info icon stays clickable on capped skills too. The thinking indicator now persists when the user navigates away from the chat and comes back while a job is still running: `setLoading(true)` is the single point of truth for the “Aura sta lavorando” message and automatically re-shows it after a webview rehydrate, so an in-flight job is never silently invisible after a page change. Hardening: the `thinkingDiv` webview variable was hoisted alongside other top-level webview state to remove a temporal-dead-zone hazard that could otherwise abort the inline-script bootstrap when `setLoading(true)` ran during `restoreState()`, leaving the Aura sidebar visually rendered but completely unclickable; `setLoading` is now wrapped in defensive try/catch boundaries so a future renderer hiccup cannot break the global click model again.

## [0.1.13] - 2026-04-24

Session cost pill moved from the bottom metrics strip into the chat header, placed to the left of the history (clock) button for at-a-glance visibility while typing. Cost formatting is now approximated for readability: amounts below $0.01 render as `<$0.01`, amounts under $1 use 3 decimals, and larger values use 2 decimals — no more six-digit fractional costs.

## [0.1.12] - 2026-04-23

Fixed an approval prompt that incorrectly fired during the very first Aura bootstrap on a new workspace, when Aura had to create its own internal `.aura/` metadata folder (in particular `.aura/.workdir_id` for the codebase RAG). Internal `.aura/` metadata writes initiated by Aura itself now bypass the approval gate, while every write requested by the agent on user files keeps going through the standard ask/auto approval flow unchanged. The bypass is strictly limited to paths inside `<workspace>/.aura/` and only when the server marks the write as internal.

## [0.1.11] - 2026-04-22

Two new specialized document agents exposed in the chat dropdown: **Aura PDF** (lettura, estrazione testo e metadati, conversione PDF verso Markdown/HTML/TXT via pandoc) and **Aura DOCX** (estrazione testo e struttura, conversione DOCX verso Markdown/HTML/TXT). The VS Code extension now supports a binary-safe mode on the existing `read_file` WebSocket call so document bytes can be fetched from the workspace without polluting the text contract.

## [0.1.10] - 2026-04-22

Added a compacted-context badge to the VS Code chat composer metrics strip, so users can immediately see when older conversation turns were compacted. Also added structured backend compaction telemetry for native OpenAI and Aura-native paths.

## [0.1.9] - 2026-04-22

Manual context compaction actions in the VS Code chat UI: a dedicated toolbar button plus `compact` / `/compact` command in the composer trigger backend context compaction for the active conversation, update session metrics immediately, and append a compacting outcome message inside the chat.

## [0.1.8] - 2026-04-21

Session metrics UI simplified in the VS Code chat composer: the previous multi-card panel is now reduced to a single context-window progress bar plus an optional session-cost pill shown only when reliable pricing exists. History and live job hydration behavior remain unchanged.

## [0.1.7] - 2026-04-21

Terminal actions from Aura chat: workspace/global scoped commands executed through the VS Code extension host with approval gating, foreground/background modes, and live interactive terminal cards rendered in the conversation. Cards stream stdout/stderr in place, show scope/mode badges, expose per-output copy buttons, and include a Stop control that cancels both foreground and background commands in real time with a stopped-by-user indicator. Still-running sessions are automatically terminated when switching chats or starting a new one, and the agent receives a coherent `status: 'stopped'` result when the user aborts.

## [0.1.6] - 2026-04-20

Classic developer welcome screen for empty chats; Skills tutorial moved behind a dedicated action inside the Skills dropdown; Tutorial Skills CTA refined with a more iconic Aura-green treatment; welcome and tutorial refined into single-card monoblocks; and localized tutorial copy updated across supported languages with clearer Windows/macOS setup guidance

## [0.1.5] - 2026-04-15

Skills splash simplified to only show the local filesystem paths where skills must be stored, with localized Windows and macOS path guidance

## [0.1.4] - 2026-04-15

Skills splash refined with explicit guidance on where to select skills in the Aura chat UI, including separate Windows and macOS guidance while keeping the onboarding compact and static

## [0.1.3] - 2026-04-15

Static skills tutorial splash for empty chats, compact Aura-branded onboarding, localized copy, and removal of the previous generic empty state

## [0.1.2] - 2026-04-15

Interactive approval flow for workspace and skill mutations, approvals dropdown in chat toolbar, diff-like approval preview, repeated-denial safeguards, and webview hardening against inline-script/bootstrap failures

## [0.1.1] - 2026-04-10

Claude/Anthropic models, provider-based model catalog, user skills from `~/.aura/skills`, skill picker with active chip, compact model labels without provider

## [0.1.0] - 2026-04-08
GPT 5.4, UI redesign, chat history improved, mention files with "@", responses streaming

## [0.0.8] - 2026-03-18
Minor fixes

## [0.0.7] - 2026-03-06

Minor fixes

## [0.0.6] - 2026-03-06

Enable async agent job execution.

## [0.0.5] - 2026-02-25

Add multilanguage support: italian, spanish, english.

## [0.0.4] - 2026-02-24

minor fixes.

## [0.0.3] - 2026-02-09

Fix relevant issue over api_key setting, update readme, several minor fixes.

## [0.0.2] - 2026-02-03

Readme changes.

## [0.0.1] - 2026-02-03

First version.
