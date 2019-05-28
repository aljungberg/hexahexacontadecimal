HHC
===

**The most compact way to encode a non-negative number in a URL.**

HHC is a compact format to express a number or binary data in a URL. It uses all characters allowed in
a URL -- the [unreserved characters](http://tools.ietf.org/html/rfc3986#section-2.3) -- making it the most concise way to express a positive integer in a URL.


| Format | Value | Length |
| ------ | --- | --- |
| Decimal | `302231454903657293676544` | 24 |
| Hex | `40000000000000000000` | 20 |
| Naive Base64[1] | `QAAAAAAAAAAAAA%3D%3D` | 20 |
| Custom Base64[2] | `BAAAAAAAAAAAAA` | 14 |
| **HHC** | **`fDpEShMz-qput`** | **13** |


Beyond being the pinacle of numeral systems for URLs, it's also a ready to go batteries included solution, unlike the most obvious alternative (base64).

HHC values sort alphabetically like the numbers they represent, so you can sort a long list of HHC values from smallest to largest without first decoding them (with one caveat, see below). 

HHC works for negative numbers too. A non-URL safe prefix is used in this case so you may need to URL quote the result. Note that `urllib.quote` [escapes the tilde character (~)](http://bugs.python.org/issue16285), which is not necessary as
of RFC3986, so if you use this on HHC data you'll waste bytes. Use the provided `hhc_url_quote` function instead. Non-negative HHC values don't need any URL quoting.

**[1]**: `hhc_url_quote(base64.urlsafe_b64encode(long_to_binary(X)))`   
**[2]**: Using the base64-like encoding scheme found [here](http://stackoverflow.com/a/561704/76900).

## Usage

    >>> print(hhc(302231454903657293676544))
    iFsGUkO.0tsxw
    >>> print(hhc_to_int('iFsGUkO.0tsxw'))
    302231454903657293676544


### Sorting

You can sort HHC encoded values in numerical order using a standard alphabetic sort. The HHC value strings need to be of equal length for this property to hold. You can left pad them with `-` just before sorting to make them so.    

The `width` parameter of the `hhc` function can be used to do this padding at encoding time. It's best not to because doing this wastes bytes and requires you to know your maximum value ahead of time.

The sortability property holds for negative numbers too.

### Negative Numbers

HHC expresses negative numbers by prefixing the number with `,` (since minus is taken). This is not a URL-safe character so if you URL escape a negative number with HHC you end up with `%2C` which takes up two extra characters. For this reason, HHC is not necessarily the shortest representation of a negative number.

## Installation

    pip install hhc

## FAQ

### Is HHC really better than base64 in URLs?

There are three parts to this.

First, if you do a naive base64 encoding, you'd rewrite your number as a binary string and then use a URL-safe base64 encoder on that binary string. You'd likely get a very long string out with padding on one end because your binary conversion gave you a 

First, you can't naively use a standard base64 encoder to efficiently encode numbers or binaries into URLs, because the common version is not URL-safe. With URL escaping on top of your encoding, you get strings much longer than what HHC would make for you.

The second part is that even if you use URL-safe base64, HHC is still better. Being radix 66 rather than radix 64, HHC is more compact.

    >>> sum(len(hhc_url_quote(num_encode_base64(n))) for n in range(10 ** 6))
    3733696
    >>> sum(len(hhc_url_quote(hhc(n))) for n in range(10 ** 8))
    3708084
    
Look at those massive savings!


### What does HHC stand for?

Hexahexaconta. In [IUPAC nomenclature](https://en.wikipedia.org/wiki/IUPAC_numerical_multiplier) (what you use to describe atoms in a molecule), 66 is hexahexaconta. 

I originally called this numeral system "hexahexacontadecimal" to make it sound like "hexadecimal" but as amusing as I found that, it was annoyingly long to type. Also, with the decimal suffix it was akin to saying "sixty-six-tenth" which didn't make sense.

### With compression, couldn't I make shorter representations?

Some data is compressible, some is not. For random data, there is no lossless compression algorithm that can compress the data, and in fact on average any such algorithm would make the data longer. HHC will still be the most compact format.

## Tests

[![Build Status](https://travis-ci.org/aljungberg/hhc.svg?branch=master)](https://travis-ci.org/aljungberg/hhc)

To run the unit tests:

    nosetests --with-doctest

## Changelog

### 3.0.1

* Backported `.` and `..` special case handling to HHC 2.

This does not affect backwards compatibility because:

1. It's disabled by default when encoding.
2. Even when enabled, the output decodes to the same value as before by older versions of HHC.

### 3.0

* Renamed to HHC.
* Sortable by default.
* Special case `.` and `..` values.

This version is **not backwards compatible**. Previously, there were two versions of HHC: normal and sortable. With HHC 3.0 the sortable version is now the "normal" and the old normal version is deprecated. The old normal version didn't offer any strong benefits, and added needless complexity.
 
To encode or decode the normal HHC 2.0, use the provided legacy methods, `hhc2` and `hhc2_to_int`.

### 2.2.1

* Fixed: pandoc accidentally required.

### 2.2

* Python 3 support (backwards compatible with Python 2.7).

### 2.1

* Fixed: `hhc(-1)` would cause an infinite loop.
* New: support for negative values.

### 2.0

* New: sortable HHC. This variant of HHC sorts the same alphabetically as numerically for equal length strings.
* Shorter, more Pythonic method names. The main function is now simply called `hhc`, styled after Python's built in `hex` function. To decode the same, `hhc_to_int` is now used.
* `import * from hexahexacontadecimal` now only imports the main functions.
* `urlquote` was renamed to `hhc_url_quote` to make it easier to differentiate from the standard library method.

### 1.0

Initial release.

## License

Free to use and modify under the terms of the BSD open source license.

## Author

Alexander Ljungberg
