# Spec 006: Auto-Clustering (Phase 2)

> Not in MVP. This spec documents the future direction for when manual topic selection data is sufficient.

## Problem

MVP requires user to manually select a topic on every share. This is fine for 5-10 shares/day, but becomes friction at scale. Users want to just share and forget.

## Prerequisites

- 100+ sources with manual topic assignments (training data)
- Embedding pipeline working (from point cloud feature)
- Topic suggestion accuracy > 80% (measure during MVP with user corrections)

## Approach

### On-Share Flow (Phase 2)

```
User shares content
  → Extract title + first 500 tokens
  → Generate embedding (Claude API or on-device)
  → Compare cosine similarity against existing topic centroids
  → If max_similarity > 0.85 → auto-assign to that topic (show toast)
  → If 0.6 < max_similarity < 0.85 → suggest top 3 topics (current MVP behavior)
  → If max_similarity < 0.6 → suggest "New topic" with AI-generated name
```

### Topic Centroids

Each topic has a centroid embedding = weighted average of all source embeddings.

```json
// _topic.json addition
{
  "embedding": [0.12, -0.34, ...],   // centroid (avg of all sources)
  "sourceCount": 7,
  "lastRecalculated": "2026-03-15T10:00:00Z"
}
```

Recalculate centroid when:
- New source added
- Source moved to different topic
- Every 50 sources (drift correction)

### Merge Suggestions

When two topic centroids are within cosine distance < 0.15:
- Suggest merge in topic feed ("These topics seem related, merge?")
- User confirms or dismisses
- On merge: move all sources to target topic, archive source topic

### Split Suggestions

When a topic's intra-cluster variance exceeds threshold:
- Suggest split ("This topic covers multiple themes, split?")
- Show proposed sub-clusters
- User confirms or modifies

## Evaluation

Track during MVP:
- How often does user accept the first suggested topic? (measures suggestion quality)
- How often does user create a new topic? (measures coverage)
- How often does user move a source to a different topic after saving? (measures error rate)

Target for enabling auto-clustering: >80% first-suggestion acceptance rate.
