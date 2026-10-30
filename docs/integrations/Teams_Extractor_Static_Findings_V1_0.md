# Teams extractor: static findings and original product direction — V1.0

Inspection date: 2026-10-30 (Asia/Bangkok).

This report supersedes the package-not-inspected status in REA_Workbench_and_Message_Capture_V1_0.md. It records static source observations, not runtime verification or a security audit.

## Artifact identity

Official listing: https://chromewebstore.google.com/detail/teams-message-extractor-c/hemdpkoomkdphclendigjhelkaknjddb

Downloaded from the Chrome Web Store update service, following its redirect; not installed or executed. CRX3 archive, manifest version 2.9.3. SHA-256:

`c7a41038613a18f280b495fc2c3628b8b2dc2a8213413982601a4ccbb3c73dac`

Archive checksum was computed; CRX signature verification was not performed. The downloaded third-party archive and source are not committed to this public fork. Static review used direct file reading; the local REA JavaScript workflow was blocked by the preparation host's process-inspection limitation. No REA-derived analysis result is claimed for this package.

## How the extension is organized

| Component | Observed role | Evidence location |
| --- | --- | --- |
| manifest.json | Manifest V3; popup, content script, background service worker | manifest_version, action, content_scripts, background |
| content.js | Inspect rendered Teams DOM, collect message records, scroll chats, traverse channel threads | extractMessage around line 1150; extractVisibleMessages 1242; autoScrollAndExtract 1357; autoScrollAndExtractChannel 2020 |
| popup.js / popup.html | User controls, progress and exports | downloadCSV 1144; downloadHTML 1229; downloadMarkdown 1256 |
| background.js | Extraction/download coordination and attachment jobs | extractionStatus 50; download fetch routines around 732–774 |
| zip.js | Local archive support | present in archive; not comprehensively reviewed |
| utils.js | Date conversion helper | toIso |
| licensing.js | Optional paid-tier scaffolding, disabled in this package | LICENSING_ENABLED near line 39 |
| _locales | 20 interface locale directories, no Burmese locale | archive inventory |

The manifest requests activeTab, scripting, storage and downloads. Host access includes Teams, SharePoint, OneDrive-related URLs and attachment/proxy hosts. LemonSqueezy is optional host access. A future text-only original version can start with fewer permissions; attachments should be a separately justified capability.

## Capture mechanism

Chat collection queries rendered message-body elements, reads nearby author/time/body information, replaces emoji images with their text alternatives in a cloned DOM fragment, and preserves quoted-message fields. A Set tracks message IDs and a Map stores the records. Repeated sightings may refresh attachments, but the observed early return does not re-read already-seen message text. Our implementation should explicitly handle changed messages.

The scrolling loop collects what is loaded, scrolls upward, waits for newly rendered records, and stops on repeated no-growth passes, message count, or user stop. The source defaults include a 10,000-message ceiling and a 2-second scroll delay, but stored settings can change behavior. These are heuristics and bounds, not proof of a complete archive.

Channel capture has a separate feed/thread path rather than relying on the chat loop. It discovers root posts, enters thread views to inspect replies, returns to the feed, and reacquires scroll containers after navigation. This separation is useful for our adapter design because virtualized feeds and replies need different traversal states.

## Exports and attachments

CSV uses UTF-8 with a BOM for spreadsheet compatibility, and preserves thread columns for channel captures. Markdown uses a UTF-8 text Blob; HTML builds a local transcript and escapes displayed text. Unicode content support and Burmese interface localization are separate concerns: absence of a Burmese locale does not prove Burmese messages fail to export. Actual Burmese rendering and spreadsheet round-trips are untested.

Attachment paths inspect DOM metadata and image elements; some image capture uses canvas, while other image/file paths fetch authenticated URLs using browser credentials. The source has an inline-image memory cap and a bounded wait for pending captures. Local processing therefore does not mean zero network traffic: downloading Teams attachments is network activity. This review does not establish whether all message-data egress paths are absent.

Licensing code exists, but its master flag is false in this artifact and checkout configuration contains placeholders. This supports the user's observation that the current package is free/unlocked; it does not establish future prices, launch plans, or permanent free availability. No license checks were modified or bypassed.

## What to reuse conceptually

Build an original TypeScript Manifest V3 extension with Burmese/English UI, user-triggered start/stop, a local preview and JSON/Markdown/CSV export. Separate:

1. Teams DOM adapter (selectors, traversal and observed source IDs).
2. Neutral message core (identity, thread relationships, original text and provenance).
3. Exporters (Unicode, HTML escaping, CSV formula safety).
4. Explicit import adapters to PM Auto, Yourbox and MASA, implemented in each destination's authenticated context.

Use a traversal state machine with progress, cancellation, retries and an honest stop reason. Report coverage as partial/unknown when history or replies cannot be established. Preserve raw timestamps and only add normalized values when qualified. Keep original Burmese text; any Zawgyi conversion must be a visible, separate transformation.

REA should help investigate and compare future package versions during development. Neither REA nor the third-party extension should become a required production dependency of our capture product. No third-party code is copied into our implementation; free distribution is not an open-source license.

## Project applications

- PM Auto: turn selected project conversations into evidence-linked decision, risk and action candidates. Human confirmation precedes changes to project records.
- Yourbox: import selected conversations into the general capture/search knowledge flow.
- MASA: allow a seller to intentionally import relevant support/catalog knowledge, with tenant authorization enforced on the receiving side.
- Individual: local Burmese/English archive and search without mandatory cloud upload.

A separate enterprise Microsoft Graph adapter is an alternative where tenant consent is available. Channel roots and replies need explicit pagination/coverage handling; API permissions and personal-account support differ. Do not treat a browser extractor as a replacement for authorized enterprise retention/export controls.

## Still unverified

No Teams conversation, employer data, popup interaction, network execution, thread completeness, attachment ZIP, Burmese round-trip, long-history performance, or live-selector compatibility was tested. Reference package static analysis informs requirements; it does not establish our future extension works. Next gate: synthetic/test Teams fixtures and a separate original product specification/UX before implementation.
