# HWGodWin Forensic Research

Independent technical and forensic research into HWGodWin software behavior, artifacts, and security-relevant observations.

## Purpose

This repository preserves research material in a form that can be independently reviewed and reproduced. The goal is to separate observed evidence from interpretation and document the methodology, limitations, and results of the investigation.

## Published research

- [`INFORME_FINAL.md`](./INFORME_FINAL.md) — principal forensic report.
- [`Activation and distribution forensics`](./analysis/activation-and-distribution-forensics.md) — analysis of activation/licensing behavior, remote distribution, persistence and tracking surface.
- [`Consolidated technical findings`](./analysis/TECHNICAL_FINDINGS.md) — current synthesis incorporating the stronger payload/runtime evidence and superseding earlier hypotheses where necessary.

## Selected evidence

- [`Network observations`](./evidence/network-observations.txt) — selected Goodwin-side POST endpoints observed during instrumented analysis; no live credentials included.
- [`Loader sandbox success log`](./evidence/sandbox-loader-success.log) — successful stubbed loader/chunk-read execution.
- [`Known sandbox error`](./evidence/sandbox-known-error.log) — preserved analysis failure rather than discarded evidence.

## Evidence standard

Claims should be supported by reproducible evidence whenever possible. Relevant artifacts should preserve their origin, acquisition context, cryptographic hash, and relationship to the corresponding finding.

Anything that cannot be demonstrated from the available evidence must be identified as **NOT VERIFIED** rather than presented as an established fact.

## Methodology

The investigation may include static analysis, behavioral observation, artifact inspection, controlled execution, comparison of outputs, integrity verification, and reproduction of relevant findings. Tests are performed only on systems, software, and material for which the researcher has authorization.

## Publication boundary

The public repository intentionally favors reviewable findings and selected evidence over redistribution of operational payloads. Raw multi-megabyte encoded tables, reconstructed cheat code, live credentials, user-specific session material and unnecessary sensitive data are not published here.

## Reproducibility

Where practical, findings include enough information for an independent reviewer to reproduce the observation without relying solely on conclusions in the report. Environment-specific limitations and unavailable dependencies should be documented explicitly.

## Repository structure

- `analysis/` — technical analysis derived from evidence.
- `evidence/` — selected public evidence, logs and observations.
- Root reports — principal research reports retained at their published paths.

## Responsible publication

Public material should not contain credentials, personal information, private customer data, secrets, or other unnecessary sensitive information. Potentially sensitive findings should preserve sufficient evidence for technical review while avoiding publication of material that creates unnecessary risk.

## Disclaimer

This repository is provided for legitimate security research, forensic analysis, interoperability, education, and defensive purposes. Findings describe observations made in the documented environment and should not automatically be generalized to other versions or configurations.

## Status

Research archive active. Reports and supporting evidence are added after review for publication.