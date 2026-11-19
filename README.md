# Agent Skills (public subset)

Public, sanitized copies of selected skills from my private
[Hermes Agent](https://hermes-agent.nousresearch.com/) skill collection.
Generated automatically from the private source — do not edit here,
changes are overwritten on the next sync.

Personal identifiers and account details have been removed from these copies.

## Skills

### vector-score-to-musicxml

Use when transcribing vector PDF scores to MusicXML.

**Category:** creative
**Tags:** musicxml, pdf, music, transcription, musescore
**Path:** `creative/vector-score-to-musicxml/`

### musicxml-engraving-accessibility

Audit and fix MusicXML/MXL scores for DAISY accessibility.

**Category:** creative
**Tags:** musicxml, mxl, music-braille, daisy, accessibility, musescore, engraving
**Path:** `creative/musicxml-engraving-accessibility/`

### makebraille

Use when converting MusicXML or CapX scores to music Braille via dzb lesen's MakeBraille service. Vault-backed login, batch conversion, verified end-to-end plumbing.

**Category:** creative
**Tags:** braille, music, accessibility, musicxml, capx, dzb-lesen, vault, daisy
**Path:** `creative/makebraille/`

### musescore-headless-plugin

Use when a MuseScore 4 QML plugin must run headless.

**Category:** productivity
**Tags:** musescore, plugin, qml, headless, xvfb, xdotool
**Path:** `productivity/musescore-headless-plugin/`

## Install

Copy a skill directory into your Hermes skills tree, for example:

```bash
mkdir -p ~/.hermes/skills/creative
cp -r creative/vector-score-to-musicxml ~/.hermes/skills/creative/
```

Some skills reference general-purpose skills such as `pdf`,
`ocr-and-documents` and `accessible-visual-verification`; those ship with
Hermes Agent or are available separately.

## License

MIT — see [LICENSE](LICENSE).
