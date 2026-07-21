# AI Fix Notes

Session: seq-1784619284339-tfm5zfpvq
Repository: Ncorp30/jsmn-C

## Summary

- Detected actionable issues: 6
- Issues with proposed PR changes: 3
- Issues requiring manual review: 3
- Automated fix mode: partial / safety-first

## Safety Policy

High-priority findings touching security, authentication, credentials, network behavior, dependency safety, privacy, request handling, or response handling are not silently edited by the agent. They are listed for manual review unless the workflow can generate a bounded, low-risk change with enough context.

## Proposed Changes Included in This PR

- [1] (medium) jsmn/example/jsondump.c: The code reports realloc failure using errno, but realloc failure is not always accompanied by a meaningful errno value in a portable way. Error handling and diagnostics may be misleading. Capture and report failures in a way that does not rely on errno being informative.
- [2] (medium) jsmn/example/simple.c: The helper jsoneq() calls strlen(s) on every comparison. In hot paths or repeated token comparisons this adds avoidable overhead. Cache the length once or pass it alongside the expected string to reduce repeated O(n) work.
- [3] (medium) jsmn/example/simple.c: The example allocates a fixed token array of 128 entries with a comment-based assumption about maximum token count. This is fragile for larger inputs and can cause parse failures when the assumption is violated. Consider a two-pass approach or dynamic token sizing based on jsmn_parse() return value.

## Manual Review Required

- [1] (high) jsmn/example/jsondump.c: The example uses a custom realloc wrapper that frees the original pointer on failure. This is risky API design because callers cannot safely continue using the old buffer after failure, which can easily lead to use-after-free bugs if the wrapper is reused incorrectly. Prefer a safer pattern where the caller explicitly handles realloc failure and retains ownership of the original pointer.
  - Reason: High-priority security-sensitive finding requires human review before code changes.
  - Next step: Confirm the intended security behavior, threat model, and tests before applying a targeted fix.
- [2] (low) jsmn/README.md: The README references outdated services and links (for example Travis CI and older project URLs). This is not a code defect, but stale documentation reduces trust and can mislead users about build health and maintenance status.
  - Reason: The target file type is not safe for automated inline patching in this workflow.
  - Next step: Review and update the file manually, then rerun analysis to confirm the finding is resolved.
- [3] (low) jsmn/test/tests.c: The test file relies heavily on dense macro-style assertions with many inline numeric expectations, which makes failures harder to diagnose and maintain. Breaking complex cases into named test helpers or data-driven fixtures would improve readability and reduce the risk of mistaken expected offsets.
  - Reason: The AI did not generate a meaningful source-file change for this issue.
  - Next step: Review the finding manually or rerun a focused fix pass with more context.
