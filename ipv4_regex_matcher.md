# IPv4 address matcher via Regex
Regular expressions are not foreign to developers as string pattern matching is such a common use case regardless of programming languages. 
In this post a Regex pattern to match dot-decimal represented IPv4 addresses is discussed and presented. 

IPv4 uses 32-bit addresses which limits the address space to 4,294,967,296 addresses.
IPv4 address can be represented in any notation expressing a 32-bit integer number. They are most often written
in the dot-decimal notation, which consists of 4 octets expressed individually in decimal numbers and separated by periods.
Out-fashioned it may be, its [wikipage](https://en.wikipedia.org/wiki/IPv4) still presents this Internet Protocol in a very succinct way. 

## Problem statement
Given an input string, find all substrings that match an IPv4 address in dot-decimal notation.

### In Scope
Dot-decimal format should be matched.
E.g. `192.178.4.235`, `127.0.0.1` and `0.0.0.0`

### Out of Scope
Other representations like below should be ignored.
* Decimal integer format: `3221226219`
* Hexadecimal integer format: `0xC00002EB`
* Dotted hex format: `0xC0.0x00.0x02.0xEB`
* Octal byte value format: `0300.0000.0002.0353`
* Invalid IP address: `009.018.099.098`

### Example
* Input:&nbsp;&nbsp;&nbsp;&nbsp;`this is a list of intersected IP addresses 127.0.0.1.0.2. and 620.0.0.2 is not IP`
* Output: `127.0.0.1`, `27.0.0.1`, `7.0.0.1`, `0.0.1.0`, `0.1.0.2`, `20.0.0.2`, `0.0.0.2`

## Approach
The most intuitive way to deal with this problem is to use Regex patterns. Since the dot-decimal notation of an IPv4 address consists of four 8-bit decimal integers (i.e. 0 - 255) separated by three dots, the Regex pattern can be constructed accordingly. 

Personally I find this [regexr.com](https://regexr.com/) website super useful when constructing a pattern. When the mouse cursor is hovered over a metacharacter, a tooltip is displayed explaining on how it works. 

The byte section of an IPv4 address, which is an 8-bit integer, can be matched by bellow pattern, with a decreasing order. 

`25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9]`

To match a full IP address, 4 repetitions of above number pattern with a literal dot seem to manage the mischief, but the tricky part is that the last section doesn't have a dot. To acheive that we need a bit more involved Regex knowledge. The [lookaround zero-length assertions](https://www.regular-expressions.info/lookaround.html) can be used here to ignore the dot of the last section. In particular, a **negative lookbehind** is needed. A good example is found [here](https://www.rexegg.com/regex-disambiguation.html#negative-lookbehind) and this [article](https://www.rexegg.com/regex-lookarounds.html) helps clear the mystery of lookaround assertions. 

`\b((25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])((?<!\.)\b|\.)){4}`

In the pattern above, `(?<!\.)\b` is the **ngative lookbehind** assertion that declares that whatever precedes the current position is not a dot. If the assertion succeeds, the Regex engine matches a word boundary with `\b`. This helps clear the match on the dot of the last section. A subtle feature of lookaround assertions to note is that they are zero-length, meaning by the end of the matching, the Regex engine hasn't moved on the string. 

This pattern matches an IP address with word boundaries, as in below string. 

`198.2.3.4`.4.5.6.2378.`9.4.1.1`.`192.1.8.255`.909.8.1.1...9.8.1.456...

If we remove the preceding word boundary, the matches are like below. 

`198.2.3.4`.4.5.6.23`78.9.4.1`.`1.192.1.8`.255.90`9.8.1.1`...9.8.1.456...

Up until here it seems quite promising, but this pattern fails to match the IP address `9.8.1.45` at the tail of the string, because there is a word boundary used in the negative lookbehind assertion. While this pattern may do a good job in checking whether a passed in string is a valid IP address, it can't be used to detect all possible IPs in an input string because the word boundary match at the end ignores IPs with concatenated numbers. 

This problem can be easily resolved, but in a rather brute-force way as below. 

`((25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])\.){3}(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])`

We try to match 3 repetitions with dot and 1 more without dot. This is not so bad actually as it no longer requires a negative lookbehind assertion. The matches are revealed as below. 

`198.2.3.4`.`4.5.6.237` `8.9.4.1`.`1.192.1.8`.255.90`9.8.1.1`...`9.8.1.45`6...

This looks pretty awesome. One thing to notice is that the number pattern matches eagerly since it is in decreasing order, e.g. instead of `4.5.6.2` it matches `4.5.6.237`. 

Additionally, the non-capturing group assertion `(?: re)` can be used to avoid unwanted sub-group matches so as to boost performance. 

`((?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])\.){3}(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])`

To obtain all possible IPs from the given string, a program can be coded to apply this Regex pattern from every digit position in the string and backtrack a bit in the found match to extract sub-matches. 

Running a DFA-compiled regular expression against a string involves a time complexity of O(n), and during a partial match the actual time complexity would be O(n\*m) where m is the size of the pattern as the Regex engine needs to check not only from the first character but also subsequent ones until it finds a match. Hence such a program plainly coded without much consideration on this would result in a worst case scenario time complexity of O(m\*n^2) where there is no IP match in the string. This issue can be easily mitigated by breaking the iteration when no match can ever be found. See the implementation in below link. 

[Source code in Java](https://github.com/MaRuifeng/DummyLovesAlgorithms/blob/master/src/string/IPv4AddressMatcher.java)

[Unit tests](https://github.com/MaRuifeng/DummyLovesAlgorithms/blob/master/src/string/IPv4AddressMatcherTest.java)

## Ownership
Ruifeng Ma (mrfflyer@gmail.com), 2019-Mar-30 night.

## Ghost links (a.k.a won't read unless doing PhD)
[Regular Expression Matching Can Be Simple And Fast](https://swtch.com/~rsc/regexp/regexp1.html)








