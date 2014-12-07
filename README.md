encoding.js
===========

[![Build Status](https://travis-ci.org/polygonplanet/encoding.js.svg?branch=master)](https://travis-ci.org/polygonplanet/encoding.js)

Converts character encoding in JavaScript.  
JavaScript で文字コード変換をします

### Installation

#### In Browser:

```html
<script src="encoding.js"></script>
```

or

```html
<script src="encoding.min.js"></script>
```

**Encoding** というオブジェクトがグローバルに定義されます  

配列に対して変換または判別します  


#### In Node.js:

`encoding-japanese` というモジュール名になっています

```bash
npm install encoding-japanese
```

```javascript
var encoding = require('encoding-japanese');
```

encoding.js の各メソッドは Node.js の Buffer に対しても使えます

#### 文字コード変換 (convert):

* {_Array.<number>_} Encoding.**convert** ( data, to\_encoding [, from\_encoding ] )  
  文字コードを変換します  
  @param {_Array.<number>_|_TypedArray_|_Buffer_} _data_ 対象のデータ  
  @param {_(string|Object)_} _to\_encoding_ 変換先の文字コード  
  @param {_(string|Array.<string>)=_} [_from\_encoding_] 変換元の文字コード or 'AUTO'  
  @return {_Array_}  変換した配列が返ります


```javascript
// UTF-8のデータをShift_JISに変換
var utf8Array = new Uint8Array(...) or [...] or Array(...) or Buffer(...);
var sjisArray = Encoding.convert(utf8Array, 'SJIS', 'UTF8');

// 自動判別で変換 (AUTO detect)
var sjisArray = Encoding.convert(utf8Array, 'SJIS');
// or  
var sjisArray = Encoding.convert(utf8Array, 'SJIS', 'AUTO');

// 文字コード判別 (戻り値は下の"Available Encodings"のいずれか)
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

※ UNICODE は JavaScript の String.charCodeAt() の値を持つ配列です  
　　(配列の各値は 256 を超える数値になり得ます)


##### 引数に Object を指定する

```javascript
var sjisArray = Encoding.convert(utf8Array, {
  to: 'SJIS', // to_encoding
  from: 'UTF8' // from_encoding
});
```

第二引数にオブジェクトで渡すことで可読性が上がります

##### UTF16 に BOM をつける

UTF16 に変換する際に bom オプションを指定すると BOM が付加できます

```javascript
var utf16Array = Encoding.convert(utf8Array, {
  to: 'UTF16', // to_encoding
  from: 'UTF8' // from_encoding
  bom: true // BOMをつける
});
```

UTF16 のバイトオーダーはデフォルトで big-endian になります

little-endian として変換したい場合は bom オプションに 'LE' を指定します  

```javascript
var utf16leArray = Encoding.convert(utf8Array, {
  to: 'UTF16', // to_encoding
  from: 'UTF8' // from_encoding
  bom: 'LE' // BOM (little-endian) をつける
});
```

BOM が不要な場合は UTF16LE または UTF16BE を使用します

```javascript
var utf16beArray = Encoding.convert(utf8Array, {
  to: 'UTF16BE',
  from: 'UTF8'
});
```

※ UTF16, UTF16BE, UTF16LE は、JavaScript の内部コードではなく各バイトを持つ配列です

#### 文字コード検出 (detect):

* {_Array_} Encoding.**detect** ( data [, encodings ] )  
  文字コードを検出します  
  @param {_Array.<number>_|_TypedArray_} _data_ 対象のデータ  
  @param {_(string|Array.<string>)_} [_encodings_] 検出を絞り込む際の文字コード  
  @return {_string|boolean_}  検出された文字コード、または false が返ります

```javascript
// 自動判別 (AUTO detect)
var detected = Encoding.detect(utf8Array);
if (detected === 'UTF8') {
  console.log('Encoding is UTF-8');
}

// 文字コード指定判別
var isSJIS = Encoding.detect(sjisArray, 'SJIS');
if (isSJIS) {
  console.log('Encoding is SJIS');
}
```


##### URL Encode/Decode:

* {_Array_} Encoding.**urlEncode** ( data )  
  URL(percent) エンコードします  
  @param {_Array.<number>_|_TypedArray_} _data_ 対象のデータ  
  @return {_string_}  エンコードされた文字列が返ります

* {_Array_} Encoding.**urlDecode** ( string )  
  URL(percent) デコードします  
  @param {_string_} _string_ 対象の文字列  
  @return {_Array.<number>_}  デコードされた文字コード配列が返ります

```javascript
// 文字コードの配列をURLエンコード/デコード
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

##### XMLHttpRequest と Typed arrays (Uint8Array) を使用した例:

このサンプルでは Shift_JIS で書かれたテキストファイルをバイナリデータとして読み込み、Encoding.convert によって Unicode に変換して表示します

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

##### 文字コード変換例:

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

##### 文字コード自動検出での変換例 (Auto detect):

```javascript
var sjisArray = [
  130, 177, 130, 241, 130, 201, 130, 191, 130, 205, 129,
   65, 130, 217, 130, 176, 129, 153, 130, 210, 130, 230
];
var unicodeArray = Encoding.convert(sjisArray, {
  to: 'UNICODE',
  from: 'AUTO'
});
// codeToStringは文字コード配列を文字列に変換(連結)して返します
console.log( Encoding.codeToString(unicodeArray) );
// output: 'こんにちは、ほげ☆ぴよ'
```

### Utilities

* {_string_} Encoding.**codeToString** ( {_Array.<number>_|_TypedArray_} data )  
  文字コード配列を文字列に変換(連結)して返します

* {_Array.<number>_} Encoding.**stringToCode** ( {_string_} string )  
  文字列を文字コード配列に分割して返します


### Demo

[文字コード変換テスト(Demo)](http://polygonplanet.github.io/encoding.js/tests/encoding-test.html)

### License

Dual licensed under the MIT or GPL v2 licenses.



