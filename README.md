# Update 

## Stage 0

Ecma-402 has a specification for Lookup Matching, described by LookupMatcher
(9.2.3) and BestAvailableLocale (9.2.2). It's based on the Lookup algorithm
described in RFC 4647 section 3.4. But, both Ecma-402 specification and RFC 4647
algorithm fail to perform such task in some cases. For example:

- `az-IR` maps to `az` (default for `az-Latn-AZ`), not to the correct
  `az-Arab-IR`.
- `en-Latn-GB` and `en-Latn-IN` map to `en` (default for `en-Latn-US`), not to
  the correct `en-Latn-GB`.
- `ha-CM` and `ha-SD` map to `ha` (default for `ha-Latn-NG`), not the correct
  `ha-Arab-*`.
- `kk-CN`, `kk-IR`, and `kk-MN` map to `kk` (default for `kk-Cyrl-KZ`), not the
  correct `kk-Arab-*`.
- `sr-ME`, `sr-RO`, `sr-RU`, `sr-TR` map to `sr` (default for `sr-Cyrl-RS`), not
  to the correct `sr-Latn-*`.
- `uz-AF` maps to `uz` (default for `uz-Latn-UZ`), not to the correct
  `uz-Arab-AF`.
- `zh-AU`, `zh-BN`, `zh-GB`, `zh-GF`, `zh-ID`, `zh-MO`, `zh-MY`, `zh-PA`,
  `zh-PF`, `zh-PH`, `zh-SR`, `zh-TH`, `zh-US`, and `zh-VN` map to `zh` (default
  for `zh-Hans-CN`), not to the correct `zh-Hant-*`.

### Impact

The wrong specification can be observed in today's implementations.

```javascript
// Both bugs bellow happen on latest Firefox and Chrome.

new Intl.NumberFormat("az").format(NaN); // "NaN" (as expected)
new Intl.NumberFormat("az-IR").format(NaN); // "NaN" (wrong, Arab expected)

new Intl.NumberFormat("en-IN").format(123456789); // "12,34,56,789" (as expected)
new Intl.NumberFormat("en-Latn-IN").format(123456789); // "123,456,789" (bypasses en-IN)

new Intl.DateTimeFormat("en-GB").format(new Date()); // "13/01/2015" (as expected)
new Intl.DateTimeFormat("en-Latn-GB").format(new Date()); // "1/13/2015" (bypasses en-GB)

new Intl.NumberFormat("ha-CM").format(NaN); // "NaN" (wrong, Arab expected)
new Intl.NumberFormat("ha-SD").format(NaN); // "NaN" (wrong, Arab expected)

new Intl.NumberFormat("kk-AF").format(NaN); // "NaN" (wrong, Arab expected)
new Intl.NumberFormat("kk-CN").format(NaN); // "NaN" (wrong, Arab expected)
new Intl.NumberFormat("kk-IR").format(NaN); // "NaN" (wrong, Arab expected)

new Intl.DateTimeFormat("sr", {month: "long"}).format(new Date()); // "фебруар" (as expected)
new Intl.DateTimeFormat("sr-Latn", {month: "long"}).format(new Date()); // "februar" (as expected)
new Intl.DateTimeFormat("sr-ME", {month: "long"}).format(new Date()); // "februar" (as expected)
new Intl.DateTimeFormat("sr-ME", {month: "long"}).format(new Date()); // "фебруар" (wrong, Latn expected)
new Intl.DateTimeFormat("sr-RO", {month: "long"}).format(new Date()); // "фебруар" (wrong, Latn expected)
new Intl.DateTimeFormat("sr-RU", {month: "long"}).format(new Date()); // "фебруар" (wrong, Latn expected)
new Intl.DateTimeFormat("sr-TR", {month: "long"}).format(new Date()); // "фебруар" (wrong, Latn expected)

new Intl.NumberFormat("uz").format(NaN); // "NaN" (as expected)
new Intl.NumberFormat("uz-AF").format(NaN); // "NaN" (wrong, Arab expected)

new Intl.NumberFormat("zh").format(NaN); // "NaN" (as expected)
new Intl.NumberFormat("zh-AU").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-BN").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-CN").format(NaN); // "NaN" (as expected)
new Intl.NumberFormat("zh-GB").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-GF").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-HK").format(NaN); // "非數值" (as expected)
new Intl.NumberFormat("zh-ID").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-MO").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-MO").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-MY").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-PA").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-PF").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-PH").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-SR").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-TH").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-TW").format(NaN); // "非數值" (as expected)
new Intl.NumberFormat("zh-US").format(NaN); // "NaN" (wrong, Hant expected)
new Intl.NumberFormat("zh-VN").format(NaN); // "NaN" (wrong, Hant expected)
```

### Cause

The algorithm specified is too simplistic. It basically instruct implementations
to perform subtags truncation until it finds a locale, which is wrong. Related
issues and more details can be found on:

- [Fixing Inheritance by Mark Davis][]
- [Cldrjs issue #17][]
- [Cldrjs's Lookup Matcher][]
- [Globalize issue #357][]
- [ICU issue #11404][]

[Fixing Inheritance by Mark Davis]: https://docs.google.com/document/d/1qZwEVb4kfODi2TK5f4x15FYWj5rJRijXmSIg5m6OH8s/edit
[Cldrjs issue #17]: https://github.com/rxaviers/cldrjs/issues/17
[Cldrjs's Lookup Matcher]: https://github.com/rxaviers/cldrjs/blob/master/doc/bundle_lookup_matcher.md#implementation-details
[Globalize issue #357]: https://github.com/jquery/globalize/issues/357
[ICU issue #11404]: http://bugs.icu-project.org/trac/ticket/11404

### Fix

According to Mark Davis, ["the recommended methodology for Bundle lookup is to
use Language Matching"](http://www.unicode.org/cldr/trac/ticket/8067).
Therefore, there are two options:

1. Follow the [Language Matching][] algorithm specified by the Unicode Technical
Standard #35. 
1. Follow the [quicker algorithm from cldrjs][], which produces the same result
as the Language Matching considering a score threshold of 100%.

[Language Matching]: http://www.unicode.org/reports/tr35/#LanguageMatching
[quicker algorithm from cldrjs]: https://github.com/rxaviers/cldrjs/blob/master/doc/bundle_lookup_matcher.md#implementation-details
