# Lightbox Code Annotations

## 1. The DOM

The lines at the top of lightbox.js that locate DOM elements are:

```javascript
const lb     = document.querySelector('.lightbox');
const lbImg  = lb.querySelector('.lightbox__img');
const lbCap  = lb.querySelector('.lightbox__caption');
const thumbs = document.querySelectorAll('.gallery__thumb');
```

The first three lines use querySelector which returns a single element. The fourth line uses querySelectorAll which returns a collection of all matching thumbnails. `lb.querySelector(...)` is used for lbImg and lbCap instead of `document.querySelector(...)` because it limits the search to inside the lightbox container. This is safer and more precise — it avoids accidentally selecting elements with the same class name elsewhere on the page.

---

## 2. Event Listeners

There are three `addEventListener` calls in lightbox.js:

1. **`thumb.addEventListener('click', () => openLightbox(i))`** 
On each thumbnail element, when clicked, open the lightbox at the image's index.

2. **`lb.addEventListener('click', (e) => { if (e.target === lb) closeLightbox(); })`** 
On the lightbox overlay element, It listens for a click on the dark backdrop not on the image itself and closes the lightbox.

3. **`document.addEventListener('keydown', (e) => { if (e.key === 'Escape' && state.isOpen) closeLightbox(); })`** 
On the document, listens for any keypress, and closes the lightbox if the Escape key is pressed while it is open.

The second listener uses **event delegation**  it attaches one listener to the `lb` overlay and uses `e.target === lb` to check whether the click landed directly on the overlay versus a child element inside it.

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

It has three fields: 
`isOpen` (It has a default state of closed and when you click on a photo, it opens the lightbox), 
`index` (It directs the lightbox on the positioning of the images, which image is currently shown),  
`images` (This holds the array of image sources and captions).

When a user clicks a thumbnail:
1. `state.isOpen` is set to `true` and `state.index` is set to the clicked thumbnail's position, this happens inside `openLightbox(i)`
2. `openLightbox(i)` then calls `render()`
3. `render()` reads `state.images[state.index]` to get the src and caption, sets the image `src` attribute, sets the caption text, and adds the `open` class to the lightbox element, making it visible

The mutators are `openLightbox(i)` and `closeLightbox()`, both update state and then call `render()`.

`render()` is defined at line 35 and is called from both mutators.

If state was updated but `render()` was forgotten, the DOM would not reflect the change, the lightbox would stay invisible even though `state.isOpen` was `true`, causing state and the page to silently drift apart.

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

**State + Render** — The `state` (lines 12-21) object holds all data, and `render()` (lines 36-45) is the single function that translates state into DOM updates. Every mutator calls `render()` explicitly.
```javascript
const state = { isOpen: false, index: 0, images: [...] };
function render() { ... }
```

**Event Listener** — Functions are registered to fire when user interactions occur (lines 49-56).
```javascript
thumb.addEventListener('click', () => openLightbox(i));
document.addEventListener('keydown', (e) => { ... });
```

**Delegation** — One listener on the `lb` overlay handles clicks and uses `e.target` to determine whether to act (lines 52-54).
```javascript
lb.addEventListener('click', (e) => {
  if (e.target === lb) closeLightbox();
});
```

**Module Scope** — All variables (`lb`, `lbImg`, `lbCap`, `thumbs`, `state`) are declared at the top level of the script, scoped to this file and not polluting the global window object (lines 6-9).

**Debounce/Throttle** — This pattern is not present in lightbox.js.