# Blender with Sidecar ( macOS + iPad )

- To attempt to adjust Blender for Sidecar (macOS + iPad).
- [Demonstration (Youtube)](https://youtu.be/9nO9vem3Smw)

## Tested Environment

- MacBook Pro 2019
- iPad Pro 2018
- Blender 2.8x

## WARRING

- It is makeshift solution.
- NO WARRANTY.
- WIP

## How to build blender from sorce oode

[link](https://wiki.blender.org/wiki/Building_Blender/Mac)

## My Adjustment

### 1. Stroke-out is high(full) pressure. Sculpt & Paint mode

- modified source code. [diff](diff/diff_stroke.txt)
- Commit id: 1f6c3699a836d485ed37f443cd0fcd19e978dbb6
- [Refarence; developer.blender.org](https://developer.blender.org/T62565)

### 2. Adjust pen pressure

- [A Blender's Add-on "Brush Adjuster for Apple Pencil" I developed](https://shikitari.github.io/blender_sidecar/baap/)
- Please purchase. Please support me.

### Increase pinch-in(Zooming) sensibility

- modified source code. [diff](diff/diff_stroke.txt)
- Commit id: 1f6c3699a836d485ed37f443cd0fcd19e978dbb6

### 4. Using s-Key can't sample color, due to no hovering with Apple Pencil

#### method1; modified source code

- [diff](diff/diff_stroke.txt)

#### method2; An Add-on for Blender

- [A Blender's Add-on "Brush Adjuster for Apple Pencil" I developed](https://shikitari.github.io/blender_sidecar/baap/)
- Please purchase. Please support me.
