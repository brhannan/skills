# JSON Output Schema

When the `--json` flag is used, the final report is output as structured JSON to stdout. This schema defines the structure.

## Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ExpertReviewReport",
  "type": "object",
  "required": ["metadata", "summary", "findings", "domain_summaries", "verdict"],
  "properties": {
    "metadata": {
      "type": "object",
      "required": ["date", "profile", "reviewers_deployed", "target_type"],
      "properties": {
        "date": {
          "type": "string",
          "format": "date-time",
          "description": "ISO 8601 timestamp of the review"
        },
        "profile": {
          "type": "string",
          "enum": ["full", "lite", "minimal"],
          "description": "Review profile used"
        },
        "reviewers_deployed": {
          "type": "integer",
          "description": "Number of reviewer agents deployed"
        },
        "target_type": {
          "type": "string",
          "enum": ["pr", "file", "staged", "uncommitted"],
          "description": "What was reviewed"
        },
        "target_value": {
          "type": "string",
          "description": "PR URL, file path, or empty string"
        },
        "pr_number": {
          "type": "string",
          "description": "PR number or 'local'"
        },
        "pr_title": {
          "type": "string",
          "description": "PR title if applicable"
        },
        "reviewer_model": {
          "type": "string",
          "description": "Model used for reviewer agents"
        }
      }
    },
    "summary": {
      "type": "string",
      "description": "2-3 sentence executive summary"
    },
    "findings": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id", "severity", "confidence", "domain", "title", "location", "description"],
        "properties": {
          "id": {
            "type": "string",
            "description": "Unique finding identifier (e.g., 'CORR-001', 'SEC-003')"
          },
          "severity": {
            "type": "string",
            "enum": ["critical", "high", "medium", "low"],
            "description": "Severity level"
          },
          "confidence": {
            "type": "string",
            "enum": ["high", "medium", "single-reviewer"],
            "description": "Consensus-based confidence level"
          },
          "domain": {
            "type": "string",
            "enum": ["correctness", "quality", "architecture", "performance", "testing", "security"],
            "description": "Review domain"
          },
          "title": {
            "type": "string",
            "description": "Short finding title"
          },
          "location": {
            "type": "object",
            "properties": {
              "file": {
                "type": "string",
                "description": "File path"
              },
              "line": {
                "type": "integer",
                "description": "Line number"
              }
            }
          },
          "description": {
            "type": "string",
            "description": "Detailed description of the finding"
          },
          "flagged_by": {
            "type": "array",
            "items": { "type": "string" },
            "description": "List of reviewer agents that flagged this issue"
          },
          "suggestion": {
            "type": "string",
            "description": "Suggested fix or improvement"
          },
          "cwe": {
            "type": "string",
            "description": "CWE ID for security findings"
          }
        }
      }
    },
    "domain_summaries": {
      "type": "object",
      "properties": {
        "correctness": {
          "$ref": "#/definitions/domain_summary"
        },
        "quality": {
          "$ref": "#/definitions/domain_summary"
        },
        "architecture": {
          "$ref": "#/definitions/domain_summary"
        },
        "performance": {
          "$ref": "#/definitions/domain_summary"
        },
        "testing": {
          "$ref": "#/definitions/domain_summary"
        },
        "security": {
          "$ref": "#/definitions/domain_summary"
        }
      }
    },
    "verdict": {
      "type": "object",
      "required": ["decision", "critical_count", "high_count", "medium_count", "low_count"],
      "properties": {
        "decision": {
          "type": "string",
          "enum": ["APPROVE", "REQUEST_CHANGES", "COMMENT"],
          "description": "Overall review decision"
        },
        "critical_count": {
          "type": "integer"
        },
        "high_count": {
          "type": "integer"
        },
        "medium_count": {
          "type": "integer"
        },
        "low_count": {
          "type": "integer"
        },
        "rationale": {
          "type": "string",
          "description": "Brief rationale for the verdict"
        }
      }
    }
  },
  "definitions": {
    "domain_summary": {
      "type": "object",
      "properties": {
        "included": {
          "type": "boolean",
          "description": "Whether this domain was included in the review profile"
        },
        "summary": {
          "type": "string",
          "description": "Domain-level summary"
        },
        "verdict": {
          "type": "string",
          "description": "Domain-level verdict"
        },
        "finding_count": {
          "type": "integer",
          "description": "Number of findings in this domain"
        },
        "reviewers": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Reviewers that contributed to this domain"
        }
      }
    }
  }
}
```

## Verdict Decision Rules

- **APPROVE**: No critical findings, at most 2 high findings
- **REQUEST_CHANGES**: Any critical findings, or 3+ high findings
- **COMMENT**: No critical findings, 1-2 high findings (borderline — author's judgment)

## Finding ID Prefixes

| Domain | Prefix |
|--------|--------|
| Correctness | CORR |
| Quality | QUAL |
| Architecture | ARCH |
| Performance | PERF |
| Testing | TEST |
| Security | SEC |

## Example

```json
{
  "metadata": {
    "date": "2026-02-23T14:30:00Z",
    "profile": "full",
    "reviewers_deployed": 12,
    "target_type": "pr",
    "target_value": "https://github.com/myorg/myapp/pull/42",
    "pr_number": "42",
    "pr_title": "Add user authentication",
    "reviewer_model": "sonnet"
  },
  "summary": "The authentication implementation has 1 critical SQL injection vulnerability and several moderate code quality concerns. The overall architecture is sound but the auth module needs security hardening before merge.",
  "findings": [
    {
      "id": "SEC-001",
      "severity": "critical",
      "confidence": "single-reviewer",
      "domain": "security",
      "title": "SQL Injection in user lookup",
      "location": { "file": "src/auth/users.py", "line": 45 },
      "description": "User input is directly interpolated into SQL query without parameterization.",
      "flagged_by": ["security"],
      "suggestion": "Use parameterized queries: db.execute('SELECT * FROM users WHERE name = %s', (username,))",
      "cwe": "CWE-89"
    }
  ],
  "domain_summaries": {
    "security": {
      "included": true,
      "summary": "Critical SQL injection vulnerability found in auth module.",
      "verdict": "VULNERABLE",
      "finding_count": 1,
      "reviewers": ["security"]
    }
  },
  "verdict": {
    "decision": "REQUEST_CHANGES",
    "critical_count": 1,
    "high_count": 0,
    "medium_count": 3,
    "low_count": 2,
    "rationale": "Critical SQL injection vulnerability must be fixed before merge."
  }
}
```
