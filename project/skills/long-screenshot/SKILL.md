---
name: long-screenshot
description: Capture full-page long screenshots of web pages using Playwright MCP, handling non-body scroll containers, sticky headers, lazy-loaded content, and pages over 8192px. Use when user asks for "long screenshot", "full page capture", "scroll screenshot", or needs to capture an entire scrollable page.
---

# Long Screenshot Skill

Capture full-page screenshots of web pages, automatically handling the specific challenges of this project's layout (non-body scroll containers in `main.content`).

## Trigger

When the user asks to:
- "Take a long screenshot of ..."
- "Capture the full page ..."
- "Screenshot the entire page ..."
- "Full page capture ..."
- "Scroll screenshot ..."
- Any request to capture a complete scrollable page

## Pre-Flight Check

1. Verify the browser is already open on the target page (check current URL)
2. If not, navigate to the URL first
3. If login is needed, perform login via Playwright MCP

## Step 1: Detect Scroll Container

Run `browser_run_code` to detect the real scroll container:

```javascript
async (page) => {
    const info = await page.evaluate(() => {
        const candidates = [
            document.body, document.documentElement,
            ...document.querySelectorAll('main, [class*="content"], [class*="scroll"], [class*="container"]')
        ];
        let maxScroll = { el: 'body', scrollHeight: 0 };
        for (const el of candidates) {
            if (!el) continue;
            const style = getComputedStyle(el);
            const hasScroll = style.overflow === 'auto' || style.overflow === 'scroll'
                || style.overflowY === 'auto' || style.overflowY === 'scroll';
            if (hasScroll && el.scrollHeight > maxScroll.scrollHeight) {
                maxScroll = {
                    el: el.tagName + (el.className ? '.' + el.className.split(' ')[0] : ''),
                    scrollHeight: el.scrollHeight,
                    clientHeight: el.clientHeight,
                };
            }
        }
        const bodySH = document.documentElement.scrollHeight;
        const mainSH = document.querySelector('main') ? document.querySelector('main').scrollHeight : 0;
        return {
            bodySH, mainSH,
            inner: maxScroll.scrollHeight > bodySH ? maxScroll : null,
            effective: Math.max(bodySH, mainSH, maxScroll.scrollHeight),
        };
    });
    return JSON.stringify(info);
}
```

## Step 2: Choose Strategy Based on Height

### Strategy A: Simple fullPage (body scrolls, height <= 8192px)

If `inner === null` and `effective <= 8192`:

```
browser_take_screenshot(fullPage=true, filename="test_screenshots/{name}_fullpage_{date}.png")
```

### Strategy B: Viewport Expansion (non-body scroll, height <= 8192px)

This is the MOST COMMON case for this project (main.content is the scroll container).

Run `browser_run_code`:

```javascript
async (page) => {
    // Get main content height
    const mainHeight = await page.evaluate(() => {
        return document.querySelector('main.content').scrollHeight;
    });

    // Set viewport to full content height
    await page.setViewportSize({ width: 1920, height: mainHeight });
    await page.waitForTimeout(500);

    // Expand scroll container overflow
    await page.evaluate(() => {
        const candidates = document.querySelectorAll('main, [class*="content"], [class*="scroll"]');
        for (const el of candidates) {
            const style = getComputedStyle(el);
            if (style.overflow === 'auto' || style.overflow === 'scroll'
                || style.overflowY === 'auto' || style.overflowY === 'scroll') {
                el.style.overflow = 'visible';
                el.style.height = 'auto';
                el.style.maxHeight = 'none';
            }
        }
        document.body.style.overflow = 'visible';
        document.body.style.height = 'auto';
    });
    await page.waitForTimeout(300);

    // Screenshot (no fullPage needed)
    await page.screenshot({
        path: 'test_screenshots/{name}_fullpage_{date}.png',
        scale: 'css',
        type: 'png'
    });

    // Restore viewport
    await page.setViewportSize({ width: 1280, height: 720 });
    return `Captured at 1920x${mainHeight}`;
}
```

### Strategy C: Scroll + Stitch (height > 8192px)

For extremely long pages that exceed Chromium's CDP limit.

Use the Python script:

```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe scripts/long_screenshot.py \
    --url {url} \
    --output test_screenshots/{name}_long_{date}.png \
    --expand-scroll
```

Optional flags:
- `--fix-sticky` — Override sticky/fixed headers
- `--force-lazy` — Force lazy-loaded images
- `--scroll-to-bottom` — For infinite scroll pages

## Step 3: Optional Pre-Capture Fixes

### Fix sticky/fixed headers (add before screenshot)

```javascript
async (page) => {
    await page.addStyleTag({content: `
        header, nav, .sticky-header, .fixed-navbar, .module-header-with-back,
        [style*="position: fixed"], [style*="position: sticky"] {
            position: static !important;
        }
    `});
    await page.waitForTimeout(300);
    return 'Sticky headers fixed';
}
```

### Force lazy-loaded images (add before screenshot)

```javascript
async (page) => {
    await page.evaluate(() => {
        document.querySelectorAll('img[loading="lazy"]').forEach(img => { img.loading = 'eager'; });
    });
    await page.waitForLoadState('networkidle');
    await page.evaluate(() => {
        return Promise.all(
            Array.from(document.images).map(img => {
                if (img.complete) return Promise.resolve();
                return new Promise(r => { img.onload = r; img.onerror = r; });
            })
        );
    });
    return 'Lazy images forced';
}
```

## Step 4: Verify with Vision MCP

After capture, verify the screenshot quality:

```
mcp__MiniMax__understand_image(
    image_source = "test_screenshots/{name}_fullpage_{date}.png",
    prompt = "Verify this full-page screenshot: 1) Is ALL content captured from top to bottom? 2) Is there any truncation? 3) Are all tables/lists fully visible? 4) Any duplicate headers?"
)
```

## Decision Flowchart

```
User requests full page screenshot
    |
    v
Is browser already on target page? --No--> Navigate + Login
    |Yes
    v
Detect scroll container (Step 1)
    |
    v
inner_scroll detected? --No--> body scrollHeight <= 8192? --Yes--> Strategy A (fullPage)
    |Yes                                      |No
    v                                         v
effective_height <= 8192? --Yes--> Strategy B (viewport expand)
    |
    |No
    v
Strategy C (Python scroll+stitch script)
```

## Output

- Screenshot saved to `test_screenshots/` with naming: `{area}_fullpage_{YYYYMMDD}.png`
- Vision MCP verification result reported to user
- If issues found (truncation, duplicates), apply fix and re-capture
