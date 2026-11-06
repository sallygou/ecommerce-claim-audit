# Output schema guide

The Skill returns one JSON object per case.

## Claim-level fields

-   `claim_id`: stable identifier within the case.
-   `claim_text`: one independently reviewable statement.
-   `claim_type`: `FACT`, `OPINION`, `PUFFERY`, or `OFFER`.
-   `source_locations`: where the claim appeared.
-   `target_product`: the SPU/SKU/version being reviewed.
-   `evidence_ids`: evidence actually considered; do not list evidence
    that was not inspected.
-   `alignment`: object, time, scope, condition, numeric, and
    source-quality checks.
-   `finding`: `SUPPORTED`, `CONFLICT`, `INSUFFICIENT`, `CONTEXTUAL`, or
    `UNDETERMINED`.
-   `risk_level`: `LOW`, `MEDIUM`, `HIGH`, or `UNKNOWN`.
-   `action`: `PASS`, `BLOCK`, `REQUEST_EVIDENCE`, or `HUMAN_REVIEW`.
-   `reason`: concise, input-grounded explanation.
-   `missing_or_next_evidence`: actionable evidence request, or an empty
    array.

## Important distinctions

-   `CONFLICT` means supplied information disagrees; it does not by
    itself prove fraud.
-   `MISSING` means the required information was not supplied.
-   `UNKNOWN` means the available information is insufficient to decide.
-   A document's presence is not proof of authenticity, validity, or SKU
    coverage.
-   Page-level notes must not overwrite or collapse claim-level results.
