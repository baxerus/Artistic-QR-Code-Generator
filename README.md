# Artistic QR Code Generator

A standalone, single-file HTML tool for creating highly customized, artistic QR codes by embedding user-defined pixel patterns. Instead of simply patching a artwork onto a QR code, this tool uses the QR codes grid as painting canvas. It than searches through thousands of URL hash fragment variations using a multi-worker optimization engine to find the QR encodings that naturally align with your art, so that the least amount of decoding errors will be introduced by the painted pattern.

## Core Value

Your painted pattern is sacred and never changes. The tool ensures your art survives at 100% by modifying the underlying data (via hash fragments) to match the visual design, rather than forcing the design to match the data.

## Features

- **Standalone Architecture:** The entire app runs perfectly offline via the `file://` protocol (of course it can also be used served by a webserver). It only uses vanilla HTML5/CSS3/ES6 and integrates everything into a single HTML file.
- **Interactive Canvas:** Paint three-state pixels (black/white/unset) onto the QR grid with an interactive legend and drag painting.
- **Hash Optimization Loop:** Multi-worker parallel search finds the best URL hash fragments that minimize pixel conflicts with your design.
- **Accurate Error Measurement:** Decodes and measures actual Reed-Solomon (RS) error correction counts to guarantee scan reliability.
- **Robust Tooling:** Includes a "Move Pattern" tool for repositioning art and full rotation controls (manual and random 90° rotations).
- **Multiple Exports:** Download your generated QR codes in clean PNG or single-path SVG formats.

## External Libraries Integrated

To maintain the single-file, zero-dependency constraint, the application directly inlines the following external open-source libraries:

- **[qrcodejs](https://github.com/davidshimjs/qrcodejs):** Used for the underlying QR code generation. The library was specifically patched to enforce explicit version selection and its core generation logic was extracted so it could run safely inside DOM-less Web Workers.
- **[jsQR](https://github.com/cozmo/jsQR):** Used for QR decoding and validating the scan reliability of the generated codes. The library was modified from its original form to expose the internal Reed-Solomon error correction metrics, allowing the tool to sort results based on true structural integrity rather than surface-level pixel differences.

## Usage

Simply click [here](https://baxerus.github.io/Artistic-QR-Code-Generator/) since it is also a **GitHub page** 🎉.  
Or open the `index.html` file in any modern web browser (Chrome, Firefox, Safari).  
No server, build step, or command-line installation is required.

1. **Enter your base URL:** Provide the destination link.
2. **Paint your design:** Lock specific modules as black or white to draw your artwork.
3. **Generate:** The app generates and evaluates variants, previewing the top 5 results live based on RS metrics and pixel diffs.
4. **Export:** Inspect your favorite result and download it instantly.

## Disclaimer

This project was mainly created to try out a developing workflow using AI (at first Claude Code, later OpenCode with all sorts of models) and to gather learnings from it. I made learnings and I would start over completly different now 😉. So the project was successful from this side.  
But IMHO it was also successful as a tool itself. I built something that I wanted to try out for a long time already. And it is working. I actually spend way to much time to make it even nicely usable. That's why I put it on GitHub. The use case is IMHO very niche, but if you need it, it is there.  
So that said, there are tradeoffs with such a test ballon. E.g. when against better knowledge, this thing is only very briefly tested. Neither bay hand or automatic. I did not test multiple browsers on multiple OS. There are no automatic test or anything (BTW one of the things I really want to do different with my next AI test project, fingers crossed 🤞, that I will really do 😉). I assume using this website **on a mobile/touch** device will be **next to impossible**. But hey, the use case is so niche, people who need _this_ will be sitting in front of a (probably fancy) Mac with a big screen anyway, isn't it...  
So long story short, use this tool and make some nice QR codes. But be aware, that maybe not all edge cases work perfectly 👍  
(still if you really use it and find something, create an issue! I'm curious what to come)
