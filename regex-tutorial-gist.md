# Regex Tutorial

In simplest terms, regex is a (somewhat) standardized language for building pattern-matching expressions. The world of regex is wide, however; for more detailed explanations that you'll find here and to research more obscure expressions, take a look at one of the many online resources. To find out about other flavors of regex, visit the [Regular Expression Engine Comparison Chart](https://gist.github.com/CMCDragonkai/6c933f4a7d713ef712145c5eb94a1816#).

Regex is similar to a wildcard search in Microsoft Word, but it's much more powerful. A number of "flavors" of regex are used in different industries; for example, Adobe InDesign offers "GREP" (Global Regular Expression Parser) searches. Each flavor has a slightly different syntax and capabilities. This tutorial generally describes the major regex search-pattern operators used by the ECMA flavor of regex, which is the flavor we use in Javascript.

While it is an extremely powerful too, regex does have one serious limitation to keep in mind: it cannot include formatting in its searches. If we need to find italic "*Foo*" followed by nonitalic "[space]Bar" but no other combinations of "Foo" and "[space]Bar," regex is not our best tool. That's not to say regex powerless here. Let’s look at InDesign again for an example, which layers format searching on top of its GREP search function. In InDesign, we could find "*Foo*"[space]Bar by first searching for italic "*Foo*" and surrounding it with plain-text tags like "\<i\>” and “\</i\>." With these tags in place we can build a regex expression that will find "\<i\>Foo\</i\> Bar" and transform it in some way. After we are finished searching, of course, we'd want to undo our tag insertion. In coding, where regex is used more for validation than for find/replace functions, this limitation is not particularly important, but in other applications, it could require creative approaches.

## Summary

Let's explore regex in more detail with an example expression: `[\.\!\?,;:~\_][’”»~F]*[ ~S]*[\p{L}]{1,2}\K[ ]`

The regex expression above is an example taken from my own InDesign toolbox. I use it to bind short words to longer words that follow them in specific situations, such as when a short word begins a sentence. Binding the short words to their neighbors avoids poor line breaks like "...went to the store. He / and I bought..." or “...made from blocks of ice, an / igloo...” The regex binds "He" to "and" and “an” to “igloo” so that they always appear on a line together.

