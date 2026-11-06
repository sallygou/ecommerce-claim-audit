---
name: ecommerce-claim-audit
display_name: 电商商品主张证据审核
display_name_en: Ecommerce Claim & Evidence Audit
description: Review ecommerce product claims against supplied SKU attributes, evidence, and rules; identify mismatches and recommend claim-level actions. Trigger when a user submits a product listing, SKU data, attributes, certificates, test reports, or an audit case.
description_zh: 逐条提取商品宣传主张，匹配 SKU 属性与证明材料，识别信息断裂并给出审核动作建议。
description_en: Extract product claims, align them with SKU attributes and evidence, identify gaps, and recommend review actions.
version: 1.0.0
author: Personal learning project
examples_zh:
  - "审核这个商品详情页，逐条提取主张并检查证据是否支持。"
  - "检查这份检测报告是否适用于当前 SKU。"
  - "找出商品标题、详情和结构化属性之间的信息冲突。"
examples_en:
  - "Audit this product page claim by claim and check supporting evidence."
  - "Check whether this test report applies to the current SKU."
  - "Find mismatches between product copy and structured attributes."
---
# Ecommerce Claim & Evidence Audit

## 1. Purpose

Review the relationship between what a product page claims and what the
supplied product data and evidence actually support.

The unit of review is an **individual claim**, not the entire product
page. One page may contain several claims with different outcomes.

This first version is a text-only MVP. It does not perform OCR, browse
official databases, verify certificates online, or make a final legal
determination.

## 2. Inputs

Accept either structured JSON or clearly labelled natural-language
product information.

Use these fields when available:

-   `case_id`
-   `category`
-   `product.spu_id`
-   `skus[]`: `sku_id`, variant, attributes, product/version identifiers
-   `page.title`
-   `page.main_image_ocr` (optional text already extracted from images)
-   `page.detail_text`
-   `page.snapshot_time`
-   `evidence[]`: evidence ID, type, source, product/SKU scope,
    conclusion, validity dates, verification status
-   `rules[]`: applicable platform rule, version, and required action

If a field is missing, record it as missing. Do not invent it. If the
case is provided as prose, normalize it into these fields and state any
assumptions.

## 3. Five-step workflow

### Step 1 --- Parse and locate page information

1.  Separate title, image OCR text, detail text, structured attributes,
    SKU data, and evidence.
2.  Preserve each item's source location, such as `title`,
    `detail_text`, `main_image_ocr`, or
    `skus[0].attributes.filling_material`.
3.  Preserve the SPU, SKU, product version, and snapshot time whenever
    supplied.
4.  Do not merge information from different SKUs or versions without an
    explicit relationship.

### Step 2 --- Extract and classify atomic claims

1.  Extract every materially distinct product statement that may need
    review.
2.  Split a sentence containing multiple independently checkable
    statements into separate claims.
3.  Merge duplicate wording only when it expresses the same claim;
    retain all source locations.
4.  Classify each claim as one of:
    -   `FACT`: objectively checkable product or performance statement.
    -   `OPINION`: subjective opinion or personal evaluation.
    -   `PUFFERY`: expressive or promotional wording without a clear
        objective measurement, subject to context.
    -   `OFFER`: price, discount, promotion, or transaction promise.
5.  Do not assume that a phrase is harmless merely because it sounds
    promotional. If it implies a concrete measurable promise, extract
    that promise as a separate `FACT` or `OFFER` claim.

### Step 3 --- Route each claim to relevant evidence

For each claim, identify what evidence would be relevant. Examples:

-   Material/composition claim → SKU attributes, material declaration,
    applicable test report.
-   Capacity/performance claim → matching product specification or test
    report.
-   Certification claim → certificate details and, where available, an
    authoritative verification result.
-   Promotion claim → campaign rules, eligible products/users, discount
    calculation, and validity period.
-   Subjective wording → usually contextual review rather than a
    fabricated evidence request.

Record evidence IDs and why each item is or is not applicable.

### Step 4 --- Align claim and evidence

Check the relevant dimensions:

-   `object_match`: same product, model, SPU/SKU, formula, or version?
-   `time_match`: valid for the relevant sale period and current product
    version?
-   `scope_match`: does the evidence cover this SKU, batch, region,
    sample, and population?
-   `condition_match`: are test conditions and limitations preserved?
-   `numeric_match`: do values, units, ranges, and statistical bases
    agree?
-   `source_quality`: is the source identified and independently
    verified, or merely merchant-provided?

Use these alignment statuses:

