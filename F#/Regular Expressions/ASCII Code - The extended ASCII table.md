# ASCII Code - The extended ASCII table

<https://www.ascii-code.com/>

**ASCII**, stands for American Standard Code for Information Interchange. It's a 7-bit character code where every single bit represents a unique character. On this webpage you will find 8 bits, 256 characters, ASCII table according to Windows-1252 (code page 1252) which is a superset of ISO 8859-1 in terms of printable characters. In the range 128 to 159 (hex 80 to 9F), ISO/IEC 8859-1 has invisible control characters, while Windows-1252 has writable characters. Windows-1252 is probably the most-used 8-bit character encoding in the world.

## ASCII control characters (character code 0-31)

The first 32 characters in the ASCII-table are unprintable control codes and are used to control peripherals such as printers.

```fsharp
| DEC| OCT | HEX| Symbol| Description                  |
| ---| ----| ---| ------| ---------------------------- |
| 0  | 000 | 00 | NUL   | Null char                    |
| 1  | 001 | 01 | SOH   | Start of Heading             |
| 2  | 002 | 02 | STX   | Start of Text                |
| 3  | 003 | 03 | ETX   | End of Text                  |
| 4  | 004 | 04 | EOT   | End of Transmission          |
| 5  | 005 | 05 | ENQ   | Enquiry                      |
| 6  | 006 | 06 | ACK   | Acknowledgment               |
| 7  | 007 | 07 | BEL   | Bell / Alert                 |
| 8  | 010 | 08 | BS    | Back Space / Backspace       |
| 9  | 011 | 09 | HT    | Horizontal Tab               |
| 10 | 012 | 0A | LF    | Line Feed / newline          |
| 11 | 013 | 0B | VT    | Vertical Tab                 |
| 12 | 014 | 0C | FF    | Form Feed                    |
| 13 | 015 | 0D | CR    | Carriage Return              |
| 14 | 016 | 0E | SO    | Shift Out / X-On             |
| 15 | 017 | 0F | SI    | Shift In / X-Off             |
| 16 | 020 | 10 | DLE   | Data Line Escape             |
| 17 | 021 | 11 | DC1   | Device Control 1 (oft. XON)  |
| 18 | 022 | 12 | DC2   | Device Control 2             |
| 19 | 023 | 13 | DC3   | Device Control 3 (oft. XOFF) |
| 20 | 024 | 14 | DC4   | Device Control 4             |
| 21 | 025 | 15 | NAK   | Negative Acknowledgement     |
| 22 | 026 | 16 | SYN   | Synchronous Idle             |
| 23 | 027 | 17 | ETB   | End of Transmit Block        |
| 24 | 030 | 18 | CAN   | Cancel                       |
| 25 | 031 | 19 | EM    | End of Medium                |
| 26 | 032 | 1A | SUB   | Substitute                   |
| 27 | 033 | 1B | ESC   | Escape                       |
| 28 | 034 | 1C | FS    | File Separator               |
| 29 | 035 | 1D | GS    | Group Separator              |
| 30 | 036 | 1E | RS    | Record Separator             |
| 31 | 037 | 1F | US    | Unit Separator               |
```

## ASCII printable characters (character code 32-127)

Codes 32-127 are common for all the different variations of the ASCII table, they are called printable characters, represent letters, digits, punctuation marks, and a few miscellaneous symbols. You will find almost every character on your keyboard. Character 127 represents the command `DEL`.

