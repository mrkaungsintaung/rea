# REA workbench and message capture — V1.0

Date: 2026-10-30 (Asia/Bangkok). Owner: personal fork mrkaungsintaung/rea.

## Two uses of REA

1. Development workbench: open this fork in Cursor Cloud to inspect targets, extend REA when needed, and preserve research evidence.
2. Reusable development tool: call the built CLI or stdio MCP from other project environments. REA is NOT a required production service for the capture extension, PM Auto, Yourbox, or MASA.

`main` tracks morluto/rea upstream. `workbench` is this fork's default development branch and carries its Cursor setup. Do not force push either branch. New task PRs target workbench, never the upstream project unless explicitly requested.

## Current baseline and verification

Upstream synced to main: `317bf3fe235e655b2d752d3be261cc0da7d00f2b` (REA 6.8.0).

Executed in the preparation environment:

- npm ci: passed.
- npm run build:cached: passed (upstream bundler warnings).
- CLI --version: 6.8.0.
- mcp doctor --json: healthy; 233/233 tools, 6/6 prompts, identity and target-free request passed.
- Static JavaScript fixture: blocked by this host's procps process-ownership inspection (`fatal library error, lookup self`). Not a successful analysis.

Cursor: existing environment `reverse engineering`; .cursor/Dockerfile and .cursor/environment.json added on workbench; personal stdio MCP `rea-workbench` added and enabled. Initial run failed; environment reported its config missing on the old default branch. Default changed to workbench and a branch build requested. Successful VM build, actual client MCP calls, and static JS analysis still require verification. Cursor also displays a Cloud Agent Degradation notice. Do not describe configured as runtime-verified.

## Cursor development use

The Dockerfile pins Node 24.22.1 and npm 11.16.0 and installs procps and build dependencies. The install command checks Node against .nvmrc, installs locked dependencies, builds REA, runs adapter doctor, and installs a symlink at $HOME/.local/bin/rea-workbench. Existing launcher paths are backed up.

The cloud MCP registration is:

```json
{
  "mcpServers": {
    "rea-workbench": {
      "command": "bash",
      "args": ["-lc", "exec /usr/local/bin/node \"$HOME/.local/bin/rea-workbench\" mcp"],
      "env": {
        "REA_MCP_INPUT_SCHEMA_PROFILE": "compact",
        "REA_MCP_OUTPUT_SCHEMA_PROFILE": "compact"
      }
    }
  }
}
```

Registration is account-side in Cursor Cloud; a committed IDE MCP file alone does not establish Cloud activation. This launcher exists only in VMs whose setup created it. Leave it disabled in environments without REA.

## Reuse from another project

Each isolated VM needs its own installation and local target files. A repo fork URL is not a remote MCP endpoint. Options:

- Add REA to a Cursor multi-repo environment and explicitly run its install/build there; single-repo .cursor config is not automatically applied to multi-repo groups.
- Clone a verified full SHA into a separate tools directory, install/build there, and configure the consumer to launch its absolute Node and REA script paths.

Example, from an empty tools directory (requires compatible Node/npm and procps already installed):

```bash
git clone https://github.com/mrkaungsintaung/rea.git rea-tool
cd rea-tool
git checkout --detach 317bf3fe235e655b2d752d3be261cc0da7d00f2b
npm ci
npm run build:cached
node scripts/rea.mjs mcp doctor --json
node scripts/rea.mjs analyze-javascript-application /absolute/path/to/extracted-target --format json
```

For MCP, use the absolute path returned by `command -v node` as command, and the absolute `scripts/rea.mjs` path plus `mcp` as args. Do not commit machine-specific paths or target artifacts to consumer repos. No changes have been made to PM Auto, Yourbox or MASA. Consumer activation is a separate environment task.

## Updates

A daily change watch was created for upstream and the extension listing. main may only fast-forward when ancestry confirms no fork commits would be lost. Merge upstream updates into workbench on an update branch, preserve local configuration, review toolchain changes, build and smoke-test before updating consumer pins. Active analyses stay on their installed commit. A scheduled check cannot guarantee updates if GitHub permissions or service access fail; report the blockage.

