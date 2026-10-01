# SailPoint (planned, Phase 5)

SP-Config JSON (SailPoint configuration export format) for a SCIM 2.0 source, its schema, and provisioning policy, plus the export, diff, and import pipeline.

Runs against a Stoplight Prism mock built from `sailpoint-oss/api-specs`, not a real SailPoint ISC (Identity Security Cloud) tenant. Validates pipeline mechanics only. Secrets are stripped on export and injected at import; never committed.
