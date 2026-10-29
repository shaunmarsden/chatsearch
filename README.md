# ChatSearch

Real search for your Claude and ChatGPT history, as a Chrome extension. Every word of every chat, searchable as you type, with typo tolerance and search by meaning. Your chats are indexed inside your own browser and never leave it.

This is my fork of [Aaron Bergman's ChatSearch](https://github.com/AaronBergman/chatsearch). The one real change is that I removed the optional cloud mode, so no setting can send your chats anywhere. Everything else is Aaron's work.

![ChatSearch popup searching for "sourdough"](assets/screens/popup-search.png)

**[Download the latest release](https://github.com/shaunmarsden/chatsearch/releases/latest/download/chatsearch.zip)** · [Original project](https://github.com/AaronBergman/chatsearch)

## Why I forked it

This extension reads every chat you have had with Claude and ChatGPT. If you work in sales, those chats hold client and prospect details. So before putting my name to it, I checked what it sends and where.

- **By default, your chats stay in your browser.** The extension only talks to claude.ai and chatgpt.com, using your existing login, and to Hugging Face once to download the search model. There are no analytics and no tracking. I checked the published zip as well as the source, and both reach the same places.
- **The exception was cloud mode.** It was an optional setting for search by meaning, off by default. When it was on, chunks of your chats went to Aaron's server and on to DeepInfra, a third-party AI company. The server code on GitHub doesn't keep the text, but nobody outside can check that the server actually running matches that code. And for a tool that holds client details, one click in Settings is too easy to turn on by mistake.

So I took cloud mode out completely. I also added a GitHub build that runs the tests and makes the download from this repo's own code, so the zip you get matches the code you can read here.

One thing this doesn't fix: the extension reads Claude and ChatGPT through their private web routes, as every tool like it has to. If either site changes, syncing can break until the extension is updated.

## What it does

- **Instant full-text search** across claude.ai and chatgpt.com: titles, your messages, the replies, artifacts, canvas documents and pasted attachments. Results update on every keystroke, with the matching passage highlighted.
- **Substring matching**, so `plane` finds "airplanes" and `kube` finds "Kubernetes".
- **Typo tolerance.** Fuzzy title matching (`flt wk` finds "Fleet Week") and "did you mean" corrections (`risoto` searches for "risotto").
- **Search by meaning.** Press Shift+Enter or the Meaning button to find chats about an idea when you don't remember the words. Runs a small embedding model on your own machine by default.
- **Jump to the match.** Opening a result scrolls the chat to the passage you searched for and highlights every match, with a small navigator to step through them.
- **Always up to date.** It reads your history with your existing login, then keeps itself current in the background. Chats you open or reply to are picked up instantly from the page itself, with no extra requests.
- **Preview pane** with the full chat rendered as Markdown, copy as Markdown, download as `.md`.
- **Filters and operators**: source, your messages only, time range, sort by relevance or date.
- **Export everything** as a zip of Markdown files, and **import** official Claude and ChatGPT data exports.
- **Search history**, a persistent popup that remembers your last search, an optional side panel, a full-page view, an address-bar keyword (`cs`), and a right-click "Search my chats for ..." on selected text.

## Install

ChatSearch isn't on the Chrome Web Store yet, so it installs as an unpacked extension (takes a minute):

1. Download [`chatsearch.zip`](https://github.com/shaunmarsden/chatsearch/releases/latest/download/chatsearch.zip) and unzip it.
2. Open `chrome://extensions` and turn on **Developer mode** (top right).
3. Click **Load unpacked** and choose the unzipped `chatsearch` folder.
4. Pin it: click the puzzle-piece icon in the toolbar, then the pin next to ChatSearch.
5. Make sure you're signed in to claude.ai and/or chatgpt.com in this browser. Indexing starts on its own.

Open it with the toolbar icon or **Cmd+Shift+K** (Ctrl+Shift+K on Windows and Linux). Chrome 120 or newer. Other Chromium browsers (Edge, Brave, Arc) should work the same way.

To update, download the new zip, replace the folder's contents, and click the reload icon on the extension's card in `chrome://extensions`. Your index is kept.

## Using it

Type to search. The list updates as you type; the preview on the right shows the selected chat with every match highlighted.

| Key | Action |
| --- | --- |
| ↑ ↓ | Select a result |
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

- **Claude**: the full history is listed and downloaded on first run (a few thousand chats take a few minutes), then a light incremental check runs every minute or two.
- **ChatGPT** only allows a slow pace of full-conversation reads, shared with your own use of the site. ChatSearch therefore lists every chat right away (titles are searchable immediately), downloads full text at a gentle pace, pauses while you are using ChatGPT, and backs off for half an hour or more if ChatGPT ever says to slow down. To get everything at once, import your ChatGPT data export (Settings, Your data, Import): ChatGPT emails you a zip, and ChatSearch reads it locally.
- If a site shows your browser a "checking your browser" page instead of data (common in brand-new browser profiles), ChatSearch says so and resumes the moment you open that site in a tab.
- **Live capture**: when you open or reply to a chat on either site, ChatSearch indexes what the page already loaded, so the chats you actually use are always fresh without extra requests.
- Deleted and renamed chats are noticed and updated.

## Search by meaning

Turned on by default, and you can switch it off in Settings. It uses [MongoDB/mdbr-leaf-ir](https://huggingface.co/MongoDB/mdbr-leaf-ir), a 23M-parameter retrieval model, through transformers.js. It runs on the GPU via WebGPU when available, otherwise on the CPU. The model is downloaded once from Hugging Face (23 to 46 MB). Embedding happens in the background, newest chats first, at about half speed on power and a lighter trickle on battery, and every passage is embedded exactly once (vectors are keyed by a hash of the text).

Typed queries are capped at 1,000 characters, and a semantic search only runs when you ask for it (or automatically once when an exact search finds nothing), never on every keystroke.

## Privacy

- Your chats, the search index and the vectors live in this browser's IndexedDB. Uninstalling the extension deletes them; Settings also has a delete button.
- Network requests go only to claude.ai and chatgpt.com (with your existing login, exactly like the sites themselves), and to Hugging Face once to download the on-device model. Nothing else, and no setting changes that.
- No analytics, no tracking, no accounts.

## Performance

Measured on a synthetic history of 10,000 chats and 127 million characters (larger than almost anyone's): searches take 2 to 105 ms with the whole index in memory, about 145 MB in an offscreen document that is unloaded after a few idle minutes. Each chat gets a trigram Bloom filter (about 3 KB on average) so most chats are ruled out without reading them, and matches are verified exactly. Typing never blocks: long searches yield to the UI and are abandoned as soon as you type the next character.

## Development

```sh
npm install
npm run build        # writes dist/; load it unpacked
npm run watch        # rebuild on change
npm test             # unit tests + a 10k-chat scale benchmark
npm run e2e          # headless Chromium against mock claude.ai / chatgpt.com
npm run zip          # release/chatsearch-<version>.zip
```

GitHub Actions runs the typecheck, unit tests and build on every pull request. Pushing a tag such as `v1.0.2` also publishes a release with the zip built from this repo.

Layout: `src/sw` (service worker: sync, capture, routing), `src/offscreen` (search engine, semantic index, embedding worker), `src/content` (page fetch observer and jump-to-match), `src/ui` (Preact UI shared by the popup, side panel and full page), `src/shared` (storage, normalization, Bloom filters, extraction, indexing).

## Notes

ChatSearch is not affiliated with Anthropic or OpenAI. It uses the same private web APIs the sites use, so a change on their side can break syncing until the extension is updated; search over what is already indexed keeps working.

MIT licensed.