-   `MATCH`
-   `CONFLICT`
-   `MISSING`
-   `NOT_APPLICABLE`
-   `UNKNOWN`

A correctly formatted certificate or report number is not proof that the
document is genuine or still valid. A merchant-uploaded file is not
independently verified unless the input explicitly supplies a
verification result.

### Step 5 --- Decide an action for each claim

Choose exactly one action for each claim:

-   `PASS`: Within the supplied data and applicable rules, the claim is
    adequately supported or does not require factual substantiation in
    the reviewed context.
-   `BLOCK`: A material conflict is established by sufficiently
    reliable, applicable information and the supplied rule supports
    blocking. Describe this as a platform action recommendation, not a
    legal verdict.
-   `REQUEST_EVIDENCE`: The claim may be reviewable, but necessary
    evidence is missing or does not cover the current product/SKU.
-   `HUMAN_REVIEW`: The meaning, evidence, rule, or conflict is
    ambiguous; sources disagree; or a high-impact decision cannot safely
    be automated.

Decision principles:

1.  Never infer missing facts.
2.  Never treat a report for SKU-B as proof for SKU-A unless the
    evidence explicitly establishes that its scope covers both.
3.  Do not block a subjective phrase solely because it is expressive.
4.  Do not pass a factual claim merely because no banned keyword was
    found.
5.  If a claim is objectively contradicted by a verified platform
    attribute and an applicable rule says to block, recommend `BLOCK`.
6.  If the only problem is missing evidence, prefer `REQUEST_EVIDENCE`
    unless the supplied rule specifies otherwise.
7.  If the source or policy is unclear, use `HUMAN_REVIEW`.
8.  Different claims on the same page may and often should receive
    different actions.

## 4. Required output

Return valid JSON only. Do not wrap it in Markdown code fences.

Use this structure:

{ "case_id": "string or null", "scope_note": "What information was and
was not reviewed", "claims": \[ { "claim_id": "C001", "claim_text":
"atomic claim", "claim_type": "FACT \| OPINION \| PUFFERY \| OFFER",
"source_locations": \["title"\], "target_product": { "spu_id": "string
or null", "sku_id": "string or null", "version": "string or null" },
"evidence_ids": \["E001"\], "alignment": { "object_match": "MATCH \|
CONFLICT \| MISSING \| NOT_APPLICABLE \| UNKNOWN", "time_match": "MATCH
\| CONFLICT \| MISSING \| NOT_APPLICABLE \| UNKNOWN", "scope_match":
"MATCH \| CONFLICT \| MISSING \| NOT_APPLICABLE \| UNKNOWN",
"condition_match": "MATCH \| CONFLICT \| MISSING \| NOT_APPLICABLE \|
UNKNOWN", "numeric_match": "MATCH \| CONFLICT \| MISSING \|
NOT_APPLICABLE \| UNKNOWN", "source_quality": "VERIFIED \|
PROVIDED_NOT_VERIFIED \| UNKNOWN \| NOT_APPLICABLE" }, "finding":
"SUPPORTED \| CONFLICT \| INSUFFICIENT \| CONTEXTUAL \| UNDETERMINED",
"risk_level": "LOW \| MEDIUM \| HIGH \| UNKNOWN", "action": "PASS \|
BLOCK \| REQUEST_EVIDENCE \| HUMAN_REVIEW", "reason": "Short explanation
grounded in the supplied inputs", "missing_or_next_evidence":
\["Specific evidence needed, or an empty array"\] } \],
"page_level_note": "Summarize mixed claim-level outcomes without erasing
them", "limitations": \["What could not be verified from the supplied
inputs"\] }

If there are no claims, return an empty `claims` array and explain why
in `page_level_note`.

## 5. Quality checks before responding

-   Every factual statement in the output must be traceable to an input
    field or supplied rule.
-   Every claim must retain its source location.
-   Every evidence item must be checked against the target
    SKU/version/scope.
-   Missing evidence must not be described as proof that a claim is
    false.
-   A format check must not be described as authenticity verification.
-   Do not invent legal citations, official database results, test
    results, or confidence scores.
-   Keep the claim-level decisions visible even when the page contains
    mixed outcomes.
-   Ensure the final response is valid JSON.

## 6. Out of scope for this MVP

-   Image OCR or visual understanding.
-   Live certificate or report lookup.
-   External legal research.
-   Automatic legal liability determination.
-   Automatic irreversible penalties.
-   Production integration, queues, retries, and human-review UI.

These can be added only after the text-only workflow is tested.
