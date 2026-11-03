# ChatSearch

Search your indexed Claude and ChatGPT history from a Chrome extension. Find words as you type, allow for typos, or search by meaning. The search index is stored in your browser. See [Privacy](#privacy) for network access and preview limitations.

This is my fork of [Aaron Bergman's ChatSearch](https://github.com/AaronBergman/chatsearch). I removed the optional cloud mode for search by meaning and added a GitHub build. The original extension is Aaron's work.

![ChatSearch popup searching for "sourdough"](assets/screens/popup-search.png)

**[Download the latest release](https://github.com/shaunmarsden/chatsearch/releases/latest/download/chatsearch.zip)** | [Original project](https://github.com/AaronBergman/chatsearch)

## Why I forked it

This extension reads chat history from accounts you are signed in to. Those chats may hold client or prospect details, so I wanted search by meaning to run locally, without an optional cloud setting.

I removed that setting. The current code runs the embedding model on your device. That is not a guarantee that the extension makes no other network requests: it fetches chats, downloads model files and can load external images in chat previews. See [Privacy](#privacy) before using it with sensitive information.

The [GitHub build](.github/workflows/build.yml) runs the typecheck and unit tests and builds a download from the tagged source. This links the build to its source, but is not an independent security audit or a recorded network-traffic audit.

The extension still reads Claude and ChatGPT through their private web routes. If either site changes, syncing can break until the extension is updated. Importing an official data export is another way to populate the index.

## What it does

- **Full-text search** across indexed Claude and ChatGPT content: titles, your messages, replies and supported artifact, canvas and pasted-attachment text. Results update as you type, with the matching passage highlighted. Content must be captured or imported before it can be searched.
- **Substring matching**, so `plane` finds "airplanes" and `kube` finds "Kubernetes".
- **Typo tolerance.** Fuzzy title matching (`flt wk` finds "Fleet Week") and "did you mean" corrections (`risoto` searches for "risotto").
- **Search by meaning.** Press Shift+Enter or the Meaning button to find chats about an idea when you don't remember the words. Runs a small embedding model on your own machine by default.
- **Jump to the match.** Opening a result scrolls the chat to the passage you searched for and highlights every match, with a small navigator to step through them.
- **Background sync.** It uses your existing login to fetch history and check for updates. It also captures supported content loaded on chat pages you visit. Rate limits, site changes and incomplete downloads can leave gaps.
- **Preview pane** for indexed messages rendered as Markdown, copy as Markdown, download as `.md`. Long messages and conversations are shortened in the preview.
- **Filters and operators**: source, your messages only, time range, sort by relevance or date.
- **Export indexed chats** as a zip of Markdown files, and **import** official Claude and ChatGPT data exports.
- **Search history**, a persistent popup that remembers your last search, an optional side panel, a full-page view, an address-bar keyword (`cs`), and a right-click "Search my chats for ..." on selected text.

## Install

ChatSearch isn't on the Chrome Web Store yet, so it installs as an unpacked extension:

1. Download [`chatsearch.zip`](https://github.com/shaunmarsden/chatsearch/releases/latest/download/chatsearch.zip) and unzip it.
2. Open `chrome://extensions` and turn on **Developer mode** (top right).
3. Click **Load unpacked** and choose the unzipped `chatsearch` folder.
4. Pin it: click the puzzle-piece icon in the toolbar, then the pin next to ChatSearch.
5. Make sure you're signed in to claude.ai and/or chatgpt.com in this browser. Indexing starts on its own.

Open it with the toolbar icon or **Cmd+Shift+K** (Ctrl+Shift+K on Windows and Linux). The extension requires Chrome 120 or newer. Compatibility with other Chromium browsers is not verified by the repository's tests.

To update, download the new zip, replace the folder's contents, and click the reload icon on the extension's card in `chrome://extensions`. Your index is kept.

## Using it

Type to search. The list updates as you type; the preview on the right shows the selected chat with every match highlighted.

| Key | Action |
| --- | --- |
| Up / Down | Select a result |
| Enter | Open the chat at the match |
| Cmd/Ctrl+Enter | Open in a new tab |
| Shift+Enter | Search by meaning |
| Esc | Clear the search, then close |
| / or Cmd/Ctrl+K | Focus the search box |

Search operators:

| Operator | Meaning |
| --- | --- |
| `"exact phrase"` | Words next to each other |
| `-word` | Exclude chats containing the word |
| `from:me` / `from:ai` | Only match your messages / only the replies |
| `in:title` | Only match titles |
| `claude:` / `chatgpt:` | One source |
| `is:starred` | Starred chats |
| `after:2026-01` `before:2025` `after:30d` | Date range (years, months, days, or relative `d` `w` `m` `y`) |

In the address bar, type `cs`, a space, and your search to get results right in the dropdown.

## How it stays current

- **Claude**: the extension lists and downloads history on first run, then checks for updates roughly every minute or two. Completion time depends on the history and successful requests.
- **ChatGPT**: ChatSearch fetches titles before downloading full conversations. Titles become searchable as they are indexed. It spaces out full-text downloads, pauses them while you use ChatGPT, and backs off after rate-limit responses. To avoid waiting for those downloads, import an official data export through Settings, Your data, Import. The extension reads the supplied export locally; it can only index what that export contains.
- If a site returns a "checking your browser" page instead of data, ChatSearch reports it. Opening that site in a tab can help it resume.
- **Live capture**: ChatSearch can index supported content already loaded when you open or reply to a chat, without making another request for that content. This does not guarantee a complete or current copy of every chat.
- Sync checks can pick up deleted and renamed chats. Updates depend on the next successful check.

## Search by meaning

Turned on by default, and you can switch it off in Settings. It uses [MongoDB/mdbr-leaf-ir](https://huggingface.co/MongoDB/mdbr-leaf-ir) through transformers.js to turn text into numerical representations for comparison. It runs on the GPU via WebGPU when available, otherwise on the CPU. Model files are downloaded from Hugging Face and cached; download size depends on the selected format. Processing happens in the background, newest chats first, with less work scheduled on battery. Cached results are reused when the text hash matches.

Search by meaning uses the first 1,000 characters of a query. This limit does not apply to keyword search. A meaning search runs when you ask for it, or automatically once when an exact search finds nothing, rather than on every keystroke.

## Privacy

- Your chats, the search index and the vectors live in this browser's IndexedDB. Uninstalling the extension deletes them; Settings also has a delete button.
- Sync requests go to claude.ai and chatgpt.com using your existing login. Local search by meaning downloads model files from Hugging Face and its download infrastructure. There is no cloud-embedding option in this fork.
- **External images are not blocked in previews.** Markdown image URLs can load from other sites. Viewing a preview may therefore contact those sites; do not treat this as a network-isolated viewer.
- Exporting or copying chat text creates a copy outside the index. Handle those copies as carefully as the original chats.
- The source contains no analytics integration. You do not need a separate ChatSearch account, but syncing needs your existing Claude or ChatGPT login.
- Local storage does not replace your employer's approval for using a browser extension with work data. This repository does not contain an independent security audit.

## Performance

The [scale test](test/unit/scale.test.ts) generates 10,000 synthetic chats with about 127 million characters. Running `npm test` prints indexing time, memory use and query timings for that generated history. These are synthetic measurements, not evidence from a real account or a guarantee for your device. The repository does not include a saved benchmark report for the earlier headline speed and memory figures.

The search engine uses compact filters to skip chats that cannot contain a match, then checks matching text. Long searches yield between batches and can be cancelled when the query changes. Actual speed depends on the query, history and device.

## Development

```sh
npm install
npm run build        # writes dist/; load it unpacked
npm run watch        # rebuild on change
npm test             # unit tests + a 10k-chat scale benchmark
npm run e2e          # headless Chromium against mock claude.ai / chatgpt.com
npm run zip          # release/chatsearch-<version>.zip
```

The end-to-end test needs Playwright's Chromium installed (`npx playwright install chromium`). It uses mock sites, not your real chat accounts. Passing it does not prove that current live Claude and ChatGPT accounts sync correctly.

GitHub Actions runs the typecheck, unit tests and build on every pull request. Pushing a tag such as `v1.0.2` also publishes a release with the zip built from this repo.

Layout: `src/sw` (service worker: sync, capture, routing), `src/offscreen` (search engine, semantic index, embedding worker), `src/content` (page fetch observer and jump-to-match), `src/ui` (Preact UI shared by the popup, side panel and full page), `src/shared` (storage, normalisation, Bloom filters, extraction, indexing).

## Notes

ChatSearch is not affiliated with Anthropic or OpenAI. It uses the same private web APIs the sites use, so a change on their side can break syncing until the extension is updated; search over what is already indexed keeps working.

MIT licensed.
