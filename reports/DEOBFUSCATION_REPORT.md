# HW Goodwin 5.1 — Deobfuscation Report

This report records the static/sandbox deobfuscation work performed against the HW Goodwin 5.1 userscript loader.

## Sample

- Size: 39,759 bytes
- SHA-256: `2b84bc148e8e0624d15b6cab4a6f877f1a68237d57ce240e8ea2b2ff8f585`
- Userscript: HW Goodwin v5.1
- Target: Hero Wars

## Loader behavior

The sample is a Tampermonkey userscript configured for `hero-wars.com` and the corresponding Facebook-hosted game surface. It runs at `document-start` and declares no privileged userscript grants.

Analysis reconstructed a loader that rotates among four Goodwin mirrors, retrieves `/updates/loader.php`, injects the returned JavaScript into the document, and maintains fallback state using Cache Storage and localStorage.

## Obfuscation

Observed layers included LZ-String, a compressed dictionary, multiple base91 alphabets, an encoded string table, and a lazy decoding cache. A deterministic decoder and sandbox instrumentation were used to reconstruct identifiers and strings.

## Validation

The reconstructed loader was exercised with network and browser operations stubbed. The test environment reproduced the expected mirror fetch, cache/localStorage behavior, script-injection path, and game-object hooks without sending requests to the real service.

## Limits

The initial loader analysis did not establish the behavior of the remotely served payload. Later payload analysis superseded that limitation and is documented separately in the repository.

## Evidence policy

The original third-party executable script and reconstructed operational payload are not redistributed here. Their hashes and analysis results are retained so observations can be tied to the analyzed samples.
