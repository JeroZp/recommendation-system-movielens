# Yeah, Random guides

## How to autogenerate Table of Contents

1. Install [Markdown All in One (Yu Zhang's)](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) VSCode extension
2. Do `Ctrl+Shift+P`
3. Type "Table Of Contents"
4. Bruh, just click it. The table can be automatically updated as well. Follow Extension's description for more

## How to generate the PDF's from the markdown?

I created this guide because my usual methods were not able to support all this three:
- Embed local images
- Embed images at all
- LaTeX
Or the style was just awful

---

So, yeah, the steps:

1. Install [Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced) VSCode extension
2. Install [PrinceXML](https://www.princexml.com/download/16/), I installed the Windows 64-bit installer
3. Add its bin/ folder into your PATH. Mine was here: `C:\Program Files\Prince\engine\bin`
4. Restart VSCode
5. Open your Markdown
6. Open its preview. If you cannot find the icon, do `Ctrl+K V` 
7. Click hamburguer icon located on bottom-right
8. Export > PDF (Prince)