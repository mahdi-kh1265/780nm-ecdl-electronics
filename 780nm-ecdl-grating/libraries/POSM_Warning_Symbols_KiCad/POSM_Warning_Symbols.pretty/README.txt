POSM Warning Symbols for KiCad
================================

Files:
- Caution_Hot_Surface_14x14mm_Silk.kicad_mod
- Caution_Hot_Surface_20x18mm_Silk.kicad_mod

Both are graphics-only F.SilkS footprints with no pads and are excluded from BOM and position files.

Import:
1. KiCad -> Preferences -> Manage Footprint Libraries.
2. Add the folder POSM_Warning_Symbols.pretty as a project-specific library.
3. In PCB Editor, Place Footprint and search for "Caution_Hot_Surface".

Use the larger version when board area allows; the smaller one is better near the OPA462 region.
