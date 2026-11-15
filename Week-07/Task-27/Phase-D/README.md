Week 07 – Phase D: Independent Extension — Trust Policy Hardening

Objective
Apply security controls to prevent the AssumeRole chaining abuse demonstrated in Phase C.

Action Taken
Added `sts: ExternalId` and `aws:SourceIp` conditions to `RoleB`'s trust policy, saved as `PhaseD_Hardened_Policy.json`, and applied it with `aws iam update-assume-role-policy`. Re-established an `ExampleRole` session exactly as in Phase C, then re-ran the identical chained `assume-role` call against `RoleB` without supplying an `ExternalId`. Repeated the call with the correct `--external-id` from the approved source IP and confirmed the resulting identity with `aws sts get-caller-identity`.

Before / After Comparison

| | Before (Phase C) | After (Phase D) |
|---|---|---|
| Trust policy principal | `role/ExampleRole` | `role/ExampleRole` (unchanged) |
| Conditions | None | `sts:ExternalId`, `aws:SourceIp` |
| Chained assumption, no ExternalId | Succeeded (`chainStep2`) | **AccessDenied** (`chainStep2retest`) |
| Chained assumption, correct ExternalId, correct IP | n/a | Succeeded (`chainStep2hardened`) |
| Attack surface | Any session of `ExampleRole`, from anywhere, could pivot to `RoleB` | Only a session that also knows the ExternalId and originates from the approved IP can pivot |

Key Observation
- The unmodified Phase C attack now fails: AWS returned `AccessDenied`, confirming the Condition block is enforced, not just present in the policy document.
- Supplying the correct `ExternalId` from the whitelisted IP restored access and produced a valid session, confirming the conditions were configured correctly rather than accidentally blocking all access.
- The vulnerability was never in the `Action` or the general `Principal` structure — it was the absence of any condition narrowing who, from where, and with what additional secret could invoke that trust.
- A real attacker holding only stolen `ExampleRole` credentials, without also knowing the ExternalId and operating from the approved network, can no longer complete the chain.

AI Review of the Hardened Policy
- ✅ `ExternalId` closes the confused-deputy style weakness exploited in Phase C.
- ✅ `SourceIp` restricts the network origin to a single verified address.
- ✅ `StringEquals` (exact match) is correctly used rather than the looser `StringLike`.

Evidence
PhaseD_Hardened_Policy.json` — the hardened trust policy with both conditions.

Checklist
- [x] Hardened trust policy implemented
- [x] Verified restricted access behavior (denied without conditions, allowed with them)
- [x] AI-reviewed final policy JSON
