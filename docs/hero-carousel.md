# Hero Image Carousel + Image Overlay — Behavior Spec

Full-width image carousel under the exhibition header, plus the full-screen overlay that opens when you click the centered image.

- **Reference prototype:** `r3-a.html` (markup in the `exh-masthead` section; JS in the "Header carousel" `<script>` block), styles in `styles/main.css` (`.hero-carousel*`, `.lightbox*`) and `styles/tokens.css`.
- **Figma:** carousel — desktop `2386:38712`, mobile `2386:39073`; overlay — `2391:55589` (desktop only; mobile adapted). File `kfWr23H6UOMF0XCG83EO7G`.
- **Breakpoint:** mobile `< 768px`, desktop `≥ 768px`.

---

## 1. Layout

| | Mobile | Desktop |
|---|---|---|
| Carousel height | 211px | 428px |
| Width | full screen width (edge to edge) | full screen width (edge to edge) |
| Gap between slides | 12px | 24px |
| Slide width | the image's natural aspect ratio at the carousel height, capped at `100vw − 72px` (`object-fit: cover`) so neighbours peek in | natural aspect ratio at 428px tall |
| Space above (from header buttons) | 32px | 48px |
| Carousel → dots | 24px | 32px |
| Dots → anchor link bar | 64px | 64px |

- **The active slide is always centered.** The previous and next slides peek in on either side.
- **Dots:**
  - One per image, laid out as 8×8px black squares with 4px between them.
  - The active dot is 100% opacity; the others are 40%.
  - Centered below the carousel.

**Order and start position**
- Slides are shown in a fixed order. On load, **slide 1 is centered** and its dot is active.
- Because the carousel loops, the **last slide peeks in on the left** of slide 1 when the page loads. Figma shows it this way.
- The prototype uses 4 images: wire sculpture, gallery view, folded paper, Desert Plant print.

---

## 2. Looping

- The carousel loops infinitely in both directions.
- **How the prototype does it:**
  - It adds a full copy of the slides before and after the real set: clones, then originals, then clones. Clones get `aria-hidden="true"`.
  - Once scrolling has settled (150ms after the last scroll event), if the centered slide is a clone, it jumps instantly to the same image in the real set.
  - The jump lands on the identical image at the identical position, so it can't be seen.
- **Dots** always reflect the real image index (centered slide index modulo the number of images).

---

## 3. Interaction

| Input | Behavior |
|---|---|
| Trackpad horizontal scroll, shift+mouse wheel, touch swipe | Native horizontal scrolling with `scroll-snap-type: x mandatory` and `scroll-snap-align: center`, so each slide snaps to the center. All breakpoints. Hide the scrollbar. |
| Mouse drag | Drag scrolls the carousel. A move under 4px still counts as a click. Snapping is off during the drag; on release it smooth-scrolls to the nearest slide. **A drag never counts as a click.** |
| Click a dot | Smooth-scrolls to that image, going whichever way round is shorter. |
| Click / tap a **peeking** (off-center) slide | Smooth-scrolls it to the center. Does **not** open the overlay. |
| Click / tap the **centered** slide | Opens the image overlay on that image (§4). Cursor: `zoom-in`. |
| Window resize | Re-centers the current image instantly. |

---

## 4. Image overlay

### Opening and closing

- **Opens:** on click or tap of the centered carousel slide, showing that image.
- **While open:**
  - The page can't scroll (`overflow: hidden` on `<html>`).
  - Focus moves to the close button.
  - **Tab / Shift+Tab** stay inside the overlay: close → previous → next.
- **Closes:** via the close (×) button or **Esc**.
- **On close:**
  - The carousel moves instantly to the image last shown in the overlay.
  - Focus returns to the element that opened it.
- **Markup:** `role="dialog"`, `aria-modal="true"`, `aria-label="Image viewer"`.

### Navigation

| Input | Behavior |
|---|---|
| Previous / next arrow buttons | Previous / next image, **looping** (from the last image, next goes to the first). |
| ← / → keys | Same as the arrows. |
| Touch swipe on the image (> 40px) | Swipe left = next, swipe right = previous. |
| Counter | "`n` of `N`" (e.g. "2 of 4"). Announced to screen readers (`aria-live="polite"`). |

### Caption

- Each image has a caption: artist, *italic title*, date. **The prototype's captions are placeholder copy**; only *Desert Plant (TAM.1460)* comes from Figma.
- The caption text is also the overlay image's `alt` text.

### Visual spec

| | Mobile (adapted) | Desktop (Figma) |
|---|---|---|
| Background | solid black (Figma: 95% black over a solid black frame) | solid black |
| Padding (top / sides / bottom) | 64 / 24 / 24px | 79 / 48 / 59px |
| Close button | 19×19px white ×, 24px from top, 24px from right | 19×19px white ×, 28px from top, 48px from right |
| Image | fills the remaining area, `object-fit: contain`, centered | same |
| Image → caption row | 24px | **32px** |
| Footer layout | counter + arrows row (right-aligned), caption below it, 24px apart | caption on the left (max 773px wide), counter + arrows on the right, 48px apart |
| Caption type | MoMA Sans Regular 16px / 24px line-height, white; title in italics | MoMA Sans Regular 18px / 24px line-height, white; title in italics |
| Counter type | MoMA Sans Bold 16px, −0.36px letter-spacing | MoMA Sans Bold 18px, −0.36px letter-spacing |
| Arrows | 72×21px (two 24px arrows, 24px apart), white | same; the first caption line is vertically centered on the arrows row |

Assets: `img/shared/icons/exit-white.svg`, `img/shared/icons/carousel-arrows-white.svg`. The arrows are one image; the prototype covers each half with a transparent button.

At Figma's 1280×911 frame, the image area is 1184×720px starting at y=79, the caption starts at y=835 and the arrows sit at x=1160–1232, y=831.

### Footer stability (important)

**The image area and the arrows/counter must stay in exactly the same position for every image, regardless of caption length.**
- The footer has a **fixed height**:
  - **Desktop:** 21px, the height of the arrows row.
  - **Mobile:** 117px, which fits the arrows row, a 24px gap and 3 lines of caption.
- Longer captions **overflow downward** into the bottom padding. They never push the image or arrows up.
- Verified in the prototype: the image area, counter and arrow positions are identical across all 4 images at 1280×911 and 390×844. Two-line captions (desktop) and three-line captions (mobile) still end above the bottom of the screen.

---

## 5. Notes

- **Text trimming:** the prototype uses CSS `text-box: trim-both cap alphabetic` (and the `cap` unit) to match Figma's cap-height alignment. Chrome and Safari support it; Firefox shows spacing a few pixels larger.
- **Image quality:** at large screen sizes the overlay may scale images up slightly. That's acceptable.
- **Image sizes:** include each image's real `width` and `height` attributes so slide widths are known before the images load.
