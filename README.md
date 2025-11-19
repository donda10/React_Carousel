# Ex05 Image Carousel
## Date: 19/11/25

## AIM
To create a Image Carousel using React 

## ALGORITHM
### STEP 1 Initial Setup:
Input: A list of images to display in the carousel.

Output: A component displaying the images with navigation controls (e.g., next/previous buttons).

### Step 2 State Management:
Use a state variable (currentIndex) to track the index of the current image displayed.

The carousel starts with the first image, so initialize currentIndex to 0.

### Step 3 Navigation Controls:
Next Image: When the "Next" button is clicked, increment currentIndex.

If currentIndex is at the end of the image list (last image), loop back to the first image using modulo:
currentIndex = (currentIndex + 1) % images.length;

Previous Image: When the "Previous" button is clicked, decrement currentIndex.

If currentIndex is at the beginning (first image), loop back to the last image:
currentIndex = (currentIndex - 1 + images.length) % images.length;

### Step 4 Displaying the Image:
The currentIndex determines which image is displayed.

Using the currentIndex, display the corresponding image from the images list.

### Step 5 Auto-Rotation:
Set an interval to automatically change the image after a set amount of time (e.g., 3 seconds).

Use setInterval to call the nextImage() function at regular intervals.

Clean up the interval when the component unmounts using clearInterval to prevent memory leaks.

## PROGRAM
```
import React, { useState, useEffect, useRef } from "react";

// SimpleImageCarousel.jsx
// Pure React + plain CSS (no Tailwind, no external icon libs)
// Props: images (array of URLs), autoPlay (bool), interval (ms)

export default function SimpleImageCarousel({
  images = [
    "https://images.unsplash.com/photo-1506765515384-028b60a970df?w=1200&q=80&auto=format&fit=crop",
    "https://images.unsplash.com/photo-1491553895911-0055eca6402d?w=1200&q=80&auto=format&fit=crop",
    "https://images.unsplash.com/photo-1503023345310-bd7c1de61c7d?w=1200&q=80&auto=format&fit=crop",
  ],
  autoPlay = true,
  interval = 3500,
}) {
  const [index, setIndex] = useState(0);
  const length = images.length;
  const containerRef = useRef(null);

  // autoplay
  useEffect(() => {
    if (!autoPlay || length <= 1) return;
    const id = setInterval(() => setIndex((i) => (i + 1) % length), interval);
    return () => clearInterval(id);
  }, [autoPlay, interval, length]);

  const prev = () => setIndex((i) => (i - 1 + length) % length);
  const next = () => setIndex((i) => (i + 1) % length);
  const goTo = (i) => setIndex(i);

  // basic touch support (swipe)
  useEffect(() => {
    const el = containerRef.current;
    if (!el) return;
    let startX = 0;
    let moved = false;

    function onTouchStart(e) {
      startX = e.touches[0].clientX;
      moved = false;
    }
    function onTouchMove(e) {
      const dx = e.touches[0].clientX - startX;
      if (Math.abs(dx) > 10) moved = true;
    }
    function onTouchEnd(e) {
      const endX = e.changedTouches[0].clientX;
      const dx = endX - startX;
      if (!moved) return;
      if (dx < -30) next();
      else if (dx > 30) prev();
    }

    el.addEventListener("touchstart", onTouchStart, { passive: true });
    el.addEventListener("touchmove", onTouchMove, { passive: true });
    el.addEventListener("touchend", onTouchEnd);

    return () => {
      el.removeEventListener("touchstart", onTouchStart);
      el.removeEventListener("touchmove", onTouchMove);
      el.removeEventListener("touchend", onTouchEnd);
    };
  }, [length]);

  return (
    <div className="sic-root" style={{ maxWidth: 900, margin: "0 auto" }}>
      <style>{`
        .sic-viewport { position: relative; overflow: hidden; border-radius: 12px; box-shadow: 0 6px 18px rgba(0,0,0,0.12); }
        .sic-track { display: flex; transition: transform 480ms ease; }
        .sic-slide { flex: 0 0 100%; user-select: none; }
        .sic-slide img { width: 100%; height: 360px; object-fit: cover; display: block; }
        .sic-button { position: absolute; top: 50%; transform: translateY(-50%); background: rgba(255,255,255,0.9); border: none; padding: 8px 12px; border-radius: 6px; cursor: pointer; box-shadow: 0 2px 6px rgba(0,0,0,0.1); }
        .sic-button:active { transform: translateY(-50%) scale(0.98); }
        .sic-prev { left: 12px; }
        .sic-next { right: 12px; }
        .sic-dots { display:flex; gap:8px; justify-content:center; margin-top:12px; }
        .sic-dot { width:10px; height:10px; border-radius:50%; border:none; background:#cfcfcf; padding:0; cursor:pointer; }
        .sic-dot[aria-pressed="true"] { background:#333; transform: scale(1.2); }
        @media (max-width:600px) {
          .sic-slide img { height: 220px; }
          .sic-button { padding:6px 8px; }
        }
      `}</style>

      <div className="sic-viewport" ref={containerRef}>
        <div
          className="sic-track"
          style={{ transform: `translateX(${-index * 100}%)` }}
        >
          {images.map((src, i) => (
            <div className="sic-slide" key={i}>
              <img src={src} alt={`slide-${i + 1}`} loading="lazy" />
            </div>
          ))}
        </div>

        <button
          className="sic-button sic-prev"
          onClick={prev}
          aria-label="Previous slide"
        >
          {/* simple chevron */}
          <span aria-hidden>◀</span>
        </button>

        <button
          className="sic-button sic-next"
          onClick={next}
          aria-label="Next slide"
        >
          <span aria-hidden>▶</span>
        </button>
      </div>

      <div className="sic-dots" role="tablist" aria-label="Slide dots">
        {images.map((_, i) => (
          <button
            key={i}
            className="sic-dot"
            aria-pressed={i === index}
            aria-label={`Go to slide ${i + 1}`}
            onClick={() => goTo(i)}
          />
        ))}
      </div>
    </div>
  );
}

/*
Usage (pure React):

1) Copy this file to src/components/SimpleImageCarousel.jsx
2) Import and use in App.js:

import React from 'react';
import SimpleImageCarousel from './components/SimpleImageCarousel';

function App(){
  const imgs = [ 'https://...','https://...','https://...' ];
  return (
    <div style={{ padding: 16 }}>
      <SimpleImageCarousel images={imgs} autoPlay={true} interval={4000} />
    </div>
  );
}

export default App;

No external CSS framework or icon library required. */


```

## OUTPUT


<img width="1622" height="786" alt="image" src="https://github.com/user-attachments/assets/b54797c0-5e2c-45fc-ac96-c8e1478aa862" />


## RESULT
The program for creating Image Carousel using React is executed successfully.
