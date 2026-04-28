# Lightbox Code Annotations

## 1. The DOM

The lines at the top of lightbox.js that locate DOM elements are:

```javascript
const lb     = document.querySelector('.lightbox');
const lbImg  = lb.querySelector('.lightbox__img');
const lbCap  = lb.querySelector('.lightbox__caption');
const thumbs = document.querySelectorAll('.gallery__thumb');
```

`document.querySelector('.lightbox')` returns a **single element** — the first match.
`lb.querySelector('.lightbox__img')` and `lb.querySelector('.lightbox__caption')` also return **single elements**.
`document.querySelectorAll('.gallery__thumb')` returns a **collection** (NodeList) of all matching elements.

The code uses `lb.querySelector(...)` for nested elements instead of `document.querySelector(...)` because it scopes the search to inside the lightbox element only. This is safer and more precise — it avoids accidentally selecting elements with the same class name elsewhere on the page.

---

## 2. Event Listeners

There are three `addEventListener` calls in lightbox.js:

1. **`thumb.addEventListener('click', () => openLightbox(i))`** — on each thumbnail element, listens for a click, and opens the lightbox at the clicked image's index.

2. **`lb.addEventListener('click', (e) => { if (e.target === lb) closeLightbox(); })`** — on the lightbox overlay element, listens for a click, and closes the lightbox only if the click was directly on the overlay background (not on the image).

3. **`document.addEventListener('keydown', (e) => { if (e.key === 'Escape' && state.isOpen) closeLightbox(); })`** — on the document, listens for any keypress, and closes the lightbox if the Escape key is pressed while it is open.

The second listener uses **event delegation** — it attaches one listener to the `lb` overlay and uses `e.target === lb` to check whether the click landed directly on the overlay versus a child element inside it.

---

## 3. State and the Render Pattern

The state object is:

```javascript
const state = {
  isOpen: false,
  index: 0,
  images: [...]
};
```

It has three fields: `isOpen` (whether the lightbox is visible), `index` (which image is currently shown), and `images` (the array of image sources and captions).

When a user clicks a thumbnail:
1. `state.isOpen` is set to `true` and `state.index` is set to the clicked thumbnail's position — this happens inside `openLightbox(i)`
2. `openLightbox(i)` then calls `render()`
3. `render()` reads `state.images[state.index]` to get the src and caption, sets the image `src` attribute, sets the caption text, and adds the `open` class to the lightbox element — making it visible

The mutators are `openLightbox(i)` and `closeLightbox()` — both update state and then call `render()`.

`render()` is defined at line 35 and is called from both mutators.

If state was updated but `render()` was forgotten, the DOM would not reflect the change — the lightbox would stay invisible even though `state.isOpen` was `true`, causing state and the page to silently drift apart.

---

## 4. Security

The two XSS-safe lines in `render()` are:

```javascript
lbImg.setAttribute('src', src);
lbCap.textContent = caption;
```

If either line used `innerHTML` instead, an attacker could inject malicious HTML or JavaScript into the page through a crafted caption or image src value. For example, a caption containing `<script>stealCookies()</script>` would execute as code if inserted with `innerHTML`. Using `textContent` treats the value as plain text, so any HTML tags are displayed literally rather than parsed and executed.

The attack class this prevents is **XSS** (Cross-Site Scripting).

---

## 5. Patterns

**State + Render** — The `state` object holds all data, and `render()` (lines 35-43) is the single function that translates state into DOM updates. Every mutator calls `render()` explicitly. lines 11-19
```javascript
const state = { isOpen: false, index: 0, images: [...] };
function render() { ... }
```

**Event Listener** — Functions are registered to fire when user interactions occur (lines 46-48).
```javascript
thumb.addEventListener('click', () => openLightbox(i));
document.addEventListener('keydown', (e) => { ... });
```

**Delegation** — One listener on the `lb` overlay handles clicks and uses `e.target` to determine whether to act (lines 50-52).
```javascript
lb.addEventListener('click', (e) => {
  if (e.target === lb) closeLightbox();
});
```

**Module Scope** — All variables (`lb`, `lbImg`, `lbCap`, `thumbs`, `state`) are declared at the top level of the script, scoped to this file and not polluting the global window object (lines 6-9).

**Debounce/Throttle** — This pattern is not present in lightbox.js.