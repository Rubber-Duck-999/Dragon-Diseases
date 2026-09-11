# Trait icons

`wyrmrot.svg` is the source art for both illness traits (see
`common/traits/00_dragon_disease_traits.txt`, `icon = "wyrmrot"`). CK3 trait icons need to
be `.dds`, not `.svg` - convert before the mod will actually show art in-game:

1. Rasterize to PNG at 128x128 (or your target size) - any SVG tool works, e.g.:
   `rsvg-convert -w 128 -h 128 wyrmrot.svg -o wyrmrot.png`
2. Convert PNG to DDS (DXT5/BC3, with mipmaps) - Paradox's own asset browser/converter, or
   `texconv -f BC3_UNORM -m 0 wyrmrot.png`, or GIMP/Photoshop's DDS export plugin.
3. Drop the resulting `wyrmrot.dds` in this same folder, replacing the `.svg`.

`dragon_wyrmrot_resistant` (the innate immunity trait) references `icon = "wyrmrot_resistant"`
but has no art yet - needs its own SVG plus the same conversion steps once you're ready.

Until that conversion happens, the mod will fail to load the icon (or show a missing-
texture placeholder) - the trait/gameplay logic itself doesn't depend on this file.
