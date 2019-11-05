## Unicode Category or Unicode Block: \p{}

The Unicode standard assigns each character a general category. For example, a particular character can be an uppercase letter (represented by the `Lu` category), a decimal digit (the `Nd` category), a math symbol (the `Sm`category), or a paragraph separator (the `Zl` category). Specific character sets in the Unicode standard also occupy a specific range or block of consecutive code points. For example, the basic Latin character set is found from \u0000 through \u007F, while the Arabic character set is found from \u0600 through \u06FF.

The regular expression construct

```
\p{name}
```

matches any character that belongs to a Unicode general category or named block, where *name* is the category abbreviation or named block name. For a list of category abbreviations, see the [Supported Unicode General Categories](#) section later in this topic. For a list of named blocks, see the [Supported Named Blocks](#) section later in this topic.

The following example uses the `\p{name}` construct to match both a Unicode general category (in this case, the `Pd`, or Punctuation, Dash category) and a named block (the `IsGreek` and `IsBasicLatin` named blocks).

```fsharp
let pattern = @"\b(\p{IsGreek}+(\s)?)+\p{Pd}\s(\p{IsBasicLatin}+(\s)?)+";
let input = "Κατα Μαθθαίον - The Gospel of Matthew";
let matched = Regex.IsMatch(input, pattern)
Assert.True(matched)
```

The regular expression `\b(\p{IsGreek}+(\s)?)+\p{Pd}\s(\p{IsBasicLatin}+(\s)?)+` is defined as shown in the following table.

* `\b`
  Start at a word boundary.                                    

* `\p{IsGreek}+`
  Match one or more Greek characters.                          

* `(\s)?`
  Match zero or one white-space character.                     

* `(\p{IsGreek}+(\s)?)+`
  Match the pattern of one or more Greek characters followed by zero or one white-space characters one or more times. 

* `\p{Pd}`
  Match a Punctuation, Dash character.                         

* `\s`
  Match a white-space character.                               

* `\p{IsBasicLatin}+`
  Match one or more basic Latin characters.                    

* `(\s)?`
  Match zero or one white-space character.                     

* `(\p{IsBasicLatin}+(\s)?)+`
  Match the pattern of one or more basic Latin characters followed by zero or one white-space characters one or more times. 

## Negative Unicode Category or Unicode Block: \P{}

The regular expression construct

```
\P{name}
```

matches any character that does not belong to a Unicode general category or named block, where *name* is the category abbreviation or named block name. For a list of category abbreviations, see the [Supported Unicode General Categories](#) section later in this topic. For a list of named blocks, see the [Supported Named Blocks](#) section later in this topic.

The following example uses the `\P{name}` construct to remove any currency symbols (in this case, the `Sc`, or Symbol, Currency category) from numeric strings.

```fsharp
let pattern = @"(\P{Sc})+"   
let values = [ "$164,091.78"; "£1,073,142.68"; "73¢"; "€120" ]
for value in values do
    output.WriteLine(Regex.Match(value, pattern).Value)

// The example displays the following output:
//       164,091.78
//       1,073,142.68
//       73
//       120
```

The regular expression pattern `(\P{Sc})+` matches one or more characters that are not currency symbols; it effectively strips any currency symbol from the result string.

## Supported Unicode General Categories

Unicode defines the general categories listed in the following table. For more information, see the "UCD File Format" and "General Category Values" subtopics at the [Unicode Character Database](https://www.unicode.org/reports/tr44/).

| Category | Description                                                  |
| :------- | :----------------------------------------------------------- |
| `Lu`     | Letter, Uppercase                                            |
| `Ll`     | Letter, Lowercase                                            |
| `Lt`     | Letter, Titlecase                                            |
| `Lm`     | Letter, Modifier                                             |
| `Lo`     | Letter, Other                                                |
| `L`      | All letter characters. This includes the `Lu`, `Ll`, `Lt`, `Lm`, and `Lo` characters. |
| `Mn`     | Mark, Nonspacing                                             |
| `Mc`     | Mark, Spacing Combining                                      |
| `Me`     | Mark, Enclosing                                              |
| `M`      | All diacritic marks. This includes the `Mn`, `Mc`, and `Me` categories. |
| `Nd`     | Number, Decimal Digit                                        |
| `Nl`     | Number, Letter                                               |
| `No`     | Number, Other                                                |
| `N`      | All numbers. This includes the `Nd`, `Nl`, and `No` categories. |
| `Pc`     | Punctuation, Connector                                       |
| `Pd`     | Punctuation, Dash                                            |
| `Ps`     | Punctuation, Open                                            |
| `Pe`     | Punctuation, Close                                           |
| `Pi`     | Punctuation, Initial quote (may behave like `Ps` or `Pe` depending on usage) |
| `Pf`     | Punctuation, Final quote (may behave like `Ps` or `Pe` depending on usage) |
| `Po`     | Punctuation, Other                                           |
| `P`      | All punctuation characters. This includes the `Pc`, `Pd`, `Ps`, `Pe`, `Pi`, `Pf`, and `Po` categories. |
| `Sm`     | Symbol, Math                                                 |
| `Sc`     | Symbol, Currency                                             |
| `Sk`     | Symbol, Modifier                                             |
| `So`     | Symbol, Other                                                |
| `S`      | All symbols. This includes the `Sm`, `Sc`, `Sk`, and `So` categories. |
| `Zs`     | Separator, Space                                             |
| `Zl`     | Separator, Line                                              |
| `Zp`     | Separator, Paragraph                                         |
| `Z`      | All separator characters. This includes the `Zs`, `Zl`, and `Zp` categories. |
| `Cc`     | Other, Control                                               |
| `Cf`     | Other, Format                                                |
| `Cs`     | Other, Surrogate                                             |
| `Co`     | Other, Private Use                                           |
| `Cn`     | Other, Not Assigned (no characters have this property)       |
| `C`      | All control characters. This includes the `Cc`, `Cf`, `Cs`, `Co`, and `Cn` categories. |

You can determine the Unicode category of any particular character by passing that character to the [GetUnicodeCategory](https://docs.microsoft.com/en-us/dotnet/api/system.char.getunicodecategory) method. The following example uses the [GetUnicodeCategory](https://docs.microsoft.com/en-us/dotnet/api/system.char.getunicodecategory) method to determine the category of each element in an array that contains selected Latin characters.

```fsharp
let chars = [ 'a'; 'X'; '8'; ','; ' '; '\u0009'; '!' ]
for ch in chars do
    output.WriteLine("{0}: {1}", JsonConvert.SerializeObject(ch.ToString()), Char.GetUnicodeCategory(ch));
//"a": LowercaseLetter
//"X": UppercaseLetter
//"8": DecimalDigitNumber
//",": OtherPunctuation
//" ": SpaceSeparator
//"\t": Control
//"!": OtherPunctuation
```

## Supported Named Blocks

.NET provides the named blocks listed in the following table. The set of supported named blocks is based on Unicode 4.0 and Perl 5.6.

```F#
[<Fact>]
member this.namedblock() =
let table = [
    "\u0000\u007F","IsBasicLatin"
    "\u0080\u00FF","IsLatin-1Supplement"
    "\u0100\u017F","IsLatinExtended-A"
    "\u0180\u024F","IsLatinExtended-B"
    "\u0250\u02AF","IsIPAExtensions"
    "\u02B0\u02FF","IsSpacingModifierLetters"
    "\u0300\u036F","IsCombiningDiacriticalMarks"
    "\u0370\u03FF","IsGreek"
    "\u0370\u03FF","IsGreekandCoptic"
    "\u0400\u04FF","IsCyrillic"
    "\u0500\u052F","IsCyrillicSupplement"
    "\u0530\u058F","IsArmenian"
    "\u0590\u05FF","IsHebrew"
    "\u0600\u06FF","IsArabic"
    "\u0700\u074F","IsSyriac"
    "\u0780\u07BF","IsThaana"
    "\u0900\u097F","IsDevanagari"
    "\u0980\u09FF","IsBengali"
    "\u0A00\u0A7F","IsGurmukhi"
    "\u0A80\u0AFF","IsGujarati"
    "\u0B00\u0B7F","IsOriya"
    "\u0B80\u0BFF","IsTamil"
    "\u0C00\u0C7F","IsTelugu"
    "\u0C80\u0CFF","IsKannada"
    "\u0D00\u0D7F","IsMalayalam"
    "\u0D80\u0DFF","IsSinhala"
    "\u0E00\u0E7F","IsThai"
    "\u0E80\u0EFF","IsLao"
    "\u0F00\u0FFF","IsTibetan"
    "\u1000\u109F","IsMyanmar"
    "\u10A0\u10FF","IsGeorgian"
    "\u1100\u11FF","IsHangulJamo"
    "\u1200\u137F","IsEthiopic"
    "\u13A0\u13FF","IsCherokee"
    "\u1400\u167F","IsUnifiedCanadianAboriginalSyllabics"
    "\u1680\u169F","IsOgham"
    "\u16A0\u16FF","IsRunic"
    "\u1700\u171F","IsTagalog"
    "\u1720\u173F","IsHanunoo"
    "\u1740\u175F","IsBuhid"
    "\u1760\u177F","IsTagbanwa"
    "\u1780\u17FF","IsKhmer"
    "\u1800\u18AF","IsMongolian"
    "\u1900\u194F","IsLimbu"
    "\u1950\u197F","IsTaiLe"
    "\u19E0\u19FF","IsKhmerSymbols"
    "\u1D00\u1D7F","IsPhoneticExtensions"
    "\u1E00\u1EFF","IsLatinExtendedAdditional"
    "\u1F00\u1FFF","IsGreekExtended"
    "\u2000\u206F","IsGeneralPunctuation"
    "\u2070\u209F","IsSuperscriptsandSubscripts"
    "\u20A0\u20CF","IsCurrencySymbols"
    "\u20D0\u20FF","IsCombiningDiacriticalMarksforSymbols"
    "\u20D0\u20FF","IsCombiningMarksforSymbols"
    "\u2100\u214F","IsLetterlikeSymbols"
    "\u2150\u218F","IsNumberForms"
    "\u2190\u21FF","IsArrows"
    "\u2200\u22FF","IsMathematicalOperators"
    "\u2300\u23FF","IsMiscellaneousTechnical"
    "\u2400\u243F","IsControlPictures"
    "\u2440\u245F","IsOpticalCharacterRecognition"
    "\u2460\u24FF","IsEnclosedAlphanumerics"
    "\u2500\u257F","IsBoxDrawing"
    "\u2580\u259F","IsBlockElements"
    "\u25A0\u25FF","IsGeometricShapes"
    "\u2600\u26FF","IsMiscellaneousSymbols"
    "\u2700\u27BF","IsDingbats"
    "\u27C0\u27EF","IsMiscellaneousMathematicalSymbols-A"
    "\u27F0\u27FF","IsSupplementalArrows-A"
    "\u2800\u28FF","IsBraillePatterns"
    "\u2900\u297F","IsSupplementalArrows-B"
    "\u2980\u29FF","IsMiscellaneousMathematicalSymbols-B"
    "\u2A00\u2AFF","IsSupplementalMathematicalOperators"
    "\u2B00\u2BFF","IsMiscellaneousSymbolsandArrows"
    "\u2E80\u2EFF","IsCJKRadicalsSupplement"
    "\u2F00\u2FDF","IsKangxiRadicals"
    "\u2FF0\u2FFF","IsIdeographicDescriptionCharacters"
    "\u3000\u303F","IsCJKSymbolsandPunctuation"
    "\u3040\u309F","IsHiragana"
    "\u30A0\u30FF","IsKatakana"
    "\u3100\u312F","IsBopomofo"
    "\u3130\u318F","IsHangulCompatibilityJamo"
    "\u3190\u319F","IsKanbun"
    "\u31A0\u31BF","IsBopomofoExtended"
    "\u31F0\u31FF","IsKatakanaPhoneticExtensions"
    "\u3200\u32FF","IsEnclosedCJKLettersandMonths"
    "\u3300\u33FF","IsCJKCompatibility"
    "\u3400\u4DBF","IsCJKUnifiedIdeographsExtensionA"
    "\u4DC0\u4DFF","IsYijingHexagramSymbols"
    "\u4E00\u9FFF","IsCJKUnifiedIdeographs"
    "\uA000\uA48F","IsYiSyllables"
    "\uA490\uA4CF","IsYiRadicals"
    "\uAC00\uD7AF","IsHangulSyllables"
    "\uD800\uDB7F","IsHighSurrogates"
    "\uDB80\uDBFF","IsHighPrivateUseSurrogates"
    "\uDC00\uDFFF","IsLowSurrogates"
    "\uE000\uF8FF","IsPrivateUse"
    "\uE000\uF8FF","IsPrivateUseArea"
    "\uF900\uFAFF","IsCJKCompatibilityIdeographs"
    "\uFB00\uFB4F","IsAlphabeticPresentationForms"
    "\uFB50\uFDFF","IsArabicPresentationForms-A"
    "\uFE00\uFE0F","IsVariationSelectors"
    "\uFE20\uFE2F","IsCombiningHalfMarks"
    "\uFE30\uFE4F","IsCJKCompatibilityForms"
    "\uFE50\uFE6F","IsSmallFormVariants"
    "\uFE70\uFEFF","IsArabicPresentationForms-B"
    "\uFF00\uFFEF","IsHalfwidthandFullwidthForms"
    "\uFFF0\uFFFF","IsSpecials"
]
let lines = [
    for (inp,pat) in table do
        let rgx = Regex (sprintf @"\p{%s}" pat)
        let m = rgx.Matches(inp)
        let count = m.Count
        yield (sprintf "%s %s %d" inp pat count)
    ]
File.WriteAllLines(@"d:\namedblock.txt",lines)
```

