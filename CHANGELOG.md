# Changelog
 
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
 
## [0.1.0] - 2026-04-08
GPT 5.4, UI redesign, chat history improved, mention files with "@", responses streaming
 
## [0.0.8] - 2026-03-18
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
