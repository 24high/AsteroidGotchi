# Independent review of a completed change

Frontier prompt: `prompts/frontier-system.md`.

---

```json
{
  "change_under_review": "<short description + task id(s)>",
  "files": [{"path": "", "snippet_or_diff_ref": ""}],
  "stated_outcome": "<what the implementer claimed: result, checks run, evidence>",
  "acceptance_criteria": ["<the criteria the change was supposed to meet>"],
  "review_focus": ["correctness", "conventions", "resource_usage", "security", "hidden_coupling"],
  "output_contract": "Review block: verdict (accept / accept-with-notes / reject-on-criteria), unmet criteria with file references, risks with concrete mitigations, and any additional verification command run by reviewer with its real output."
}
```

Rules:
- Read the actual diff and the actual check output; do not judge from a summary.
- Demote anything implemented but merely believed to work ("unverified" until observed).
- Flag new abstractions with no consumer and any invented/unsupported API name.
- Where opened questions remain, list them as concrete follow-up `task_assign` items,
  not as vague worries.
