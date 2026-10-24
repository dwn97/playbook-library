# A minimal DevTools client in TypeScript

Runs on Node 22 or later with no packages (Node 24 runs `.ts` files directly; on older versions, rename to `.mjs` and drop the type annotations). Checked on Node 24.11 with Chrome on macOS, 2026-10-24.

```ts
import { spawn, type ChildProcess } from "node:child_process";
import { once } from "node:events";
import { existsSync, mkdtempSync, readFileSync, rmSync, writeFileSync } from "node:fs";
import { tmpdir } from "node:os";
import { join } from "node:path";
import { setTimeout as sleep } from "node:timers/promises";

// macOS default; on Linux usually "google-chrome" or "chromium".
const CHROME = process.env.CHROME_BIN ?? "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome";

export class Browser {
  readonly errors: string[] = [];
  private ws!: WebSocket;
  private chrome!: ChildProcess;
  private profile!: string;
  private next = 0;
  private pending = new Map<number, { resolve: (v: any) => void; reject: (e: Error) => void }>();

  static async launch(width = 1280, height = 900): Promise<Browser> {
    const b = new Browser();
    b.profile = mkdtempSync(join(tmpdir(), "cdp-profile-"));
    b.chrome = spawn(CHROME, ["--headless=new", "--remote-debugging-port=0", `--user-data-dir=${b.profile}`,
      "--no-first-run", "--no-default-browser-check", `--window-size=${width},${height}`, "about:blank"], { stdio: "ignore" });
    const portFile = join(b.profile, "DevToolsActivePort");
    for (let i = 0; i < 100 && !existsSync(portFile); i++) await sleep(100);
    const port = readFileSync(portFile, "utf8").split("\n")[0];
    const targets = (await (await fetch(`http://127.0.0.1:${port}/json/list`)).json()) as { type: string; webSocketDebuggerUrl: string }[];
    b.ws = new WebSocket(targets.find((t) => t.type === "page")!.webSocketDebuggerUrl);
    await new Promise((resolve, reject) => { b.ws.onopen = resolve; b.ws.onerror = reject; });
    b.ws.onmessage = (e) => b.receive(JSON.parse(String(e.data)));
    await b.send("Page.enable");
    await b.send("Runtime.enable");
    return b;
  }

  async close() {
    this.ws.close();
    this.chrome.kill("SIGTERM");
    if (this.chrome.exitCode === null && this.chrome.signalCode === null) await once(this.chrome, "exit");
    rmSync(this.profile, { recursive: true, force: true, maxRetries: 5 }); // only after Chrome has exited
  }

  send(method: string, params: object = {}): Promise<any> {
    const id = ++this.next;
    this.ws.send(JSON.stringify({ id, method, params }));
    return new Promise((resolve, reject) => this.pending.set(id, { resolve, reject }));
  }

  private receive(msg: any) {
    if (msg.id) {
      const p = this.pending.get(msg.id);
      this.pending.delete(msg.id);
      msg.error ? p?.reject(new Error(msg.error.message)) : p?.resolve(msg.result);
    } else if (msg.method === "Runtime.exceptionThrown") {
      this.errors.push(msg.params.exceptionDetails.exception?.description ?? msg.params.exceptionDetails.text);
    } else if (msg.method === "Runtime.consoleAPICalled" && msg.params.type === "error") {
      this.errors.push(msg.params.args.map((a: any) => a.value ?? a.description).join(" "));
    }
  }

  async eval<T>(expression: string): Promise<T> {
    const r = await this.send("Runtime.evaluate", { expression, returnByValue: true, awaitPromise: true });
    if (r.exceptionDetails) throw new Error(r.exceptionDetails.exception?.description ?? r.exceptionDetails.text);
    return r.result.value as T;
  }

  /** Full page load; waits until the old document is gone. */
  async goto(url: string) {
    await this.eval("window.__old = true");
    await this.send("Page.navigate", { url });
    for (let i = 0; i < 100; i++) {
      if (await this.eval<boolean>("window.__old !== true && document.readyState === 'complete'").catch(() => false)) return;
      await sleep(100);
    }
    throw new Error("the page did not load");
  }

  async waitForText(text: string, timeoutMs = 15_000) {
    const end = Date.now() + timeoutMs;
    while (Date.now() < end) {
      const body = await this.eval<string>("document.body ? document.body.textContent : ''").catch(() => "");
      if (body.includes(text)) return;
      await sleep(150);
    }
    throw new Error(`timed out waiting for "${text}"`);
  }

  async click(text: string, selector = "button, a, label") {
    const js = `(() => { const el = [...document.querySelectorAll(${JSON.stringify(selector)})]
      .find((e) => e.textContent.includes(${JSON.stringify(text)}) && !e.disabled);
      if (!el) return false; el.scrollIntoView({ block: "center" }); el.click(); return true; })()`;
    const end = Date.now() + 10_000;
    while (Date.now() < end) {
      if (await this.eval<boolean>(js)) return;
      await sleep(150);
    }
    throw new Error(`nothing to click with text "${text}"`);
  }

  async type(selector: string, text: string) {
    const ok = await this.eval<boolean>(`(() => { const el = document.querySelector(${JSON.stringify(selector)});
      if (!el) return false; el.focus(); el.select && el.select(); return true; })()`);
    if (!ok) throw new Error(`no field ${selector}`);
    await this.send("Input.insertText", { text });
  }

  async screenshot(file: string, width = 1280) {
    const { cssContentSize } = await this.send("Page.getLayoutMetrics");
    const clip = { x: 0, y: 0, width, height: Math.ceil(cssContentSize.height), scale: 1 };
    const { data } = await this.send("Page.captureScreenshot", { format: "png", captureBeyondViewport: true, clip });
    writeFileSync(file, Buffer.from(data, "base64"));
  }
}
```

## Using it

```ts
const browser = await Browser.launch();
try {
  await browser.goto("http://127.0.0.1:5299/");
  await browser.waitForText("Welcome");
  await browser.click("Start");
  await browser.type('input[name="title"]', "smoke test");
  await browser.screenshot("screenshots/after-start.png");
  if (browser.errors.length) throw new Error(browser.errors.join(" | "));
} finally {
  await browser.close();
}
```

Force a colour scheme: `await browser.send("Emulation.setEmulatedMedia", { features: [{ name: "prefers-color-scheme", value: "light" }] })`.
