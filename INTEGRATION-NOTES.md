# Integration Notes

## Where did you put the lightbox trigger(s), and why?
I placed the gallery section inside the `<main>` element of my landing page, directly below the header. This placement makes sense because the page is about me as a person, and the photo gallery naturally introduces who I am visually before any other content. Visitors see my photos immediately after reading my name and bio, which makes the page feel personal and engaging.

## What content did you choose, and what does it represent?
I chose four personal travel and lifestyle photos:
- **Eiffel Tower** — taken during a trip to Paris, France
- **Louvre Pyramid** — also from Paris, reflecting my love of travel and culture
- **Poodle Dogs** — a photo of cute poodles representing my personality and love for little dogs
- **Work Photo** — representing my professional side in healthcare IT

These photos reflect the two sides of me described in my bio — a professional and a traveller.

## How did you reconcile class names?
The starter's class names (`.gallery__thumb`, `.lightbox`, `.lightbox__img`, `.lightbox__caption`) were used as-is in my HTML. I did not have conflicting class names in my existing `style.css` since my original page used generic element selectors. No renaming was necessary.

## What CSS conflicts did you have to resolve?
The main consideration was ensuring the lightbox overlay appeared above all other page content. The starter's `lightbox.css` handled this with a high `z-index`. I kept my existing `style.css` for the header, nav, and footer styling, and added `css/lightbox.css` as a separate stylesheet so the two did not interfere with each other.