In plain text, our regex says: Find one period, exclamation point, question mark, comma, semicolon, colon, inverted exclamation point, or em dash followed by zero or more closing single or double quotation marks, closing double guillemots, or footnote numbers (the `~F` is specific to InDesign's GREP) followed by one space or nonbreaking space followed by one or two characters that are designated to be letters in the Unicode standard followed by a space. (The `\K` is an instruction to the regex engine and is explained below.)

In the following sections, let’s analyze the elements of this regular expression and discover what each part does. Two things to note: First, the brackets around the space at the end of expression are unnecessary, but I've included them here so that it's clear that the expression ends with a space. Second, the expression is using InDesign's GREP flavor of regex. The `~F` is an example that differentiates GREP from straight regex.

## Table of Contents

- [Anchors](#anchors)
- [Quantifiers](#quantifiers)
- [Grouping Constructs](#grouping-constructs)
- [Bracket Expressions](#bracket-expressions)
- [Character Classes](#character-classes)
- [The OR Operator](#the-or-operator)
- [Flags](#flags)
- [Character Escapes](#character-escapes)

## Regex Components

### Anchors

Regex anchors include ^, $, \b, \B, [primary string](?=[secondary string]), [primary string](?![secondary string]), (?<=[secondary string])[primary string], and (?<![secondary string])[primary string].

The `^` anchor means "The next character(s) starts a paragraph." If I wanted to find every tab at the start of a paragraph, my expression would be `^\t`. Contrastingly, `$` indicates that the expression ends a paragraph. `\b` and `\B` indicate a word boundary or not a word boundary, respectively. The last four anchors can be tricky. They are "positive lookahead," "negative lookahead," "positive lookbehind," and "negative lookbehind."

"Positive lookahead" searches for a primary string that is followed immediately by secondary string. A search expression like `The (?=Hobbit)` will only return the word "The" when it's followed by "Hobbit" and will ignore all other occurrences. This is useful because any manipulation that I perform will affect only the text of the primary string. In a find/replace function using this expression to apply italic, only "The" will be italicized. The three remaining lookahead/lookbehind anchors perform similar functions, each looking for the presence or absence of a secondary string either after the primary string (lookahead) or before the primary string (lookbehind). The `\K` in our example is similar to the lookbehind anchor; it finds a primary string preceded by a secondary string but omits the secondary string from the returned match. Writing our expressions using lookbehind rather than `\K` would give us this expression: `(?<=[\.\!\?,;:~\_][’”»~F]*[ ~S]*[\p{L}]{1,2})[ ]`.

### Quantifiers

The regex quantifiers are \*, +, ?, {n}, {n,}, {n,m}

The quantifiers mean "match 0 or more occurrences," "match 1 or more," "match exactly 0 or 1", "match *n*," match *n* or more,” and "match *n* to *m*," respectively. The "?" quantifier also serves a second function. When it follows one of the other quantifiers, it makes the other qualifier nongreedy. That is, the qualifier will return a string up to the first occurrence of the terminating character in the expression. For example, searching the text "abacadaeaf" for `a.+a` will return "abacadaea"—it's greedy and returns a string from the first "a" to the very last "a" it can find. If we add the `?` quantifier after the `+`, though, the expression becomes nongreedy; `a.+?a` will return "aba"; it stops looking when it finds the first a" after an initial "a."

In our example expression, we use the "zero-or-more" quantifier twice: after `[’”»\F]*` to make the presence of a closing quotation mark, guillemot, or footnote reference optional and after `[ ~S]*` to make the space or nonbreaking space optional. We want to return any matches both with and without those substrings.

### Grouping Constructs

The regex grouping constructs include (), (?:), and (?[name]). The lookahead and lookbehind assertions are also sometimes considered to be grouping constructs. (See Anchors above for an explanation.) These constructs include the capturing group (everything in the group should be treated as a logical unit), the noncapturing group (the string in the parens is to be excluded from whatever the search returns), and the named capturing group (the returned string is saved to the variable "matches.groups.[name]").

In our example string, we don't have any grouping constructs, but if we needed to isolate the beginning of the expression from the terminating space—perhaps to insert something between them—we would do this `([\.\!\?,;:~\_][’”»~F]*[ ~S]*[\p{L}]{1,2}\K)([ ])`. To reference the groups in the returned string, we would use "$1" and "$2" for the first group and second group, respectively. For example, to insert the string "nobreak" between our groups, our replacement string would be "$1nobreak$2."

### Bracket Expressions

A bracket expression in regex is a method for restricting the pool of search characters in all or part of a search expression. For example, the bracketed expression `[a-z]` restricts the pool of characters being sought to the lowercase letters "a" through "z"; `[a,z]` restricts the pool to the letters "a" and "z." To search for "anything except,” precede the string with a carat: `[^abc]` will find anything except "a," "b," and "c." (Note that `[a-c]` is identical to `[abc]`.)

In our example string, we use bracketed expressions five times. The first expression, `[\.\!\?,;:~\_]`, defines a pool of characters, one of which must be first in the returned string. (I explain the backslashes in Character Escapes below.) The second expression, `[’”»~F]`, says that the next character may be (“may be” rather than “must be” is indicated by the `*` quantifier that follows the expression in the example) a closing single- or double quotation mark, a closing double guillemot, or a footnote reference marker. The third bracketed expression, `[ ~S]`, indicates that a space or nonbreaking space may follow any of the characters in the preceding two bracketed expressions. The fourth expression is `[\p{L}]`. It searches for any letters in any Unicode-documented language (vs. numbers, puncuation marks, emojis, etc.); the `{1,2}` quantifier following the expression limits the length of the returned string to either 1 or 2 characters—no more and no less. Finally, the fifth expression, `[ ]` indicates that our search pattern must end with a space. Without the terminal space, our search pattern would find words of any length, though it would return only the first two characters of words with three or more letters.

### Character Classes

The strings of characters in the bracketed expressions above are called character classes. Regex provides some predefined classes like `\d` (any digit), `\s` (any white space), and `\w` (any word), among others. (POSIX expressions are also predefined classes, but we won’t cover POSIX here.) An important character class to remember is a single period—`.`—which means "any character." (See Character Escapes below to learn why our `.` is preceded by a `\`.) Other code classes, some of which are only one character long, include `\s` for a white-space character, `\t` for a tab, `\r` for a hard return (especially important because in most if not all flavors of regex, searches stop at a hard return by default), and `\u{[hexadecimal string]}` for a Unicode character. I sometimes use an intimidating looking string to find all Arabic characters in a document: `[\u{0600}-\u{06FF}\u{0750}-\u{077F}\u{08A0}-\u{08FF}\u{FB50}-\u{FDFF}\u{FE70}-\u{FEFF}\u{10E60}-\u{10E7F}\u{1EE00}-\u{1EEFF}]+`. The character classes above are the predefined classes we're most likely to use, but there are others. And of course, we can create our own classes as we have done in our search string by enclosing the class in brackets—for example, `[\.\!\?,;:~\_]`.

### The OR Operator

Regex provides an "or" operator in the form of a pipe ("|"). To find either "cat" or "dog”, for example, we type `[cat|dog]` in our search pattern. When used in conjunctions with other regex expressions, it can provide us with the flexiblity to minimize the number of regex searches we need to accomplish our goal.

### Flags
Again, our search pattern does not include any of them, but regex provides flags that allow us to further restrict our search. Generally, these restrictions can also be imposed using the expressions we've already discussed, but in some cases, flags will provide a faster or better solution. Some of the most-used flags are `i` meaning the search is case-insensitive (the regex search engine will not differentiate between "A" and "a") and `g` to find all matches, not just the first occurrences. (In some text-handling software, we can perform a search, and the software will highlight all occurrences of the search string that appear in the document; the `g` flag performs the same function.) We also have `m` for multiline searches (see `\r` in character classes above for why this might be needed) and `s`, which allows the "anything" character `.` to include new-line characters. If we wanted to find only upper- and lowercase letters used in English (no accents) instead of any letter in any language, we could modify our example script and use the "i" flag like this: `[\.\!\?,;:~\_][’”»~F]*[ ~S]*[a-z]i{1,2}\K[ ]`.

### Character Escapes

As we have seen, we use plain ASCII text to build regex expressions. What happens, though, if we want to search for something that the regex search engine interprets as a code? The period is a perfect example. Regex understands `.` to mean “anything,” but what if we want to find an actual period, for example in a URL? Regex provides us a way to “escape” the "code" characters so that we can do just that. To search for a character that regex normally interprets as a code, precede the character with a backslash. To search for a period, `\.` is the correct search expression. We see just this string in the first bracketed expression in our example search.

Be aware that the use of the backslash is not always intuitive; we have to consider context. For example, in some flavors of regex, we might expect to use "\<" to search for a less-than sign. After all, `<` is a code used in lookback assertions. However, in InDesign's GREP language, `\<` is the code for "at the beginning of a word"; it won't return a less-than sign. In this case, a simple `<` is required. (Don't confuse `\<` meaning “at the beginning of a word” with `^` meaning "at the beginning of a paragraph."

One final note about escaping characters: When using regex expression literals in Javascript code, the expression must be enclosed in slashes. This effectively escapes the entire search string from being interpreted by the Javascript engine. To use our example string as an expression literal in a Javascript function, we would type `/[\.\!\?,;:~\_][’”»~F]*[ ~S]*[\p{L}]{1,2}\K[ ]/`.

## Author

Matthew Williams is a typesetter, book designer, and aspiring web developer. His crystal ball tells him that he won't be able to retire just working on print materials, so he is preparing to make a move into electronic publication of websites and EPUBs. You can see some of his current and past coding projects at https://github.com/MatthewWilliamsCMH.