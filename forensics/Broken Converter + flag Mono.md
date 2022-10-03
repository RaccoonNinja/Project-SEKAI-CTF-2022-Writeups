## Skills involved: XPS analysis, Font analysis

It's a very interesting challenge for me. Especially I don't have any experience of the 2 file types.

## Solution:

We're given an [XPS](https://en.wikipedia.org/wiki/Open_XML_Paper_Specification) file. It can be opened in both Windows and Ubuntu, but working in Windows is slightly more convenient since it's developed by Microsoft.

There are some ways to learn what has happened within the document:
- XPS files are zip files, we can decompress the content
- Online XPS -> PDF converters will allow selection of text in non-native platforms
- Work on Windows

No matter which method we use, we will learn that the actual *text* is different from the displayed *glyph* (fancy term for the character symbol graphic in a font that is displayed), thanks to a weird font which we can find by decompressing the archive.

The font is in [ODTTF](https://en.wikipedia.org/wiki/ODTTF) format, although it's obfuscated the deofuscation is very simple. There are [many resources on how to do it programmatically](https://gist.github.com/dungsaga/ab8d2379bb566c9925b27df3bc82ca8b).

The first challenge is solved by inspecting the font file in [FontForge](https://fontforge.org/en-US/), alternatively it can be solved by installing the font and making educated guesses.

![image](https://user-images.githubusercontent.com/114584910/193539545-8b86c832-c632-486f-8d9b-2cb3c4f5835f.png)

The second challenge hints at `stylistic sets`(https://glyphsapp.com/learn/stylistic-sets). If you already have FontForge you can access `View>Show ATT>...>GSUB`:

![image](https://user-images.githubusercontent.com/114584910/193541604-2d1f2b04-a32d-4eca-98c2-e51d2a1883c3.png)

The flag can be discovered by looking all substitution rules.

P.S. There are also very awesome solutions on the discord server from using [FontDrop](https://fontdrop.info/) to using CSS or M$ Word to render the stylistic sets.
