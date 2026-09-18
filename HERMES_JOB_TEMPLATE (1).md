# Hermes Job Template

## VERIFIED INPUTS / READ-ONLY DISCOVERY
<verified runtime facts or exact discovery instructions>

## GOAL
<one bounded infrastructure/runtime outcome>

## IN SCOPE
- ...

## OUT OF SCOPE
- ...

## MUTATION RIGHTS
<read-only by default; exact authorized paths/services if writes are allowed>

## SECRETS POLICY
Do not echo secret values. Return only sanitized status/IDs/fingerprints where safe.

## REQUIRED TESTS / ERROR PATHS
- ...

## ROLLBACK / STOP CONDITIONS
- ...

## EVIDENCE TO RETURN
- commands/actions and exit states;
- before/after identifiers;
- service/container/build IDs;
- checksums where relevant;
- test results;
- sanitized logs/excerpts;
- no success claim without evidence.
