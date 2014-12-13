encoding.js
===========

[![Build Status](https://travis-ci.org/polygonplanet/encoding.js.svg?branch=master)](https://travis-ci.org/polygonplanet/encoding.js)

Converts character encoding in JavaScript.  

**[README(Japanese)](https://github.com/polygonplanet/encoding.js/blob/master/README_ja.md)**

### Installation

#### In Browser:

```html
<script src="encoding.js"></script>
```

or

```html
<script src="encoding.min.js"></script>
```

Object **Encoding** will defined in the global scope.

Conversion and detection for the Array (like Array object).  

#### In Node.js:

encoding.js is published by module name of `encoding-japanese` in npm.

```bash
npm install encoding-japanese
```

```javascript
var encoding = require('encoding-japanese');
```

Each methods are also available for the *Buffer* in Node.js.

#### Convert character encoding (convert):

* {_Array.<number>_} Encoding.**convert** ( data, to\_encoding [, from\_encoding ] )  
  Converts character encoding.  
  @param {_Array.<number>_|_TypedArray_|_Buffer_} _data_ Target data.  
  @param {_(string|Object)_} _to\_encoding_ The encoding name of conversion destination.  
  @param {_(string|Array.<string>)=_} [_from\_encoding_] The encoding name of source or 'AUTO'.  
  @return {_Array_}  Return the converted array.


```javascript
// Convert character encoding to Shift_JIS from UTF-8.
var utf8Array = new Uint8Array(...) or [...] or Array(...) or Buffer(...);
var sjisArray = Encoding.convert(utf8Array, 'SJIS', 'UTF8');

// Convert character encoding by automatic detection (AUTO detect).
var sjisArray = Encoding.convert(utf8Array, 'SJIS');
// or  
var sjisArray = Encoding.convert(utf8Array, 'SJIS', 'AUTO');

// Detect the character encoding.
// The return value be one of the "Available Encodings" below.
var detected = Encoding.detect(utf8Array);
if (detected === 'UTF8') {
  console.log('Encoding is UTF-8');
}
```

##### Available Encodings:

* '**UTF32**'   (detect only)
* '**UTF16**'
* '**UTF16BE**'
* '**UTF16LE**'
* '**BINARY**'  (detect only)
* '**ASCII**'   (detect only)
* '**JIS**'
* '**UTF8**'
* '**EUCJP**'
* '**SJIS**'
* '**UNICODE**' (JavaScript Unicode Array)

Note: UNICODE is an array that has a value of String.charCodeAt() in JavaScript.
　　(Each value in the array possibly has a number of more than 256.)


##### Specify the Object argument

```javascript
var sjisArray = Encoding.convert(utf8Array, {
  to: 'SJIS', // to_encoding
  from: 'UTF8' // from_encoding
});
```

Readability goes up by passing an object to the second argument.

##### Specify BOM in UTF-16

It's can add the UTF16 BOM with specify the bom option on convert.

```javascript
var utf16Array = Encoding.convert(utf8Array, {
  to: 'UTF16', // to_encoding
  from: 'UTF8', // from_encoding
  bom: true // With BOM
});
```

The byte order of UTF16 is big-endian by default.

Specify the 'LE' in bom options if you want to convert as little-endian.  

```javascript
var utf16leArray = Encoding.convert(utf8Array, {
  to: 'UTF16', // to_encoding
  from: 'UTF8', // from_encoding
  bom: 'LE' // With BOM (little-endian)
});
```

Convert with specifying the UTF16LE or UTF16BE if BOM is not required.

```javascript
var utf16beArray = Encoding.convert(utf8Array, {
  to: 'UTF16BE',
  from: 'UTF8'
});
```

Note: UTF16, UTF16BE and UTF16LE is not JavaScript internal encoding, that is an byte array.

#### Detect character encoding (detect):

* {_Array_} Encoding.**detect** ( data [, encodings ] )  
  Detect character encoding.  
  @param {_Array.<number>_|_TypedArray_} _data_ Target data  
  @param {_(string|Array.<string>)_} [_encodings_] The encoding name that to specify the detection.  
  @return {_string|boolean_} Return the detected character encoding, or false.


```javascript
// Detect character encoding by automatic. (AUTO detect).
var detected = Encoding.detect(utf8Array);
if (detected === 'UTF8') {
  console.log('Encoding is UTF-8');
}

// Detect character encoding by specific encoding name.
var isSJIS = Encoding.detect(sjisArray, 'SJIS');
if (isSJIS) {
  console.log('Encoding is SJIS');
}
```


##### URL Encode/Decode:

* {_Array_} Encoding.**urlEncode** ( data )  
  URL(percent) encode.  
  @param {_Array.<number>_|_TypedArray_} _data_ Target data.  
  @return {_string_}  Return the encoded string.

* {_Array_} Encoding.**urlDecode** ( string )  
  URL(percent) decode.  
  @param {_string_} _string_ Target data.  
  @return {_Array.<number>_} Return the decoded array.

```javascript
// URL encode to an array that has character code.
var sjisArray = [
  130, 177, 130, 241, 130, 201, 130, 191, 130, 205, 129,
  65, 130, 217, 130, 176, 129, 153, 130, 210, 130, 230
];

var encoded = Encoding.urlEncode(sjisArray);
console.log(encoded);
// output:
// '%82%B1%82%F1%82%C9%82%BF%82%CD%81A%82%D9%82%B0%81%99%82%D2%82%E6'

var decoded = Encoding.urlDecode(encoded);
console.log(decoded);
// output: [
//   130, 177, 130, 241, 130, 201, 130, 191, 130, 205, 129,
//    65, 130, 217, 130, 176, 129, 153, 130, 210, 130, 230
// ]
```

#### Example:

##### Example using the XMLHttpRequest and Typed arrays (Uint8Array):

In this sample, reads the text file written in Shift_JIS as binary data.  
And displays string that is converted to Unicode by Encoding.convert.

```javascript
var req = new XMLHttpRequest();
req.open('GET', '/my-shift_jis.txt', true);
req.responseType = 'arraybuffer';

req.onload = function (event) {
  var buffer = req.response;
  if (buffer) {
    // Shift_JIS Array
    var sjisArray = new Uint8Array(buffer);

    // Convert encoding to UNICODE (JavaScript Unicode Array).
    var unicodeArray = Encoding.convert(sjisArray, {
      to: 'UNICODE',
      from: 'SJIS'
    });

    // Join to string.
    var unicodeString = Encoding.codeToString(unicodeArray);
    console.log(unicodeString);
  }
};

req.send(null);
```

##### Example of the character encoding conversion:

```javascript
var eucjpArray = [
  164, 179, 164, 243, 164, 203, 164, 193, 164, 207, 161,
  162, 164, 219, 164, 178, 161, 249, 164, 212, 164, 232
];

var utf8Array = Encoding.convert(eucjpArray, {
  to: 'UTF8',
  from: 'EUCJP'
});
console.log( utf8Array );
// output: [
//   227, 129, 147, 227, 130, 147, 227, 129, 171,
//   227, 129, 161, 227, 129, 175, 227, 128, 129,
//   227, 129, 187, 227, 129, 146, 226, 152, 134,
//   227, 129, 180, 227, 130, 136
// ]
//   => 'こんにちは、ほげ☆ぴよ'
```

##### Example of convert a character code by automatic detection (Auto detect):

```javascript
var sjisArray = [
  130, 177, 130, 241, 130, 201, 130, 191, 130, 205, 129,
   65, 130, 217, 130, 176, 129, 153, 130, 210, 130, 230
];
var unicodeArray = Encoding.convert(sjisArray, {
  to: 'UNICODE',
  from: 'AUTO'
});
// codeToString is a utility method that Joins a character code array to string.
console.log( Encoding.codeToString(unicodeArray) );
// output: 'こんにちは、ほげ☆ぴよ'
```

### Utilities

* {_string_} Encoding.**codeToString** ( {_Array.<number>_|_TypedArray_} data )  
  Joins a character code array to string.

* {_Array.<number>_} Encoding.**stringToCode** ( {_string_} string )  
  Splits string to an array of character codes.


### Demo

[Test for character encoding conversion (Demo)](http://polygonplanet.github.io/encoding.js/tests/encoding-test.html)

### License

MIT


