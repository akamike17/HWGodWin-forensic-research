# Consolidated Technical Findings

## Scope

This note consolidates the strongest reproducible findings from the HW Goodwin v5.1 loader and the remotely delivered `loader.php` payload. It is intended as a reviewable technical index, not as an exploitation guide.

## Loader identity and behavior

The examined local artifact is a Tampermonkey userscript named **HW Goodwin v5.1** for Hero Wars. The local script acts primarily as a bootstrap loader: it installs runtime hooks, selects one of several `goodwin.best` mirrors, downloads `/updates/loader.php`, injects the returned JavaScript into the page, and stores a cached copy.

Observed loader persistence includes Cache Storage (`HW-Goodwin-loader`) and `localStorage` state for the last-used mirror.

## Obfuscation

The loader uses multiple layers including LZ-String, a compressed dictionary, multiple Base91 alphabets, a string table and lazy decode caching. The decoding process was reconstructed and cross-checked in a sandbox.

The remotely delivered payload is substantially larger and uses an additional obfuscated string/pointer structure. Analysis identified a 15,015-element pointer/string array and a separate encoded table containing 16,182 entries.

## Remote payload

The server-delivered payload was observed as a roughly 22.4 MB JavaScript artifact. Analysis reconstructed the decoder pipeline and recovered readable strings, identifiers and network-related behavior without relying solely on plain-text keyword searches.

## Network behavior

Runtime-oriented analysis identified a generic request routine that constructs URLs under:

`https://<selected-goodwin-domain>/request/<operation>.php`

Requests use HTTP POST and include timeout/retry behavior. The domain can rotate through the configured Goodwin mirrors, with the selected domain persisted locally.

Observed operation names include authentication, rating, event/invasion functions, uploads, quiz functions and activation-related requests. These observations establish that the delivered payload contains its own Goodwin-side network channel in addition to traffic associated with the game.

## Authentication-related material

Decoded identifiers and runtime context show references to game/session authentication material (including fragments resolving to `auth_key` / FlashVars-related state) used as part of request construction. This repository intentionally does not publish live credentials or user-specific session values.

## Licensing / activation conclusion

The small local loader itself does not contain a conventional local license-key validation mechanism. Later analysis of the remotely delivered payload found activation-related server endpoints, so the earlier hypothesis that all Goodwin-side behavior was limited to the bootstrap loader was superseded by stronger runtime evidence.

The evidence supports a server-mediated model rather than a simple local patch/key check. This is a behavioral conclusion, not a bypass procedure.

## Evidence quality and limitations

The loader behavior was reproduced in a controlled JavaScript sandbox with network and browser primitives stubbed or instrumented. A sandbox error involving an undefined `.call` target was also preserved as evidence rather than silently discarded.

Some behaviors remain environment-dependent and were not validated against a live game session. Those items should be treated as **NOT VERIFIED** unless independently reproduced.

## Publication boundary

Raw multi-megabyte encoded payload tables, reconstructed cheat code and material that would unnecessarily redistribute an operational game-automation payload are intentionally excluded from the public repository. The repository instead publishes findings, mappings and selected evidence needed to review the forensic conclusions.