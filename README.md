# lala-labs — content

Content updates for the **Coolhunting** track of the Lala-Labs iOS app.

The app ships with Issue 01 inside the binary and works completely offline. When it has a
connection it reads `manifest.json` from this repository, and downloads a new issue only when
the manifest's `revision` is higher than the one installed. A downloaded issue is cached on
the device and used offline from then on.

Nothing in this repository can affect a reader's progress, XP, streak or saved archive —
those live in the app's own storage and are never written by the update process.

## Layout

```
manifest.json          what the app reads first — always at the repository root
issues/
  issue-01.json        a complete issue
  issue-02.json        the next one, and so on
.github/workflows/
  validate.yml         checks every push: valid JSON, manifest points at a real file,
                       revisions agree, required fields present
```

## Publishing a new issue

1. Add the new file, e.g. `issues/issue-02.json`.
2. Set `"revision"` inside that file to the next integer.
3. Edit `manifest.json` so `revision`, `issue`, `title`, `date` and `path` match it.
4. Commit to `main`.

The app picks it up on its next check — at launch, when the Coolhunting tab opens, or
immediately if the reader taps **Check**. Checks are throttled to once every six hours
unless the reader asks.

**`revision` is the only thing that triggers a download.** It must increase every time,
and never go backwards. The issue number is for display; revision is the mechanism.
If you correct a typo in a published issue, bump the revision so readers actually receive it.

Keep old issue files in place. Readers who saved an article from Issue 01 keep their copy
inside the app, but leaving the files here costs nothing and keeps history readable.

## Rules the app enforces

- A downloaded issue is decoded and checked before it replaces anything. If it is malformed,
  incomplete, or missing an `id` or title on any item, the download is discarded and the
  current issue stays.
- `schema` in the manifest is currently `1`. If it is ever raised, older app versions stop
  applying updates instead of guessing — so only raise it alongside an App Store release.
- Downloads are capped at 8 MB and time out after 20 seconds.
- The reader can revert to the bundled issue at any time from Profile → Content.

## Issue file format

An issue file is the complete Coolhunting track. The quickest way to make Issue 02 is to copy
`issues/issue-01.json`, bump `revision`, edit `issue`, and replace the `items` array.

```jsonc
{
  "revision": 2,                       // must match manifest.json
  "id": "cool",
  "title": "Coolhunting",
  "sub": "Issue 02 — <theme>",
  "lang": "en",
  "accent": "#BF5AF2",                 // keep, unless you want a new colour per issue
  "accent2": "#FF9F0A",
  "nav": ["ISSUE", "SAVED", "TERMS", "PROFILE"],
  "ui": { /* copy from issue-01.json unchanged */ },
  "issue": {
    "n": "02",
    "t": "<theme>",
    "date": "September 2026",
    "dek": "<one paragraph on what this issue argues>",
    "note": "<editorial note, shown under the cover>"
  },
  "items": [ /* the pieces — see below */ ],
  "terms": [ { "term": "…", "def": "…", "ex": "…" } ]
}
```

### An item

`kind` decides which section it lands in: `news` → Dispatch, `case` → Case studies,
`trend` → Trends, and anything else (`guide`, `palette`, `timeline`, `calendar`, `refs`)
→ Reference.

```jsonc
{
  "id": "c-example",                   // unique and stable; it is how "read" and "saved" are tracked
  "kind": "case",
  "plate": "islands",                  // illustration name, see the list below
  "kicker": "Case study · 1972",
  "t": "Headline",
  "dek": "One sentence under the headline.",
  "short": ["Three lines", "that give the", "whole argument"],
  "body": "<p>Paragraphs. <b>Bold</b> and <em>accent</em> are supported.</p>",
  "facts":  [{ "l": "Architect", "v": "Name" }],
  "steal":  ["What to steal, one line each"],
  "spot":   ["How to spot it"],         // trends
  "fake":   ["Counterfeit tells"],      // trends
  "stage":  "Rising",                   // trends: Rising | Peak | Late
  "life":   "3–5 years of runway",
  "links":  [{ "t": "Source name", "u": "https://…" }]
}
```

Optional item shapes, each rendered natively:

- `"steps": [{ "t": "Step name", "d": "What to do" }]` — numbered field guide
- `"swatches": [{ "hex": "#B7442C", "name": "Iron oxide", "note": "What it does" }]` — palette board
- `"rows": [{ "y": "1972", "t": "Event", "d": "Why it matters" }]` — timeline
- `"events": [{ "date": "27–30 Oct 2026", "title": "Orgatec", "place": "Cologne", "why": "…", "confirmed": true }]` — calendar

Fields you leave out are simply not drawn. Unknown fields are ignored, so the repository can
run ahead of the app without breaking older installs.

### Illustration names for `plate`

`mushroom` `islands` `skin` `action` `petal` `rooms` `palette` `timber` `light` `curve`
`method` `prize` `wave` `crinkle` `reuse` `timeline` `cal` `refs`

An unrecognised name falls back to a neutral illustration rather than failing.
New illustrations require an app release.
