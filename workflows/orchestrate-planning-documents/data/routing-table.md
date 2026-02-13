# Document Status Routing Table

**Purpose:** Reference table for routing workflow based on document status.

| Status | Action | Next Step Variable |
|--------|--------|-------------------|
| `draft` or incomplete | Create/resume document | `{step03CreateDocument}` |
| `complete` | Adversarial review | `{step04AdversarialReview}` |
