Ecommerce Claim & Evidence Audit — starter package

What this is
- A text-only MVP Skill for the course assignment.
- It follows the five stages: parse page -> extract claims -> route evidence -> align evidence -> recommend an action.
- It reviews claims one by one, so a single page can contain different outcomes.

Files
- SKILL.md: the Skill instructions.
- examples/case_001.json: a sample product case.
- references/output-schema.md: output field guide.

How to try it
1. Put this folder into the Skill location supported by your AI coding/agent environment, or paste SKILL.md into that environment's Skill editor.
2. Provide examples/case_001.json as the input case.
3. Ask: "Use the ecommerce-claim-audit Skill to review this case. Follow SKILL.md and return valid JSON."
4. Check that the output separates the three kinds of claims:
   - "100%桑蚕丝填充" — factual material claim; the platform attribute says polyester, while the uploaded report is for SKU-002, not SKU-001.
   - "柔软如云" — likely puffery in this context; do not block solely because it is expressive.
   - "限时立减50元" — offer claim; campaign rules and validity evidence are missing.

Platform note
The package is platform-neutral. The exact installation path and invocation method depend on the Skill-capable environment you use. This package does not connect to external databases or perform OCR.

MVP test checklist
- Does it extract separate claims from one product page?
- Does it preserve each claim's source location?
- Does it refuse to apply SKU-002's report to SKU-001 without scope proof?
- Does it distinguish a material conflict from missing promotional evidence?
- Does it avoid claiming that a merchant-uploaded report is authentic?
- Does it return valid JSON with one action per claim?
