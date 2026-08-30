# Editorial Carousel Storyboard

A single-file web app for designing Instagram portrait carousels (4:5). Open `carousel11.html` in a browser — no build step or server required.

Edit copy and layout on the left, preview the live 4:5 card on the right, reorder slides in the thumbnail rail, then export PNG slides or a JSON project backup.

## Run it

- **Live site (GitHub Pages):** [https://raynew.github.io/editorial-carousel-storyboard/](https://raynew.github.io/editorial-carousel-storyboard/)
- **Local:** open `carousel11.html` in a modern browser (Chrome, Edge, or Firefox). No build step.

Photos stay in memory as data URLs. Saving the project writes them into the JSON file so the backup is self-contained.

CDN assets (Tailwind, Lucide, Google Fonts) need a network connection the first time the page loads.

## Slide data

Each item in the `slides` array is one card:

| Field | Meaning |
| --- | --- |
| `stage` | Eyebrow / narrative beat (Hook, Context, …) |
| `number` | Slide label, e.g. `01 / 06`. Auto-filled if it matches `NN / NN` |
| `headline` | Main title |
| `body` | Body copy; line breaks are preserved |
| `caption` | Footer caption |
| `cta` | Call to action |
| `imageUrl` / `imageUrl2` | Photo 1 and (stacked layout) photo 2 as data URLs |
| `imageFit` | `cover` (crop) or `contain` (letterbox) |
| `focal` / `focal2` | Crop anchor: `center`, `top`, `bottom` |
| `layout` | `photo-first`, `text-first`, or `stacked` |
| `backdrop` | Overlay: `none`, `dark`, `teal` |
| `contrast` | Text colour: `standard`, `cream`, `gold`, `teal`, `blue` |
| `hideText` | Hide all type on the card |
| `hideBorder` | Hide the inner gold frame |
| `isItalic` | Italicize headline and body |

## Functions

### `safeText(value)`

Returns a trimmed string. Empty or missing values become `""`. Used when writing headline, caption, and CTA from the form so stray whitespace is not stored.

### `normalizeNumbers()`

Walks every slide. If `number` is missing or looks like `01 / 06`, it rewrites it as the current index and deck length (`01 / 03`, `02 / 03`, …). Custom labels that do not match that pattern are left alone.

### `setMessage(message)`

Shows status text under the editor (save, load, export, errors).

### `fillEditor()`

Copies the active slide into the form: text fields, layout, fit, focal points, backdrop, contrast, and checkboxes. Shows the second focal-point control only for stacked layout. Clears the file inputs so you can re-upload the same file. Updates “Slide X of Y”.

### `paintCard(slide)`

Renders the live 4:5 preview: layout class, overlay and contrast classes, border/text/italic flags, background images (or a placeholder gradient), focal positions, and all copy. Photo 2 is used in stacked layout.

### `renderRail()`

Rebuilds the thumbnail strip. Each thumb is clickable (select slide) and draggable (reorder). Dropping updates `slides` and `activeIndex` so the selection stays on the same card. Updates the “N slides” label.

### `updateInterface()`

Full refresh: `normalizeNumbers()`, `fillEditor()`, `paintCard()`, `renderRail()`, then enables/disables Previous/Next and updates the inline counter.

### `syncFromEditor()`

Writes the form back onto the active slide, toggles stacked focal-point UI, then `paintCard` and `renderRail`. Bound to `input` and `change` on every editor field so the preview stays live.

### `fileToDataUrl(file)`

Reads an uploaded image with `FileReader` and returns a Promise of a data URL. Photos can then be saved inside the JSON backup.

### `loadImage(src)`

Loads an image for canvas export. Resolves `null` if `src` is empty. Sets `crossOrigin` so canvas export can stay un-tainted when the source allows it.

### `drawNativeImage(ctx, img, x, y, w, h, fit, focal)`

Draws a photo into a clipped rectangle on the export canvas.

- **contain**: scale to fit, centre, fill leftover with paper colour `#fffaf0`.
- **cover**: scale to fill; `focal` (`top` / `bottom` / `center`) chooses the crop when the image is taller than the box.

### `wrapText(ctx, text, x, y, maxWidth, lineHeight)`

Draws multi-line text on the canvas. Splits on newlines first, then wraps words to `maxWidth`. Used for headline and body in the PNG export.

## UI actions (event handlers)

These are not named functions; they are listeners on buttons and inputs.

| Control | Behaviour |
| --- | --- |
| **Clear all text** | Empties stage, number, headline, body, caption, and CTA on the current slide |
| **Photo 1 / Photo 2** | Converts the file to a data URL and paints the card |
| **Save Project** | Downloads `carousel-storyboard-backup.json` with the full `slides` array (including embedded photos) |
| **Load Project** | Parses a JSON backup, replaces `slides`, selects slide 1 |
| **Previous / Next** | Move `activeIndex` and refresh |
| **Add slide** | Appends a blank beat and selects it |
| **Duplicate** | Inserts a copy after the current slide; appends ` (copy)` to the headline if there is one |
| **Delete** | Removes the current slide; refuses to delete the last remaining slide |
| **Download Carousel** | Renders each slide to a 2160×2700 PNG (`carousel-slide-01.png`, …) via canvas and triggers a download per slide |
| **Share to Instagram** | Renders JPEG slides and opens the system share sheet (phones) so you can pick Instagram. Does not skip Instagram’s composer. |
| **Publish via Meta API** | Uploads JPEGs to your public image host, then creates and publishes a photo or carousel through Instagram’s Graph API. Professional accounts only. |

### `drawSlideOnCanvas(ctx, slide, targetW, targetH)`

Paints one slide onto an export canvas: photos, layout, overlay, border, and type. Shared by download, share, and publish.

### `renderSlidesToFiles({ mimeType, quality, extension })`

Renders the whole deck to `File` objects (PNG for local download, JPEG for Instagram).

### `canvasToBlob(canvas, mimeType, quality)`

Promise wrapper around `canvas.toBlob`.

### `downloadFiles(files)`

Triggers a sequential local download for each file.

### `graphRequest(version, path, { method, params })`

Calls `https://graph.facebook.com/{version}/{path}`. Throws if Meta returns `error`.

### `waitForContainer(version, containerId, token)`

Polls a media container until `FINISHED` (or errors / times out). Required before publish.

### `uploadSlideForInstagram(endpoint, file)`

POSTs a slide as multipart `file` to your host. Expects `{ "url": "https://..." }` because Instagram must fetch the image itself.

### `loadIgSettings()` / `saveIgSettings()`

Read and write Instagram user ID, token, upload URL, and Graph version in `localStorage` (`carousel-ig-settings`). Tokens never leave this browser unless you click publish.

Export recreates layout (photo-first, text-first, stacked), overlay, border, type colour, italics, wrapping, and captions so the PNG matches the on-screen composition as closely as canvas text allows.

## Instagram: what is and is not possible

A static page **cannot** silently post to a normal personal Instagram account. Instagram has no “upload these local files as a feed post” API for consumer logins, and unofficial bots violate Instagram’s terms.

Two supported paths:

1. **Share to Instagram (phones)** — uses the Web Share API with the rendered JPEGs. You still confirm the post in the Instagram app. Desktop browsers usually cannot attach files this way; use **Download Carousel** instead.
2. **Publish via Meta API (Professional accounts)** — official Content Publishing:
   - Instagram Business or Creator account linked to a Facebook Page
   - Meta app with `instagram_business_content_publish` (or the equivalent content-publish permission)
   - Long-lived access token
   - A **public HTTPS** image host. Meta’s servers download each slide; `file://` and data URLs will not work.

Your upload endpoint should accept `POST` multipart field `file` and return JSON `{"url":"https://cdn.example/slide.jpg"}`. Then this app creates child containers (`is_carousel_item=true`), a parent `CAROUSEL` (2–10 slides) or a single image, and calls `media_publish`.

Keep **Download Carousel** for lossless local PNGs regardless of Instagram.

## Layouts

- **Photo-first** — image fills the card; type sits on top.
- **Text-first** — type on the upper 60%; photo in the lower 40% with slightly reduced saturation.
- **Stacked** — two photos, 50% / 50%; second upload and second focal point apply here.

## Notes

- Export size is 2160×2700 (4:5 at a high Instagram-friendly resolution).
- Browsers may prompt once per PNG; allow multiple downloads if asked.
- Lucide is loaded on `DOMContentLoaded` if present; the UI does not depend on icons to work.
