# Spec 001: Auto-Save + Dedup + CWS Listing

> Sprint 1 | Priority: P0 | Est: 1 week
> Status: pending

## Goal

Ship the minimum viable "install and forget" experience: extension auto-saves conversations to Google Drive, deduplicates, and is publicly listed on Chrome Web Store.

## Quantified Sprint Metrics

| Metric | Target | How to measure |
|--------|--------|---------------|
| Auto-save triggers per page load | >= 1 (when conversation changes) | Console log count in test |
| Extraction success rate | >= 90% across 3 platforms | Test suite: 10 conversations per platform |
| Dedup accuracy | 100% (no duplicate files on Drive) | Save same conversation 3x, verify 1 file |
| Save latency (user-perceived) | < 3s from detection to Drive upload | Performance test with timer |
| CWS listing | Published + indexed | Manual verification |
| README conversion | Has demo GIF, badges, 3-step quick start | Checklist |

## Acceptance Criteria

### AC1: Auto-Save (MutationObserver)
- [ ] Content script injects MutationObserver on claude.ai, chatgpt.com, gemini.google.com
- [ ] Observer detects new messages (assistant response complete)
- [ ] Debounces: waits 30s after last mutation before saving
- [ ] Minimum interval: 5 min between saves of same conversation
- [ ] Saves to `crossmem/{platform}/{date}_{title}.md` on Google Drive
- [ ] Shows badge icon "✓" on extension icon after successful save (clears after 3s)
- [ ] Fails silently (no user-facing error) — logs to `chrome.storage.local` error array
- [ ] Works when tab is in background (not just active tab)
- [ ] Does NOT save empty conversations (< 2 messages)

### AC2: Deduplication
- [ ] SHA-256 hash of first 10KB of markdown content
- [ ] Hash stored in `.crossmem-meta.json` sidecar file alongside conversation
- [ ] Before upload: check if hash exists in sidecar → skip if match
- [ ] If content changed (hash differs): overwrite existing file (update, not create new)
- [ ] Test: save same conversation 3 times → verify exactly 1 file on Drive

### AC3: Metadata Sidecar
- [ ] Each conversation has a `.crossmem-meta.json` file on Drive:
  ```json
  {
    "conversations": {
      "conversation-id-or-title-slug": {
        "platform": "claude.ai",
        "title": "API Design Discussion",
        "hash": "a1b2c3d4...",
        "messageCount": 24,
        "firstSaved": "2026-02-20T12:00:00Z",
        "lastSaved": "2026-02-20T14:30:00Z",
        "driveFileId": "1abc..."
      }
    }
  }
  ```
- [ ] One sidecar per platform folder (not per conversation)
- [ ] Sidecar is read before each save to check dedup

### AC4: CWS Public Listing
- [ ] Icon: 128x128 PNG (clean, recognizable)
- [ ] Screenshots: 3x 1280x800 showing save flow on Claude, ChatGPT, Gemini
- [ ] Description: benefit-driven, <132 char title, keywords for ASO
- [ ] Category: Productivity
- [ ] Privacy policy URL: GitHub Pages link to PRIVACY.md
- [ ] Permission justifications: per EXTENSION-GUIDE.md checklist
- [ ] Submitted and listed (may take 1-3 days for review)

### AC5: README Rewrite
- [ ] Hero: demo GIF showing auto-save in action (save → search via MCP)
- [ ] Badges: CWS installs, GitHub stars, license
- [ ] "Why crossmem" section: 3 bullet points (convenience, security, cross-platform)
- [ ] Quick start: 3 steps (install → authorize Drive → done)
- [ ] Architecture diagram (extension → Drive → MCP)
- [ ] Comparison table vs Voyager, MemoryPlugin

## Out of Scope (this sprint)
- Tags/folders (Phase 1 P1, next sprint)
- AI summary (Phase 2)
- Prompt library (Phase 2)
- Context injection (Phase 2)
- Firefox/Safari (Icebox)
- Lemon Squeezy setup (Phase 1.5, after 500 users)

## Test Plan

```bash
# Unit tests (run locally, no network)
npx tsx test/verify-autosave.ts        # MutationObserver logic, debounce, dedup
npx tsx test/verify-extractors.ts      # All 3 platform extractors on sample HTML
npx tsx test/verify-dedup.ts           # Hash + sidecar read/write

# Integration tests (requires Drive auth)
npx tsx test/verify-save-flow.ts       # Full save → Drive → verify file exists

# Manual tests (in browser)
# 1. Load extension on Claude.ai, start a conversation, wait 30s → verify save
# 2. Repeat on ChatGPT and Gemini
# 3. Save same conversation twice → verify no duplicate on Drive
# 4. Check extension badge shows ✓ after save
# 5. Kill tab during save → verify no crash, no partial file
```

## Completion Signal

All acceptance criteria checked. Test plan passes. CWS submitted.

Output: `<promise>SPRINT1-DONE</promise>`
