# ai-image-metadata-checklist

A practical reference for where provenance, AI-labeling and privacy metadata live in common image formats, how to inspect and strip it with `exiftool`, and what no metadata tool can remove.

## Where metadata lives

| Format | EXIF | XMP (incl. IPTC `DigitalSourceType`) | C2PA / Content Credentials | Text / other |
|---|---|---|---|---|
| JPEG | APP1 `Exif` | APP1 XMP packet | APP11 segments (JUMBF boxes) | COM comment; APP13 IPTC-IIM |
| PNG | `eXIf` chunk | `iTXt` chunk `XML:com.adobe.xmp` | `caBX` chunk | `tEXt`/`iTXt`/`zTXt` (e.g. SD `parameters` prompt) |
| WebP | `EXIF` RIFF chunk | `XMP ` RIFF chunk | C2PA in RIFF `C2PA` chunk | — |
| HEIC/AVIF | `Exif` item in ISOBMFF | `mime` item (XMP) | C2PA JUMBF in a `uuid` box | — |

Key IPTC values in `Iptc4xmpExt:DigitalSourceType`: `trainedAlgorithmicMedia` (fully AI-generated), `compositeWithTrainedAlgorithmicMedia` (AI-edited/composite).

## Inspect with exiftool

```bash
exiftool -a -G1 -s image.jpg                         # everything, with group names
exiftool -a -G1 -JUMBF:all -CBOR:all -JSON:all image.jpg   # C2PA manifest (JUMBF + decoded CBOR/JSON)
exiftool -XMP-iptcExt:DigitalSourceType image.jpg
exiftool -a -gps:all -n image.jpg                    # GPS (-a, or Composite hides GPSLatitude/Longitude)
exiftool -PNG:all image.png                          # PNG text chunks (prompts)
```

## Strip with exiftool

```bash
exiftool -all= -o clean.jpg image.jpg                # all metadata (keeps pixels)
exiftool -all= -tagsfromfile @ -ColorSpaceTags -Orientation -o clean.jpg image.jpg   # keep color/orientation
exiftool -gps:all= -o nogps.jpg image.jpg            # only GPS
exiftool -JUMBF:all= -XMP-iptcExt:DigitalSourceType= -o noai.jpg image.jpg            # only C2PA + AI source type
exiftool -PNG:Parameters= -o clean.png render.png    # SD prompt chunk
exiftool -r -all= -overwrite_original ./folder       # batch, in place (back up first)
```

Tested with exiftool 13.25. Always re-inspect the output: `exiftool -a -G1 -s clean.jpg`.

PDFs: exiftool's PDF edits are appended as an incremental update and are reversible, so the old values stay in the file. Rewrite afterwards (e.g. `qpdf --linearize in.pdf out.pdf`) or use a tool that rebuilds the file.

## Pre-upload checklist

- [ ] Inspect the exact file you'll publish, not an earlier export.
- [ ] C2PA manifest present? Decide whether to keep it (it's also a provenance record you may want).
- [ ] XMP `DigitalSourceType` present?
- [ ] EXIF GPS present? Camera serial / owner name?
- [ ] PNG text chunks with prompts or workflow JSON?
- [ ] Clean, then **re-scan the cleaned file**.
- [ ] Keep the original if you might need provenance later.

## Limits (read this)

- **SynthID, TrustMark and other invisible watermarks live in the pixels.** Metadata stripping does not remove them.
- A label on an **already-posted** image can't be removed by editing your local file.
- Platforms may still detect or label content by other means; a clean file is not a guarantee.
- Re-saving through some editors re-adds metadata. Check the final export.

## Browser option

Disclosure: I'm the developer of [Remove AI Meta](https://removeaimeta.com). It does the inspect → strip → re-scan loop in the browser, without uploading the file, for JPEG, PNG, WebP, HEIC/AVIF and others, plus a separate PDF metadata page. One file is free; Pro is $9.99 one-time. It has the same limits listed above. exiftool remains the free, scriptable choice.

## License

CC0. Corrections welcome via PR.

## See also

- [Featured on: third-party pages that mention our tools](https://kibovrwa.github.io/featured-on/)
