# Story Status Routing Table

**Purpose:** Reference table for routing workflow based on story status.

| Status | Action | Next Step Variable |
|--------|--------|-------------------|
| `backlog` | Create story file | `{step03CreateStory}` |
| `drafted` | Adversarial story review | `{step04AdversarialStoryReview}` |
| `ready-for-dev` | Develop story | `{step06DevelopStory}` |
| `in-progress` | Resume development | `{step06DevelopStory}` |
| `review` | Code review | `{step08ReviewStory}` |
| `done` | Show summary only | `{step11Summary}` |
