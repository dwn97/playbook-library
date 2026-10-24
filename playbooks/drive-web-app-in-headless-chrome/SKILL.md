---
name: drive-web-app-in-headless-chrome
description: Use when you need to click through a local web app in a real browser (smoke test, screenshots, UI check) and no browser-automation package such as Playwright is available or approved.
compatibility: Chrome or Chromium with --headless=new (checked with Chrome on macOS, 2026-10-24). Node 22 or later for the built-in WebSocket and fetch (checked on Node 24.11). Any language with a WebSocket client works the same way.
metadata:
  status: "candidate"
  topic: "testing"
  created: "2026-10-24"
  updated: "2026-10-24"
  verified: "2026-10-24"
  origin: "own"
  approval: "none"
---

# Drive a local web app in headless Chrome over the DevTools protocol

Chrome can be driven directly over its DevTools protocol (CDP): start it headless with a debugging port, open a WebSocket to the page and send JSON commands. With Node's built-in `WebSocket` this needs no packages, so it works where Playwright or Puppeteer are not installed or need approval. About 150 lines cover launching, waiting, clicking, typing, screenshots and console errors; a skeleton is in `references/cdp-client.md`.

## Steps

1. **Run the app on a test port with throwaway data.** Start the server as a child process on its own port with an empty temporary data folder, never the user's running instance or real data. Poll a status URL with `fetch` until it answers instead of sleeping a fixed time. If the child exits early, the port is probably taken.
2. **Launch Chrome headless with a fresh profile:** `--headless=new --remote-debugging-port=0 --user-data-dir=<new temp folder> --no-first-run --no-default-browser-check --window-size=1280,900 about:blank`. Port 0 lets Chrome pick a free port and write it as the first line of `<profile>/DevToolsActivePort`; wait for that file.
3. **Connect to the page.** `GET http://127.0.0.1:<port>/json/list`, take the entry whose `type` is `page` and open a WebSocket to its `webSocketDebuggerUrl`. Each command is `{id, method, params}`, and its reply carries the same `id`. Send `Page.enable` and `Runtime.enable`, then collect `Runtime.exceptionThrown` events and `Runtime.consoleAPICalled` events of type `error`: those are the page's errors.
4. **Build a few commands on `Runtime.evaluate`** (with `returnByValue` and `awaitPromise`):
   - wait for text: poll `document.body.textContent` until it holds the text, with a timeout;
   - click: find the first enabled element by selector, filtered by its text, and call `el.click()`; retry until it appears;
   - type: focus and select the field in the page, then send `Input.insertText`, which fires real input events, so React and Vue controlled inputs update;
   - navigate: set a marker on `window`, send `Page.navigate` or `Page.reload`, then wait until the marker is gone and `document.readyState` is `complete`.
5. **Take screenshots of the whole page:** read `cssContentSize` from `Page.getLayoutMetrics`, then call `Page.captureScreenshot` with `captureBeyondViewport: true` and a clip of the full height. Write them to a folder that version control ignores.
6. **Script the user's path as named checks:** load, act, check visible text or a value, take a screenshot, and end with "no console errors". Fail fast with a message naming the step.
7. **Clean up in a `finally` block:** close the WebSocket, stop Chrome and wait until it has exited, then remove its profile folder; stop the app server and remove the temporary data. Handle SIGINT and SIGTERM the same way.
8. **Look at the screenshots** of every screen the change touched. Passing checks only show that the text was there; the images show whether the page looks right.

To explore instead of running a fixed script, wrap the same commands in a small REPL that reads one command per line from stdin. Where no terminal multiplexer is available, let it follow a command file that you append to, so one browser session lasts across several tool calls.

## Verify

- The run prints each named check and ends with zero console errors.
- You opened the screenshots of the screens the change touched, in both colour schemes if the app has two. Force one with `Emulation.setEmulatedMedia` and the feature `prefers-color-scheme`.
- A deliberately broken check, such as waiting for text that is not on the page, fails with a non-zero exit code, and afterwards no Chrome or server process holds a port (`lsof -ti:<port> -sTCP:LISTEN` prints nothing).

## Pitfalls

- `innerText` applies CSS: with `text-transform: uppercase`, "Prediction 1" reads as "PREDICTION 1". Match against `textContent`.
- Right after a navigation the old page is still in the DOM, and a text wait can succeed on it. Wait for the new document first (step 4). A change of only the URL hash is no new document: change the path or the query string, or wait for text that only the target view shows.
- A failing test that leaves a server or socket open keeps the Node process alive, and the runner hangs without printing anything. Close servers in an after hook from the first test, calling `server.closeAllConnections()` before `server.close()`.
- macOS has no `timeout` command; `perl -e 'alarm 300; exec @ARGV' <command>` sets a hard limit. Piping a long run into `tail` hides all output until it ends, so a hang looks like silence.
- Removing the profile folder while Chrome is still writing to it fails with ENOTEMPTY. Wait for Chrome's exit first.
- `click()` exists on HTML elements only; SVG shapes need a dispatched `MouseEvent`. Clickable cells built as buttons avoid this and work with the keyboard too.
- An app that guards against cross-site requests may refuse a POST without a JSON content type or with a foreign `Origin`. Test helpers must send what the real page sends.
- Headless Chrome followed the system's dark mode on macOS (2026-10-24), so screenshots can come out dark when the light theme was meant.
