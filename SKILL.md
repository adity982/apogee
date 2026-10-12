---
name: apogee
description: Navigate and change Apogee's private browser-page extraction and local summarization workflows. Use when a task concerns extracting pages, selections, videos, PDFs, DOCX files, or pasted text; summarizing that content; or testing either pipeline.
license: MIT
compatibility: Requires Node.js 24 or newer. Browser smoke tests require a Chromium or Firefox extension build.
---

# Apogee extraction and summarization

Work from `apogee-extension/`. Keep page content, prompts, summaries, and files on-device; do not add cloud inference or telemetry.

## Choose the entry point

- Browser pages: start at `lib/extract/pageExtraction.js`. It injects the content scripts and asks `content/content.js` for `extract-page-content`. Site-specific behavior belongs in `content/extractors/`; register a new extractor in both files.
- Selected text: use `lib/extract/selection.js`. Preserve the minimum-selection and user-facing error behavior.
- PDF and DOCX files: use `lib/extract/pdfExtract.js` and `lib/extract/docxExtract.js`. Keep size, expanded-text, archive, and eval-safety limits fail-closed.
- Summarization: the UI begins in `ui/app.js` at `summarizeActivePage()` or `summarizeCustomContent()`. Provider adapters live in `lib/engines/providers.js`; local generation and long-input map-reduce live in `lib/summarize/ollamaSummarize.js` and `lib/summarize/mapReduce.js`.
- Background or offscreen routing: keep the message contract aligned between `background/service-worker.js` and `offscreen/offscreen.js`. Never let a tab-originated message invoke privileged actions.

## Make the change

1. Trace the chosen entry point through extraction, provider routing, and storage before editing.
2. Reuse an existing extractor or provider pattern. Return `null` from a site extractor when the page is outside its scope so generic extraction can continue.
3. Preserve cancellation, bounded inputs, user-safe errors, and the privacy rules in `PRIVACY.md`.
4. Add the narrowest Node test. Extractors need a trimmed, anonymized fixture under `tests/extractors/fixtures/`; pure helpers should use the matching directory under `tests/`.

## Validate

Run from `apogee-extension/`:

```text
npm run format:check
npm run lint
npm test
npm run build
```

If behavior depends on extension APIs, also load `dist/chrome` or `dist/firefox` and exercise the exact summarize or extract path in a browser. Report that separately from the automated checks.
