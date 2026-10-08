# Playwright harness

A starting point for `qa-test-session` probes. Adapt it per charter; keep each probe to one goal.

## Install (scratch, outside the repo)

Don't add Playwright to the repo's dependencies. If the repo already has it, use that; otherwise:

```bash
mkdir -p /tmp/qa-pw && cd /tmp/qa-pw && npm init -y >/dev/null && npm i playwright && npx playwright install --with-deps chromium
```

Put probe scripts in `/tmp/qa-pw` and run them with `QA_DIR=<abs path to repo>/qa/<session> TARGET_URL=<url> node probe.mjs`. Delete `/tmp/qa-pw` at cleanup.

## Dev server

If you start the app yourself, use the repo's documented command, log it, and record the pid:

```bash
mkdir -p "$QA_DIR/logs"
nohup npm run dev > "$QA_DIR/logs/dev-server.log" 2>&1 & echo "dev server pid $!" >> "$QA_DIR/notes.md"
# at cleanup:
pkill -TERM -P <pid>; kill <pid>; ss -ltnp | grep :<port> || echo "port free"
```

## Probe skeleton

```js
// probe.mjs
import { chromium } from 'playwright';
import fs from 'node:fs';
import path from 'node:path';

const OUT = process.env.QA_DIR;
const URL = process.env.TARGET_URL;
for (const d of ['shots', 'video', 'traces', 'logs']) fs.mkdirSync(path.join(OUT, d), { recursive: true });
const stamp = () => new Date().toISOString().slice(11, 23); // HH:MM:SS.mmm UTC
const log = (file, line) => fs.appendFileSync(path.join(OUT, 'logs', file), `${stamp()} ${line}\n`);

const touch = { isMobile: true, hasTouch: true };
const SIZES = {
  'small-phone':      { viewport: { width: 360, height: 640 }, deviceScaleFactor: 3, ...touch },
  'large-phone':      { viewport: { width: 430, height: 932 }, deviceScaleFactor: 3, ...touch },
  'phone-landscape':  { viewport: { width: 932, height: 430 }, deviceScaleFactor: 3, ...touch },
  'tablet':           { viewport: { width: 768, height: 1024 }, deviceScaleFactor: 2, ...touch },
  'tablet-landscape': { viewport: { width: 1024, height: 768 }, deviceScaleFactor: 2, ...touch },
  'laptop':           { viewport: { width: 1366, height: 768 } },
  'wide-desktop':     { viewport: { width: 1920, height: 1080 } },
  'laptop-zoom-200':  { viewport: { width: 683, height: 384 }, deviceScaleFactor: 2 },
};

const browser = await chromium.launch(); // one browser per run
try {
  for (const [size, opts] of Object.entries(SIZES)) {
    const context = await browser.newContext({ ...opts, recordVideo: { dir: path.join(OUT, 'video'), size: opts.viewport } });
    await context.tracing.start({ screenshots: true, snapshots: true });
    const page = await context.newPage();
    log('steps.log', `[${size}] video t0`);
    page.on('console', (m) => log('console.log', `[${size}] ${m.type()}: ${m.text()}`));
    page.on('pageerror', (e) => log('console.log', `[${size}] PAGEERROR ${e.message}`));
    page.on('requestfailed', (r) => log('network.log', `[${size}] FAILED ${r.method()} ${r.url()} ${r.failure()?.errorText}`));
    page.on('response', (r) => { if (r.status() >= 400) log('network.log', `[${size}] ${r.status()} ${r.url()}`); });
    try {
      log('steps.log', `[${size}] goto ${URL}`);
      await page.goto(URL, { waitUntil: 'load' });
      await page.screenshot({ path: path.join(OUT, 'shots', `01-${size}-first-look.png`), fullPage: true });
      // The probe: exact values, log before and after each step, screenshot each notable state.
    } finally {
      const video = page.video();
      await context.tracing.stop({ path: path.join(OUT, 'traces', `${size}.zip`) });
      await context.close(); // finalizes the video
      if (video) log('steps.log', `[${size}] video ${await video.path()}`);
    }
  }
} finally {
  await browser.close();
}
```

Map a log time to a video position as (log time − that context's "video t0"); it is accurate to a few hundred milliseconds. For anything tighter, use the action timestamps inside the trace zip.

## Memory and leak probe (Chromium)

Repeat the suspected flow (open and close a dialog, navigate away and back, run the game loop) N times and log heap, DOM nodes, and listeners after a forced GC each round. Steady growth that never comes back down is a leak: top severity.

```js
const cdp = await context.newCDPSession(page);
await cdp.send('Performance.enable');
async function memory() {
  await cdp.send('HeapProfiler.collectGarbage');
  const { metrics } = await cdp.send('Performance.getMetrics');
  const get = (n) => metrics.find((m) => m.name === n)?.value;
  return { heapMB: +(get('JSHeapUsedSize') / 1048576).toFixed(1), nodes: get('Nodes'), listeners: get('JSEventListeners') };
}
for (let i = 1; i <= 20; i++) {
  // ...one round of the flow...
  log('memory.log', `round ${i} ${JSON.stringify(await memory())}`);
}
```

Also watch the dev server's and browser's resident memory with `ps -o pid,rss,etime,cmd -p <pids>` during long runs.