## Reference extension investigation

Reference: [Teams Message Extractor - Chat Export](https://chromewebstore.google.com/detail/teams-message-extractor-c/hemdpkoomkdphclendigjhelkaknjddb), publisher Adam Toms LTD, listing version 2.9.3. The listing claims local processing, automatic scrolling, direct/group chats and channels, nested replies, reactions, CSV/HTML/Markdown, images and attachment ZIP. These are publisher claims. The installed package and runtime have NOT been inspected. Free pricing does not establish an open-source license.

Next evidence needed: obtain the official package, record version and SHA-256, read manifest/permissions, trace message selection, scrolling, deduplication, export and attachment paths with REA, and record observed facts separately from inferences. Use synthetic/test conversations. Do not upload workplace messages to a public repo or model provider. Build an original implementation from the requirements and verified behavior; preserve third-party licenses if any licensed code is reused.

## Original capture product proposal

Separate TypeScript Manifest V3 extension, Burmese/English interface, local-first processing. Use ordinary browser extension APIs and a separately testable capture/export core. Reuse REA for development research; do not embed it in the shipped extension.

V0.1: user chooses the open conversation, starts/stops capture, previews the collected text, and exports JSON/Markdown/UTF-8 CSV. Preserve Burmese Unicode as received; never silently convert Zawgyi. Display progress, skipped/unknown fields and partial coverage. Scrolling alone cannot prove complete history. Preserve observed raw timestamps; normalize only when timezone/date are known. DOM virtualization requires collecting incrementally and deduplicating without merging different messages with identical text. Replies need stable parent relationships when available.

Later: safe HTML transcript, resumable traversal, reaction and attachment metadata, optional attachment download, and explicit authenticated imports to a selected project/workspace. Escape HTML and protect CSV cells against spreadsheet formula execution. Do not collect session tokens or silently send messages to cloud services. An approved enterprise Microsoft Graph connector is a separate adapter; list/paging/reply coverage and tenant consent must be verified. A local DOM extractor does not automatically inherit enterprise compliance guarantees.

### Shared message envelope (proposed, not implemented)

| Field | Purpose |
| --- | --- |
| schema_version, capture_id | Versioned import and idempotency |
| source.platform, source.url, source.conversation_id | Provenance; unknown IDs remain null |
| source.workspace_id | Source boundary; never infer destination tenancy |
| message_id, parent_id, thread_id | Message identity and reply structure |
| sender.display_name, sender.id | Preserve observed identity; null when unavailable |
| timestamp_raw, timestamp_utc | Keep original and qualified normalization |
| text_original, content_format | Preserve Burmese/English content |
| reactions, attachment_refs | Explicitly observed metadata only |
| captured_at, extractor_version | Collection provenance |
| coverage.status, stop_reason, warnings | Partial/unknown capture and limits |

Destination tenant/project authorization is established at import by each product, not trusted from extension fields. Preserve original source IDs separately from destination IDs. Add per-message provenance, deduplication, updates and provenance-linked chunks before retrieval/RAG.

| Product | Potential authorized use |
| --- | --- |
| PM Auto | Decisions, risks, action candidates with message citations; require confirmation before writing project actions |
| Yourbox | Searchable conversation imports and knowledge capture with source links |
| MASA | Seller-selected product/support knowledge; maintain tenant separation and exclude unrelated private chats |
| Individual | Local Burmese/English archive and search |

## Next acceptance gates

1. Cursor build succeeds, source CLI runs, actual client calls MCP, synthetic JS analysis succeeds.
2. Consumer environment installation verified before enabling REA there.
3. Official reference package inspection recorded with evidence and unknowns.
4. Separate capture project requirements and UX reviewed before implementation.
5. Test Burmese text, duplicates, edited/deleted messages, long/virtualized histories and incomplete thread loading before any completeness claim.
