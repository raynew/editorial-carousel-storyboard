# Editorial Carousel Storyboard

A single-file web app for designing Instagram portrait carousels (4:5). Open `carousel.html` in a browser — no build step or server required.

Edit copy and layout on the left, preview the live 4:5 card on the right, reorder slides in the thumbnail rail, then export PNG slides or a JSON project backup.

## Run it

- **Live site (GitHub Pages):** [https://raynew.github.io/editorial-carousel-storyboard/](https://raynew.github.io/editorial-carousel-storyboard/)
- **Local:** open `carousel.html` in a modern browser (Chrome, Edge, or Firefox). No build step.

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
| `layout` | `photo-first`, `text-first`, `stacked`, or `text-only` |
| `wallpaper` | Textured background on a `text-only` slide: `paper` (default), `watercolour`, `riso`, `marble`, `terrazzo`, `kraft`, `midnight`, `chalk` |
| `backdrop` | Overlay: `none`, `dark`, `teal` |
| `contrast` | Text colour: `standard`, `cream`, `gold`, `teal`, `blue` |
| `hideText` | Hide all type on the card |
| `hideBorder` | Hide the inner gold frame |
| `isItalic` | Italicize headline and body |
| `font` | Headline and body font key from `FONTS`: `playfair` (default), `franklin`, or a handwriting face: `caveat`, `dancing`, `homemade`, `kalam`, `marker`, `shadows` |
| `frame` | Polaroid photo frame: `none` (default), `white`, `cream`, `black`, `gold`, `teal`, `blue`, or `custom` |
| `frameColor` | The hex colour picked for a `custom` frame, e.g. `#d9534f` |
| `borderColor` | Inner border line colour: `gold` (default), `cream`, `white`, `black`, `teal`, `blue` |
| `aiFlags` / `aiFlags2` | Reasons the metadata of photo 1 / photo 2 gives for generative AI use, e.g. `["Midjourney"]`. Empty when none were found |
| `textOffsets` | Where each text piece was dragged, e.g. `{ "headline": { "x": 0, "y": 12.5 } }`, as % of card width (x) and height (y) |

Older backups without these fields open with the defaults.

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

Rebuilds the thumbnail rail, which wraps onto extra rows as slides are added. Each thumb is clickable (select slide) and draggable (reorder). While dragging, a line shows the slot the slide will land in, and anywhere in the Slide sequence section accepts the drop. No line means the drop would not move the slide. Dropping updates `slides` and `activeIndex` so the selection stays on the same card, and the moved thumb flashes. Updates the “N slides” label.

### `applyTextLayout()`

Collapses the Story structure and Words & direction panels while the open slide has **Hide all text** on. When every slide has text hidden, the Slide sequence also moves into the left column, which gets wider. Mixed slides keep the standard layout, so the thumbs stay put when moving between slides.

### Multi-select

Tick a thumb's checkbox or Ctrl/⌘-click to select several slides, Shift-click to select a range, or use **Select all**. A plain click clears the selection. **Delete selected** asks for confirmation; deleting every slide leaves one blank slide. Dragging moves one slide at a time and leaves the selection unchanged.

### Keyboard controls

Outside text fields and dropdowns: arrow keys move between slides, Shift+arrow extends the selection, Delete or Backspace removes the selected slides (with confirmation) or the open slide (without), and Esc clears the selection.

### Fonts, frames and border colours

**Headline & body font** (Story structure panel) switches the headline and body between two editorial faces and six handwriting faces. The small labels (eyebrow, number, caption, CTA) stay in Libre Franklin. Each face in `FONTS` has its own size scale and spacing so they look roughly the same size. The export waits for the slide's font to download before drawing.

**Photo frame** (Photography panel) puts the photos on a coloured polaroid mat that fills from the card's edge in to the photo: white, cream, black, gold, teal, blue, or any colour from the picker that appears with **Custom colour…**. The mat has even margins on three sides and a deeper one at the bottom. `getPhotoRects(slide)` gives the photo areas for every layout, and both the preview and the export use it. **Border colour** sets the colour of the thin inner border line.

### Textured wallpapers and text-only slides

The top of the **Story structure** panel has a row of wallpaper swatches. Picking one makes the slide a text-only slide: the photo is hidden, the text is switched on, and the text and border colours change to ones that read well on that texture (change them afterwards if you like). **Photo** puts the slide's photo back, with the layout, tint and colours it had before. The photo stays on the slide while it's text-only, but it isn't shown, exported or counted in the duplicate and AI notices, and folder upload skips text-only slides. Choosing **Text only (Textured wallpaper)** in Slide composition does the same with cream paper, and **Add text slide** adds a new text slide after the open one, on the same wallpaper.

The textures (cream paper, watercolour wash, riso glow, gold-vein marble, terrazzo, kraft paper, midnight gold leaf, chalkboard) are painted in the browser by `WALLPAPERS[key].paint` from a fixed seed, so each looks the same every time and only the key is saved in the project. `paintWallpaper(key, scale)` paints one at any size. The preview paints each wallpaper the first time a slide shows it (`getWallpaperArt`), the swatches and slide thumbs use a small copy (`getWallpaperThumb`), and the export paints it at full 2160 × 2700 so the grain stays sharp (`getExportWallpaper`). Soft shapes use the canvas blur filter, or radial gradients in browsers without it. On a text-only slide the headline block sits in the middle of the card.

### Moving text

Drag any text piece on the preview to move it. It can't be dragged off the card. Double-click a piece, or use **Reset text positions**, to put it back. Offsets are stored per slide in `textOffsets` and applied to the export.

### `findDuplicatePhotos()`

Scans the photo file names on all slides and groups possible duplicates, ignoring case, extension, and copy markers (`IMG_1 (1).jpg`, `IMG_1 - Copy.jpg`). A bare `(1)` only counts when the unnumbered name is also present. `renderRail()` badges the matching thumbs and lists the groups above the rail. It only compares names, so photos that differ but share a name are flagged too, and nothing is removed automatically.

### AI metadata check

Each photo is checked when it's uploaded, singly or by folder. `detectAiMetadata(bytes)` reads only the metadata parts of the file (JPEG APP and comment segments, PNG text chunks including compressed ones, WebP metadata chunks, or the start and end of other formats). It looks for markers that generative AI tools write:

- IPTC digital source types `trainedAlgorithmicMedia` (AI-generated), `compositeWithTrainedAlgorithmicMedia` (edited with AI, e.g. Photoshop Generative Fill) and `compositeSynthetic`, which C2PA Content Credentials also use
- Stable Diffusion generation settings and ComfyUI workflows in PNG text
- Tool names such as Midjourney, DALL·E, OpenAI, Adobe Firefly, Google Imagen/Gemini, NovelAI, Ideogram, Leonardo.Ai, FLUX and Meta AI
- Text labels like "AI-generated"

A flagged photo gets an amber **AI** badge on its thumb, a warning under its upload field, and a line in the notice above the rail. Nothing is blocked or removed. Backups saved before this check are scanned when they're loaded.

It only reads what the metadata says. Social platforms, messaging apps and screenshots often strip metadata, so a photo with no warning may still be AI-generated, and a real photo whose description mentions one of these tools will be flagged.

### Folder upload

**Upload photo folder** loads every image in the chosen folder, including subfolders, as one slide per photo in file name order. Empty slides fill first, then new slides are added. The per-slide upload inputs remain for swapping one photo and for Photo 2 in stacked layout.

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

### `measureSlideText(slide, targetW)` / `drawTextLines(ctx, lines)`

The export draws text from the preview's own layout. `measureSlideText` paints the slide onto a hidden 1080px copy of the preview card, waits for its fonts, then reads every line of text back: where the browser broke the lines, where each line sits (drag offsets included), and its font, letter spacing, colour, opacity and shadow, scaled to the export size. `drawTextLines` draws those lines on the canvas. Text on the card is sized in `cqw` (a share of the card's width), so the preview is the export in miniature and the PNG comes out the same at any window size.

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

Paints one slide onto an export canvas: frame, photos, layout, overlay, border, and type. Shared by download, share, and publish.

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

Export recreates layout (photo-first, text-first, stacked), photo frame, overlay and border, and draws the text exactly as the preview lays it out (see `measureSlideText`).

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
- **Text only** — no photo; a textured wallpaper fills the card behind the text. The photo controls are hidden.

## Notes

- The small label in the bottom-right corner is the version: `v` plus the number of commits on `main`. The Pages workflow stamps it in when it deploys (hover for the commit and date). Opened locally, it reads `dev`.

- Export size is 2160×2700 (4:5 at a high Instagram-friendly resolution).
- Browsers may prompt once per PNG; allow multiple downloads if asked.
- Lucide is loaded on `DOMContentLoaded` if present; the UI does not depend on icons to work.
