# Slither mascot

Eight poses of the Slither snake, drawn from the same body (green gradient `snakeGrad`, scale overlay, yellow slit eyes) as the map route. He always faces the camera.

| # | File | Expression | Carousel slide |
|---|------|------------|----------------|
| 1 | pose-01-peek.svg | wide eyes, brow up, tongue out | 1 (always) |
| 2 | pose-02-wave.svg | wink, tail flick | 2 |
| 3 | pose-03-cheers.svg | half-lidded, open grin, glass up | 3 |
| 4 | pose-04-shades.svg | smirk under sunglasses | 4 |
| 5 | pose-05-swipe.svg | side-eye left, arrow | 5 |
| 6 | pose-06-loop.svg | cross-eyed, tied in a knot | 6 |
| 7 | pose-07-asleep.svg | eyes shut, drool, zzz | 7 |
| 8 | pose-08-locked.svg | scheming brows, biggest grin, padlock | 8 |

`sprite.svg` holds all eight as `<symbol id="pose-01">…</symbol>`; inline it once and draw with `<use href="#pose-03"/>`.

Carousel rule: `pose = (slideIndex % 8) + 1`. Bottom-right, 18% of the short edge, 4% inset, tilt the head 6–10°, never the body. Last slide is the map route card with no mascot. If a face sits in that corner, mirror to bottom-left.

Source of truth for edits: the Design canvas "Slither Mascot Poses".
