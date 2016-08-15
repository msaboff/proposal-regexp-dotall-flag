# ECMAScript proposal: `s` (`singleline`) flag for regular expressions

## Status

This proposal is in stage 0 of [the TC39 process](https://tc39.github.io/process-document/).

## Motivation

In regular expression patterns, the dot `.` matches a single character, regardless of what character it is. In ECMAScript, there are two exceptions to this:

* `.` doesn’t match astral characters. Setting the `u` (`unicode`) flag fixes that.
* `.` doesn’t match [line terminator characters](https://tc39.github.io/ecma262/#prod-LineTerminator).

ECMAScript recognizes the following line terminator characters:

* U+000A LINE FEED (LF) (`\n`)
* U+000D CARRIAGE RETURN (CR) (`\r`)
* U+2028 LINE SEPARATOR
* U+2029 PARAGRAPH SEPARATOR

These are the same characters [matched by `\p{Line_Break=Mandatory_Break}`](https://github.com/mathiasbynens/es-regexp-unicode-property-escapes). However, there are more characters that, depending on the use case, [could be considered as newline characters](http://www.unicode.org/reports/tr14/):

* U+000B VERTICAL TAB (`\v`)
* U+000C FORM FEED (`\f`)
* U+0085 NEXT LINE (`\p{Line_Break=Next_Line}`)

This makes the current behavior of `.` problematic:

* By design, it excludes _some_ newline characters, but not all of them, which often does not match the developer’s use case.
* It’s commonly used to match _any_ character, which it doesn’t do.

The former issue is addressed by the [proposal to add Unicode property escapes to regular expressions](https://github.com/mathiasbynens/es-regexp-unicode-property-escapes) (through `\p{Line_Break=…}`).
The proposal you’re looking at right now addresses the latter issue.

Developers wishing to truly match *any* character, including these line terminator characters, cannot use `.`:

```js
/foo.bar/.test('foo\nbar');
// → false
```

Instead, developers have to resort to cryptic workarounds like `[\s\S]` or `[^]`:

```js
/foo[^]bar/.test('foo\nbar');
// → true
```

Since the need to match any character is quite common, other regular expression engines support a mode in which `.` matches any character, including line terminators.

* Engines that support constants to enable regular expression flags implement `DOTALL` or `SINGLELINE` modifiers to opt-in to singleline mode.
    * [Java](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html#DOTALL) supports `Pattern.DOTALL`.
    * [C# and VB](https://msdn.microsoft.com/en-us/library/system.text.regularexpressions.regexoptions.aspx) support `RegexOptions.Singleline`.
    * Python supports both `re.DOTALL` and [`re.S`](https://docs.python.org/2/library/re.html#re.S).
* Engines that support embedded flag expressions implement `(?s)` to opt-in to single-line mode.
    * [Java](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html#DOTALL)
    * [C# and VB](https://msdn.microsoft.com/en-us/library/yd1hzczs.aspx)
* Engines that support regular expression flags implement the flag `s`.
    * [Perl](http://perldoc.perl.org/perlre.html#*s*)
    * [PHP](https://secure.php.net/manual/en/reference.pcre.pattern.modifiers.php#s)

The most common naming pattern seems to be `singleline` & `s`.

## Proposed solution

We propose the addition of a new `s` flag (short for `singleline`) for ECMAScript regular expressions that makes `.` match any character, including line terminators.

```js
/foo.bar/s.test('foo\nbar');
// → true
```

## High-level API

```js
const re = /foo.bar/s; // Or, `const re = new RegExp('foo.bar', 's');`.`
re.test('foo\nbar');
// → true
re.singleline
// → true
re.flags
// → 's'
```

### FAQ

#### What about backwards compatibility?

The meaning of existing regular expression patterns isn’t affected by this proposal since the new `s` flag is required to opt-in to the new behavior.

#### How does `singleline` mode affect `multiline` mode?

Both modes are independent and can be combined. `multiline` mode only affects anchors, and `singleline` mode only affects `.`.

When both the `s` (`singleline`) and `m` (`multiline`) flags are set, `.` matches any character while still allowing `^` and `$` to match, respectively, just after and just before line terminators within the string.

## Specification

* [Ecmarkup source](https://github.com/mathiasbynens/es-regexp-singleline-flag/blob/master/spec.html)
* [HTML version](https://mathiasbynens.github.io/es-regexp-singleline-flag/)

## Implementations

* [regexpu (transpiler)](https://github.com/mathiasbynens/regexpu) with the `{ singlelineFlag: true }` option enabled