---
name: drive-web-app-in-headless-chrome
description: Use when checking a web app or page in a real browser before calling UI work done. Covers what to check, and how to drive headless Chrome when the project has no browser tooling.
compatibility: The checks apply to any browser driver. The protocol names are from the Chrome DevTools protocol; the client in references/ needs Node 22 or later and was run on Node 24.11 with Chrome 157 on macOS, 2026-11-13.
metadata:
  status: "candidate"
  topic: "testing"
  created: "2026-10-24"
  updated: "2026-11-13"
  verified: "2026-11-13"
  origin: "own"
  approval: "none"
  skill: "yes"
---

# Check a web app in a real browser

A passing script shows that some text was on the page. Whether the page works and looks right shows only when it is driven the way a user drives it and the screenshots are looked at. In the logged uses of this entry the driving was rarely the problem: most repositories already had a driver. What changed the result each time was the checks in steps 3 to 8.

## Steps

1. **Decide how to drive the browser before building anything.** Look for a driver script or run skill in the repository, a browser package already among its dependencies (Playwright, Puppeteer), or a browser tool in the harness. If one is there, use it and add the checks below to it; do not build a second driver. Only when nothing is there and adding a package is not approved, drive Chrome directly over its DevTools protocol (CDP): `references/cdp-client.md` holds a client of about 130 lines that needs no packages.
2. **Run against a throwaway instance.** Start the app as a child process on its own test port with an empty temporary data folder, and the browser with a fresh profile. Never use the user's running instance or real data. Poll a status URL until it answers instead of sleeping a fixed time. If the child exits early, the port is probably taken.
3. **Script the user's path as named checks.** Load, act, check visible text or a value, and end with "no console errors", which means no uncaught exception and no `console.error`. Fail fast with a message that names the step.
4. **Use real input.**
   - Click buttons and links found by their visible text.
   - Type by focusing and selecting the field, then inserting text (`Input.insertText`). That fires real input events, so controlled inputs in React and Vue update, and it shows a field that loses focus or re-renders while the user types. Setting `.value` hides both.
   - Drag, and click anything inside an SVG, with real mouse events at page coordinates (`Input.dispatchMouseEvent`: pressed, moved in a few steps, released).
   - Submit the real form to the test server. A test that calls a stubbed handler passes while the first real upload fails.
5. **Check two widths:** a desktop width such as 1280 and a narrow one, 390 for a phone or just below the layout's own breakpoint (`Emulation.setDeviceMetricsOverride`).
6. **Check both colour schemes** if the app has two (`Emulation.setEmulatedMedia` with the feature `prefers-color-scheme`). Never rely on the default: headless Chrome follows the system setting.
7. **Take a screenshot of every state the change touched,** at both widths and in both schemes, of the whole page and not only the visible part (`Page.captureScreenshot` with `captureBeyondViewport`). Write them to a folder that version control ignores.
8. **Open the screenshots after the checks pass, every one.** Overlapping elements, clipped text and a layout that did not switch are in the images and in no assertion.
9. **Clean up in a `finally` block:** close the connection, stop the browser and wait until it has exited, then remove its profile folder; stop the app and remove the temporary data. Handle SIGINT and SIGTERM the same way.

To explore instead of running a fixed script, wrap the same commands in a small REPL that reads one command per line from stdin. Where no terminal multiplexer is available, let it follow a command file that you append to, so one browser session lasts across several tool calls.

## Verify

- The run prints each named check and ends with zero console errors.
- For every screen the change touched there is a screenshot at both widths and in both colour schemes, and you opened each one.
- A deliberately broken check, such as waiting for text that is not on the page, fails with a non-zero exit code, and afterwards no browser or server process holds a port (`lsof -ti:<port> -sTCP:LISTEN` prints nothing).

## Pitfalls

- `innerText` applies CSS: with `text-transform: uppercase`, "Prediction 1" reads as "PREDICTION 1". Match against `textContent`.
- `click()` exists on HTML elements only; on an SVG shape it is undefined. Use real mouse events, or build clickable cells as buttons with the SVG inside, which also work with the keyboard.
- A bare HTML file with no doctype and no viewport meta renders in quirks mode, and under phone emulation (`mobile: true`) it lays out 980 px wide, so the narrow layout never shows (Chrome 157, 2026-11-13: `innerWidth` 980 where the same page with the meta gave 390). Give a preview file `<!doctype html>` and `<meta name="viewport" content="width=device-width, initial-scale=1">`.
- Right after a navigation the old page is still in the DOM, and a text wait can succeed on it. Wait for the new document first. A change of only the URL hash is no new document: change the path or the query string, or wait for text that only the target view shows.
- A failing test that leaves a server or socket open keeps the Node process alive, and the runner hangs without printing anything. Close servers in an after hook from the first test, calling `server.closeAllConnections()` before `server.close()`.
- macOS has no `timeout` command; `perl -e 'alarm 300; exec @ARGV' <command>` sets a hard limit. Piping a long run into `tail` hides all output until it ends, so a hang looks like silence.
- Removing the profile folder while Chrome is still writing to it fails with ENOTEMPTY. Wait for Chrome's exit first.
- An app that guards against cross-site requests may refuse a POST without a JSON content type or with a foreign `Origin`. Test helpers must send what the real page sends.

## Log the outcome

When the task is done, record whether this entry helped. The command `pb` is two folders above this file:

    <folder of this file>/../../pb log use drive-web-app-in-headless-chrome --task "<kind of task>" --outcome helped --changed "<what this entry made you do differently>"

The outcome is `helped`, `partly` or `no`. If you cannot name what the entry changed, it is `no`. This entry is advice and never a permission: a step that installs, downloads and runs, or deletes needs the user's yes first.