```table
| DEC | OCT | HEX| Symbol | HTML Name | Description                            |
| ----| ----| ---| ------ | --------- | -------------------------------------- |
| 32  | 040 | 20 |        |           | Space                                  |
| 33  | 041 | 21 | !      |           | Exclamation mark                       |
| 34  | 042 | 22 | "      | &quot;    | Double quotes (or speech marks)        |
| 35  | 043 | 23 | #      |           | Number                                 |
| 36  | 044 | 24 | $      |           | Dollar                                 |
| 37  | 045 | 25 | %      |           | Per cent sign                          |
| 38  | 046 | 26 | &      | &amp;     | Ampersand                              |
| 39  | 047 | 27 | '      |           | Single quote                           |
| 40  | 050 | 28 | (      |           | Open parenthesis (or open bracket)     |
| 41  | 051 | 29 | )      |           | Close parenthesis (or close bracket)   |
| 42  | 052 | 2A | *      |           | Asterisk                               |
| 43  | 053 | 2B | +      |           | Plus                                   |
| 44  | 054 | 2C | ,      |           | Comma                                  |
| 45  | 055 | 2D | -      |           | Hyphen                                 |
| 46  | 056 | 2E | .      |           | Period, dot or full stop               |
| 47  | 057 | 2F | /      |           | Slash or divide                        |
| 48  | 060 | 30 | 0      |           | Zero                                   |
| 49  | 061 | 31 | 1      |           | One                                    |
| 50  | 062 | 32 | 2      |           | Two                                    |
| 51  | 063 | 33 | 3      |           | Three                                  |
| 52  | 064 | 34 | 4      |           | Four                                   |
| 53  | 065 | 35 | 5      |           | Five                                   |
| 54  | 066 | 36 | 6      |           | Six                                    |
| 55  | 067 | 37 | 7      |           | Seven                                  |
| 56  | 070 | 38 | 8      |           | Eight                                  |
| 57  | 071 | 39 | 9      |           | Nine                                   |
| 58  | 072 | 3A | :      |           | Colon                                  |
| 59  | 073 | 3B | ;      |           | Semicolon                              |
| 60  | 074 | 3C | <      | &lt;      | Less than (or open angled bracket)     |
| 61  | 075 | 3D | =      |           | Equals                                 |
| 62  | 076 | 3E | >      | &gt;      | Greater than (or close angled bracket) |
| 63  | 077 | 3F | ?      |           | Question mark                          |
| 64  | 100 | 40 | @      |           | At symbol                              |
| 65  | 101 | 41 | A      |           | Uppercase A                            |
| 66  | 102 | 42 | B      |           | Uppercase B                            |
| 67  | 103 | 43 | C      |           | Uppercase C                            |
| 68  | 104 | 44 | D      |           | Uppercase D                            |
| 69  | 105 | 45 | E      |           | Uppercase E                            |
| 70  | 106 | 46 | F      |           | Uppercase F                            |
| 71  | 107 | 47 | G      |           | Uppercase G                            |
| 72  | 110 | 48 | H      |           | Uppercase H                            |
| 73  | 111 | 49 | I      |           | Uppercase I                            |
| 74  | 112 | 4A | J      |           | Uppercase J                            |
| 75  | 113 | 4B | K      |           | Uppercase K                            |
| 76  | 114 | 4C | L      |           | Uppercase L                            |
| 77  | 115 | 4D | M      |           | Uppercase M                            |
| 78  | 116 | 4E | N      |           | Uppercase N                            |
| 79  | 117 | 4F | O      |           | Uppercase O                            |
| 80  | 120 | 50 | P      |           | Uppercase P                            |
| 81  | 121 | 51 | Q      |           | Uppercase Q                            |
| 82  | 122 | 52 | R      |           | Uppercase R                            |
| 83  | 123 | 53 | S      |           | Uppercase S                            |
| 84  | 124 | 54 | T      |           | Uppercase T                            |
| 85  | 125 | 55 | U      |           | Uppercase U                            |
| 86  | 126 | 56 | V      |           | Uppercase V                            |
| 87  | 127 | 57 | W      |           | Uppercase W                            |
| 88  | 130 | 58 | X      |           | Uppercase X                            |
| 89  | 131 | 59 | Y      |           | Uppercase Y                            |
| 90  | 132 | 5A | Z      |           | Uppercase Z                            |
| 91  | 133 | 5B | [      |           | Opening bracket                        |
| 92  | 134 | 5C | \      |           | Backslash                              |
| 93  | 135 | 5D | ]      |           | Closing bracket                        |
| 94  | 136 | 5E | ^      |           | Caret - circumflex                     |
| 95  | 137 | 5F | _      |           | Underscore                             |
| 96  | 140 | 60 | `      |           | Grave accent - backquote - tick        |
| 97  | 141 | 61 | a      |           | Lowercase a                            |
| 98  | 142 | 62 | b      |           | Lowercase b                            |
| 99  | 143 | 63 | c      |           | Lowercase c                            |
| 100 | 144 | 64 | d      |           | Lowercase d                            |
| 101 | 145 | 65 | e      |           | Lowercase e                            |
| 102 | 146 | 66 | f      |           | Lowercase f                            |
| 103 | 147 | 67 | g      |           | Lowercase g                            |
| 104 | 150 | 68 | h      |           | Lowercase h                            |
| 105 | 151 | 69 | i      |           | Lowercase i                            |
| 106 | 152 | 6A | j      |           | Lowercase j                            |
| 107 | 153 | 6B | k      |           | Lowercase k                            |
| 108 | 154 | 6C | l      |           | Lowercase l                            |
| 109 | 155 | 6D | m      |           | Lowercase m                            |
| 110 | 156 | 6E | n      |           | Lowercase n                            |
| 111 | 157 | 6F | o      |           | Lowercase o                            |
| 112 | 160 | 70 | p      |           | Lowercase p                            |
| 113 | 161 | 71 | q      |           | Lowercase q                            |
| 114 | 162 | 72 | r      |           | Lowercase r                            |
| 115 | 163 | 73 | s      |           | Lowercase s                            |
| 116 | 164 | 74 | t      |           | Lowercase t                            |
| 117 | 165 | 75 | u      |           | Lowercase u                            |
| 118 | 166 | 76 | v      |           | Lowercase v                            |
| 119 | 167 | 77 | w      |           | Lowercase w                            |
| 120 | 170 | 78 | x      |           | Lowercase x                            |
| 121 | 171 | 79 | y      |           | Lowercase y                            |
| 122 | 172 | 7A | z      |           | Lowercase z                            |
| 123 | 173 | 7B | {      |           | Opening brace                          |
| 124 | 174 | 7C | |      |           | Vertical bar                           |
| 125 | 175 | 7D | }      |           | Closing brace                          |
| 126 | 176 | 7E | ~      |           | Equivalency sign - tilde               |
| 127 | 177 | 7F |        |           | Delete                                 |
```

32~126共96个可见打印字符。

## The extended ASCII codes (character code 128-255)

There are several different variations of the 8-bit ASCII table. The table below is according to Windows-1252 (CP-1252) which is a superset of ISO 8859-1, also called ISO Latin-1, in terms of printable characters, but differs from the IANA's ISO-8859-1 by using displayable characters rather than control characters in the 128 to 159 range. Characters that differ from ISO-8859-1 is marked by light blue color.

```table
| DEC | OCT | HEX| Symbol | HTML Name | Description                                |
| ----| ----| ---| ------ | --------- | ------------------------------------------ |
| 128 | 200 | 80 | €      | &euro;    | Euro sign                                  |
| 129 | 201 | 81 |        |           |                                            |
| 130 | 202 | 82 | ‚      | &sbquo;   | Single low-9 quotation mark                |
| 131 | 203 | 83 | ƒ      | &fnof;    | Latin small letter f with hook             |
| 132 | 204 | 84 | „      | &bdquo;   | Double low-9 quotation mark                |
| 133 | 205 | 85 | …      | &hellip;  | Horizontal ellipsis                        |
| 134 | 206 | 86 | †      | &dagger;  | Dagger                                     |
| 135 | 207 | 87 | ‡      | &Dagger;  | Double dagger                              |
| 136 | 210 | 88 | ˆ      | &circ;    | Modifier letter circumflex accent          |
| 137 | 211 | 89 | ‰      | &permil;  | Per mille sign                             |
| 138 | 212 | 8A | Š      | &Scaron;  | Latin capital letter S with caron          |
| 139 | 213 | 8B | ‹      | &lsaquo;  | Single left-pointing angle quotation       |
| 140 | 214 | 8C | Œ      | &OElig;   | Latin capital ligature OE                  |
| 141 | 215 | 8D |        |           |                                            |
| 142 | 216 | 8E | Ž      |           | Latin capital letter Z with caron          |
| 143 | 217 | 8F |        |           |                                            |
| 144 | 220 | 90 |        |           |                                            |
| 145 | 221 | 91 | ‘      | &lsquo;   | Left single quotation mark                 |
| 146 | 222 | 92 | ’      | &rsquo;   | Right single quotation mark                |
| 147 | 223 | 93 | “      | &ldquo;   | Left double quotation mark                 |
| 148 | 224 | 94 | ”      | &rdquo;   | Right double quotation mark                |
| 149 | 225 | 95 | •      | &bull;    | Bullet                                     |
| 150 | 226 | 96 | –      | &ndash;   | En dash                                    |
| 151 | 227 | 97 | —      | &mdash;   | Em dash                                    |
| 152 | 230 | 98 | ˜      | &tilde;   | Small tilde                                |
| 153 | 231 | 99 | ™      | &trade;   | Trade mark sign                            |
| 154 | 232 | 9A | š      | &scaron;  | Latin small letter S with caron            |
| 155 | 233 | 9B | ›      | &rsaquo;  | Single right-pointing angle quotation mark |
| 156 | 234 | 9C | œ      | &oelig;   | Latin small ligature oe                    |
| 157 | 235 | 9D |        |           |                                            |
| 158 | 236 | 9E | ž      |           | Latin small letter z with caron            |
| 159 | 237 | 9F | Ÿ      | &Yuml;    | Latin capital letter Y with diaeresis      |
| 160 | 240 | A0 |        | &nbsp;    | Non-breaking space                         |
| 161 | 241 | A1 | ¡      | &iexcl;   | Inverted exclamation mark                  |
| 162 | 242 | A2 | ¢      | &cent;    | Cent sign                                  |
| 163 | 243 | A3 | £      | &pound;   | Pound sign                                 |
| 164 | 244 | A4 | ¤      | &curren;  | Currency sign                              |
| 165 | 245 | A5 | ¥      | &yen;     | Yen sign                                   |
| 166 | 246 | A6 | ¦      | &brvbar;  | Pipe, Broken vertical bar                  |
| 167 | 247 | A7 | §      | &sect;    | Section sign                               |
| 168 | 250 | A8 | ¨      | &uml;     | Spacing diaeresis - umlaut                 |
| 169 | 251 | A9 | ©      | &copy;    | Copyright sign                             |
| 170 | 252 | AA | ª      | &ordf;    | Feminine ordinal indicator                 |
| 171 | 253 | AB | «      | &laquo;   | Left double angle quotes                   |
| 172 | 254 | AC | ¬      | &not;     | Not sign                                   |
| 173 | 255 | AD |        | &shy;     | Soft hyphen                                |
| 174 | 256 | AE | ®      | &reg;     | Registered trade mark sign                 |
| 175 | 257 | AF | ¯      | &macr;    | Spacing macron - overline                  |
| 176 | 260 | B0 | °      | &deg;     | Degree sign                                |
| 177 | 261 | B1 | ±      | &plusmn;  | Plus-or-minus sign                         |
| 178 | 262 | B2 | ²      | &sup2;    | Superscript two - squared                  |
| 179 | 263 | B3 | ³      | &sup3;    | Superscript three - cubed                  |
| 180 | 264 | B4 | ´      | &acute;   | Acute accent - spacing acute               |
| 181 | 265 | B5 | µ      | &micro;   | Micro sign                                 |
| 182 | 266 | B6 | ¶      | &para;    | Pilcrow sign - paragraph sign              |
| 183 | 267 | B7 | ·      | &middot;  | Middle dot - Georgian comma                |
| 184 | 270 | B8 | ¸      | &cedil;   | Spacing cedilla                            |
| 185 | 271 | B9 | ¹      | &sup1;    | Superscript one                            |
| 186 | 272 | BA | º      | &ordm;    | Masculine ordinal indicator                |
| 187 | 273 | BB | »      | &raquo;   | Right double angle quotes                  |
| 188 | 274 | BC | ¼      | &frac14;  | Fraction one quarter                       |
| 189 | 275 | BD | ½      | &frac12;  | Fraction one half                          |
| 190 | 276 | BE | ¾      | &frac34;  | Fraction three quarters                    |
| 191 | 277 | BF | ¿      | &iquest;  | Inverted question mark                     |
| 192 | 300 | C0 | À      | &Agrave;  | Latin capital letter A with grave          |
| 193 | 301 | C1 | Á      | &Aacute;  | Latin capital letter A with acute          |
| 194 | 302 | C2 | Â      | &Acirc;   | Latin capital letter A with circumflex     |
| 195 | 303 | C3 | Ã      | &Atilde;  | Latin capital letter A with tilde          |
| 196 | 304 | C4 | Ä      | &Auml;    | Latin capital letter A with diaeresis      |
| 197 | 305 | C5 | Å      | &Aring;   | Latin capital letter A with ring above     |
| 198 | 306 | C6 | Æ      | &AElig;   | Latin capital letter AE                    |
| 199 | 307 | C7 | Ç      | &Ccedil;  | Latin capital letter C with cedilla        |
| 200 | 310 | C8 | È      | &Egrave;  | Latin capital letter E with grave          |
| 201 | 311 | C9 | É      | &Eacute;  | Latin capital letter E with acute          |
| 202 | 312 | CA | Ê      | &Ecirc;   | Latin capital letter E with circumflex     |
| 203 | 313 | CB | Ë      | &Euml;    | Latin capital letter E with diaeresis      |
| 204 | 314 | CC | Ì      | &Igrave;  | Latin capital letter I with grave          |
| 205 | 315 | CD | Í      | &Iacute;  | Latin capital letter I with acute          |
| 206 | 316 | CE | Î      | &Icirc;   | Latin capital letter I with circumflex     |
| 207 | 317 | CF | Ï      | &Iuml;    | Latin capital letter I with diaeresis      |
| 208 | 320 | D0 | Ð      | &ETH;     | Latin capital letter ETH                   |
| 209 | 321 | D1 | Ñ      | &Ntilde;  | Latin capital letter N with tilde          |
| 210 | 322 | D2 | Ò      | &Ograve;  | Latin capital letter O with grave          |
| 211 | 323 | D3 | Ó      | &Oacute;  | Latin capital letter O with acute          |
| 212 | 324 | D4 | Ô      | &Ocirc;   | Latin capital letter O with circumflex     |
| 213 | 325 | D5 | Õ      | &Otilde;  | Latin capital letter O with tilde          |
| 214 | 326 | D6 | Ö      | &Ouml;    | Latin capital letter O with diaeresis      |
| 215 | 327 | D7 | ×      | &times;   | Multiplication sign                        |
| 216 | 330 | D8 | Ø      | &Oslash;  | Latin capital letter O with slash          |
| 217 | 331 | D9 | Ù      | &Ugrave;  | Latin capital letter U with grave          |
| 218 | 332 | DA | Ú      | &Uacute;  | Latin capital letter U with acute          |
| 219 | 333 | DB | Û      | &Ucirc;   | Latin capital letter U with circumflex     |
| 220 | 334 | DC | Ü      | &Uuml;    | Latin capital letter U with diaeresis      |
| 221 | 335 | DD | Ý      | &Yacute;  | Latin capital letter Y with acute          |
| 222 | 336 | DE | Þ      | &THORN;   | Latin capital letter THORN                 |
| 223 | 337 | DF | ß      | &szlig;   | Latin small letter sharp s - ess-zed       |
| 224 | 340 | E0 | à      | &agrave;  | Latin small letter a with grave            |
| 225 | 341 | E1 | á      | &aacute;  | Latin small letter a with acute            |
| 226 | 342 | E2 | â      | &acirc;   | Latin small letter a with circumflex       |
| 227 | 343 | E3 | ã      | &atilde;  | Latin small letter a with tilde            |
| 228 | 344 | E4 | ä      | &auml;    | Latin small letter a with diaeresis        |
| 229 | 345 | E5 | å      | &aring;   | Latin small letter a with ring above       |
| 230 | 346 | E6 | æ      | &aelig;   | Latin small letter ae                      |
| 231 | 347 | E7 | ç      | &ccedil;  | Latin small letter c with cedilla          |
| 232 | 350 | E8 | è      | &egrave;  | Latin small letter e with grave            |
| 233 | 351 | E9 | é      | &eacute;  | Latin small letter e with acute            |
| 234 | 352 | EA | ê      | &ecirc;   | Latin small letter e with circumflex       |
| 235 | 353 | EB | ë      | &euml;    | Latin small letter e with diaeresis        |
| 236 | 354 | EC | ì      | &igrave;  | Latin small letter i with grave            |
| 237 | 355 | ED | í      | &iacute;  | Latin small letter i with acute            |
| 238 | 356 | EE | î      | &icirc;   | Latin small letter i with circumflex       |
| 239 | 357 | EF | ï      | &iuml;    | Latin small letter i with diaeresis        |
| 240 | 360 | F0 | ð      | &eth;     | Latin small letter eth                     |
| 241 | 361 | F1 | ñ      | &ntilde;  | Latin small letter n with tilde            |
| 242 | 362 | F2 | ò      | &ograve;  | Latin small letter o with grave            |
| 243 | 363 | F3 | ó      | &oacute;  | Latin small letter o with acute            |
| 244 | 364 | F4 | ô      | &ocirc;   | Latin small letter o with circumflex       |
| 245 | 365 | F5 | õ      | &otilde;  | Latin small letter o with tilde            |
| 246 | 366 | F6 | ö      | &ouml;    | Latin small letter o with diaeresis        |
| 247 | 367 | F7 | ÷      | &divide;  | Division sign                              |
| 248 | 370 | F8 | ø      | &oslash;  | Latin small letter o with slash            |
| 249 | 371 | F9 | ù      | &ugrave;  | Latin small letter u with grave            |
| 250 | 372 | FA | ú      | &uacute;  | Latin small letter u with acute            |
| 251 | 373 | FB | û      | &ucirc;   | Latin small letter u with circumflex       |
| 252 | 374 | FC | ü      | &uuml;    | Latin small letter u with diaeresis        |
| 253 | 375 | FD | ý      | &yacute;  | Latin small letter y with acute            |
| 254 | 376 | FE | þ      | &thorn;   | Latin small letter thorn                   |
| 255 | 377 | FF | ÿ      | &yuml;    | Latin small letter y with diaeresis        |
```

# ASCII 可打印字符与控制字符

基本的 ASCII 字符集共有 128 个字符，其中有 95 个可打印字符，包括常用的字母、数字、标点符号等，另外还有 33 个控制字符。标准 ASCII 码使用 7 个二进位对字符进行编码，对应的 ISO 标准为 ISO646 标准。

## 控制字符

在ASCII码中，第0～31号及第127号(共33个)是控制字符或通讯专用字符。如控制符：LF（换行）、CR（回车）、FF（换页）、DEL（删除）、BS（退格)、BEL（振铃）等；通讯专用字符：SOH（文头）、EOT（文尾）、ACK（确认）等。

## 可打印字符

在ASCII码中，第32~126号（共95个）是可打印字符，也就是在显示器上输出能够看得见的。

# 标点符号的英语名称

． period / full stop 句号
， comma 逗号
： colon 冒号
； semicolon 分号
！ exclamation mark 惊叹号
？ question mark 问号
\- hyphen 连字符
\* asterisk 星号
' apostrophe 所有格符号，单词内部的省略
— dash 破折号
_ underscore
‘ ’ single quotation marks 单引号
“ ” double quotation marks 双引号
( ) parenthesis / round brackets 圆括号
[ ] square brackets 方括号
<> Angle brackets 尖括号
{} curly brackets / braces 大括号
《 》French quotes 法文引号；书名号
... ellipsis 省略号
¨ tandem colon 双点号
" ditto 同上
‖ parallel 双线号
／ slash / virgule / diagonal mark 斜线号
＆ ampersand / and
～ tilde / swung dash 代字号
§ section / division 分节号
→ arrow 箭号；参见号
| vertical bar 竖线
\ backslash 反斜线

---

附：部分数学符号的英文名称

＋ plus 加号；正号
－ minus 减号；负号
± plus or minus 正负号
× is multiplied by 乘号
÷ is divided by 除号
＝ is equal to 等于号
≠ is not equal to 不等于号
≡ is equivalent to 全等于号
≌ is equal to or approximately equal to 等于或约等于号
≈ is approximately equal to 约等于号
＜ less than sign 小于号
＞ more than / greater than sign大于号
≮ is not less than 不小于号
≯ is not more than 不大于号
≤ is less than or equal to 小于或等于号
≥ is more than or equal to 大于或等于号
％ per cent 百分之…
‰ per mill 千分之…
∞ infinity 无限大号
∝ varies as 与…成比例
√ (square) root 平方根
∵ since; because 因为
∴ hence 所以
∷ equals, as (proportion) 等于，成比例
∠ angle 角
⌒ semicircle 半圆
⊙ circle 圆
○ circumference 圆周
△ triangle 三角形
⊥ perpendicular to 垂直于
∪ union of 并，合集
∩ intersection of 交，通集
∫ the integral of …的积分
∑ (sigma) summation of 总和
° degree 度
′ minute 分
″ second 秒
＃ number …号
℃ Celsius system 摄氏度
＠ at 在
