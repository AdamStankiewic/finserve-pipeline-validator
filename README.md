# FinServe AI Pipeline — Output Validator

A Python-based validation tool for the FinServe Lead-to-Credit-Memo AI pipeline. Built as part of the **10Clouds Financial Institutions — AI Intern/Specialist** recruitment task.

## Overview

This validator works alongside the **AIConsole-based pipeline** that processes lending leads through 5 stages:

```
Raw Lead → Parse & Extract → Score → Generate Proposal → Generate Credit Memo → Validate
```

The validator performs **10 automated checks** across security, data integrity, and business logic:

| # | Check | Category | What it does |
|---|-------|----------|--------------|
| 1 | Prompt Injection Detection | Security | Detects manipulation attempts in lead text and outputs |
| 2 | Compliance Status | Security | Ensures AI never marks compliance items as COMPLETED |
| 3 | NIP Format | Data Integrity | Validates Polish tax ID format + checksum |
| 4 | Amount Consistency | Data Integrity | Cross-checks amounts across all documents |
| 5 | Financial Data Integrity | Data Integrity | Verifies figures match original lead input |
| 6 | Completeness | Data Integrity | Checks required fields are populated |
| 7 | Scoring Logic | Data Integrity | Validates score breakdown, weights, priority mapping |
| 8 | Cross-Document Consistency | Data Integrity | Checks client details match across documents |
| 9 | Deal Size Fit | Business Logic | Validates amount within 1M-15M PLN target range |
| 10 | Amount-to-Revenue Ratio | Business Logic | Flags disproportionate loan requests |

## Quick Start

```bash
# Run built-in demo with 3 test scenarios
python validator.py --demo

# Validate your own pipeline output
python validator.py pipeline_output.json

# Run unit tests
python -m pytest tests/ -v
```

## Demo Output

```bash
$ python validator.py --demo

--- Test 1: Valid Lead (BudDom Invest) ---
Overall: PASS_WITH_WARNINGS | Recommendation: READY_FOR_REVIEW

--- Test 2: Prompt Injection Attempt ---
Overall: FAIL | Recommendation: SECURITY_REVIEW_REQUIRED

--- Test 3: Anomalous Data (50M request, 2M revenue) ---
Overall: FAIL | Recommendation: NEEDS_CORRECTION
```

## Input Format

The validator expects a JSON file with 5 fields matching the pipeline stages:

```json
{
  "raw_lead": "Original lead text...",
  "parsed": { "company_name": "...", "requested_amount_pln": 12000000, ... },
  "scoring": { "overall_score": 87, "priority": "HIGH", ... },
  "proposal": { "client_info": { ... }, "proposal_summary": { ... } },
  "memo": { "credit_memo": { "compliance_checklist": { ... }, ... } }
}
```

See `sample_data/` for complete examples.

## Key Design Decisions

- **Security-first**: Prompt injection and compliance checks run before any other validation
- **Graceful degradation**: Missing data produces warnings, not crashes
- **No external dependencies**: Pure Python 3.10+ — no pip install needed
- **NIP checksum**: Full Polish tax ID validation including the modulo-11 checksum algorithm
- **Amount parsing**: Handles Polish formats ("12 mln PLN", "500 tys PLN", "5,000,000")

## Architecture

```
validator.py          — Main script with all validation logic + CLI
tests/
  test_validator.py   — Unit tests for each validation function
sample_data/
  valid_lead.json     — Example: complete, valid pipeline output
  injection.json      — Example: prompt injection attempt
  anomalous.json      — Example: suspicious financial data
```

## Testing Approach

This validator was developed as part of a QA-driven approach to the AI pipeline:

1. **Built** the pipeline in AIConsole (5-step workflow)
2. **Tested** with 8 scenarios (happy path, prompt injection, edge cases, AML)
3. **Found** critical security vulnerability (prompt injection bypassed all checks)
4. **Fixed** with guardrails in system prompts
5. **Verified** fix with regression testing
6. **Built this validator** as a standalone safety net

## Author

**Adam Stankiewicz** — QA Engineer with experience in E2E automation (Playwright/TypeScript), API testing, and AI pipeline development.

## License

MIT
