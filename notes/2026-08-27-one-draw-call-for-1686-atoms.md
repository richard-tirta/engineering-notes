# One Draw Call for 1,686 Atoms

Molecular viewers have been on my curiosity list for a while — a whole genre of interface where the data is a physical object, the users are scientists, and the rendering has to hold up under counts that would flatten an ordinary dashboard. I have never worked in that domain. Rather than read about it, I built a small lab: load a real protein structure, render it in WebGL, and wire a 2D panel, a table, and the 3D scene to one shared selection. It is live at [molviewer-lab.vercel.app](https://molviewer-lab.vercel.app) and the code is [on GitHub](https://github.com/richard-tirta/molviewer-lab).

A note on authorship, because it matters for what follows. I did this with Claude, and I used it the way I would use a patient colleague who knows the domain: it wrote the scaffolding — the three.js scene setup, raycast picking, the Mol* embed, the 2D structure panel, the tabs — and it explained the things I did not know, from what a PDB file is to why a draw call costs what it does. The three pieces I hand-typed, and the ones this note is about, are the PDB parser, the instanced atom cloud, and the selection store. I chose that split on purpose. I did not want to explain a viewer I had not built the load-bearing parts of, and I did not want the understanding to be something I had only read.

## The file format is older than whitespace

The structure is 1HSG: HIV-1 protease with the drug indinavir bound in it. It comes as a PDB file, a text format from the 1970s, and the first thing to learn is that it is **fixed-width**. Every field lives at a column range — serial in columns 7–11, atom name in 13–16, x/y/z in 31–38, 39–46, 47–54 — and the instinct to `split(' ')` will silently corrupt it the moment two fields run together, which they do.

So the parser is a list of `slice` calls with the spec's 1-indexed columns converted to JavaScript's 0-indexed ones, followed by checks: reject any record that is not exactly `ATOM  ` or `HETATM`, and throw if any coordinate or serial comes back `NaN`. Old files sometimes leave the element column blank; the fallback is the first letter of the atom name. Water is `HETATM` but not interesting, so the ligand is "hetero atoms that are not `HOH`." The result for 1HSG is 1,686 atoms.

## Draw calls, and an analogy from Flash

My mental model for 3D on the web came from Flash, and it turned out to map cleanly: a **Scene** is the stage, a **Geometry** is a library symbol, a **Mesh** is an instance placed on stage. That got me through the setup. It also set up the trap.

The obvious way to render 1,686 atoms is 1,686 meshes — one sphere each, placed and colored. That works, and it is 1,686 draw calls per frame: the GPU is told "draw a sphere" 1,686 separate times. It is fine until it is not, and molecular viewers routinely handle a hundred times this.

The fix is one `InstancedMesh`: one sphere geometry, one material, and a count. Per atom you write a transform matrix — position from the file, scale from the element's radius — and a color into two big typed arrays on the mesh. Then the whole cloud is one draw call. The GPU gets the sphere once and a list of 1,686 places to put it.

The line that caught me was `needsUpdate = true`. Writing into those arrays changes nothing on screen, because three.js uploaded the buffer to the GPU once and does not watch it. The flag says "push this array again on the next frame," and it clears itself after. Forget it and your data is correct, your arrays are correct, and nothing moves.

## The store outside React

Hover an atom in 3D and its row in the table should light; click a row and the sphere should grow. Three surfaces, one selection. The pattern is an external store consumed through `useSyncExternalStore`, and it is four parts:

1. One variable holding the current state object.
2. A `Set` of listeners; `subscribe` adds one and returns the remover.
3. `emit()`, which calls every listener with no payload.
4. Setters that bail if nothing changed, then **replace** the state object and emit.

`getSnapshot` just returns the variable. The thing I had to actually understand is why the setter replaces the object instead of mutating it: React compares snapshots by reference. Mutating in place leaves the reference unchanged, so React sees "same" even after `emit()` fires, and the UI never updates. The new object is not a style preference. It is the change signal. The same reason explains the loop you get if `getSnapshot` builds a fresh object every call — now everything looks changed, every render.

This is the mechanism under every modern store library. It was worth writing once by hand to stop treating it as magic.

## What I actually learned

Less about molecules than I expected, and more about the three places a viewer like this can go wrong: parsing a format that punishes assumptions, rendering a count that punishes naive scene graphs, and sharing state across surfaces that do not know about each other. Those are not biotech problems. They are the same problems as a trading dashboard or a map, wearing a lab coat.
