# [dashphrase.js][dashphrasejs] (for browsers)

Secure Dash HD Wallet Passphrase Generator that works in Node, Bundlers, and
Browsers.

[BIP-39][bip39]-compatible \
<small>uses standard dictionary of Base2048 mnemonic passphrases (word lists)</small>

Lightweight. Zero dependencies. 20kb (17kb min, 7.4kb gz) ~150 LoC. \
<small>(most of the package weight is due to the base2048 word list)</small>

[bip39]: https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki
[dashphrasejs]: https://github.com/dashhive/dashphrase.js

```html
<script src="https://unpkg.com/dashphrase/dashphrase.js"></script>
<script type="module">
  "use strict";

  let Dashphrase = window.Dashphrase;

  let passphrase = await Dashphrase.generate(128);
  // often delay margin arch
  // index wrap fault duck
  // club fabric demise scout

  let keyBytes = await Dashphrase.pbkdf2(passphrase);
  // Uint8Array[64] (suitable for use with importKey for AES, etc)

  let fooKeyBytes = await Dashphrase.pbkdf2(passphrase, "foo");
  // Uint8Array[64] (a completely different key, determined by "foo")
</script>
```

| Target Entropy |         Number of Words | Total Bits                             |
| -------------- | ----------------------: | :------------------------------------- |
| 128-bit        | 12 words @ 11 bits each | = 132 bits (128 bits + 4-bit checksum) |
| 160-bit        | 15 words @ 11 bits each | = 165 bits (160 bits + 5-bit checksum) |
| 192-bit        | 18 words @ 11 bits each | = 198 bits (192 bits + 6-bit checksum) |
| 224-bit        | 21 words @ 11 bits each | = 231 bits (224 bits + 7-bit checksum) |
| 256-bit        | 24 words @ 11 bits each | = 264 bits (256 bits + 8-bit checksum) |

## Features & Use Cases

- [x] Base2048 (BIP-0039 compliant)
- [x] Easy to retype on different devices
- [x] Seed many, distinct keys from a single passphrase
- [x] Keys for AES Encryption & Decryption
- [x] Air Gap security
- [x] Cryptocurrency wallets

## API

- generate
  - encode
- checksum
  - decode
- pbkdf2
- base2048.includes

### Dashphrase.generate(bitlen)

Generate a "Base2048" passphrase - each word represents 11 bits of entropy.

```js
await Dashphrase.generate(bitLen); // *128*, 160, 192, 224, or 256
```

### Dashphrase.encode(bytes)

Encode an array of 16, 20, 24, 28, or 32 bytes (typically a `Uint8Array`) into a
passphrase using the Base2048 word list dictionary.

```js
let bytes = Uint8Array.from([0, 255, 0, 255, 0, 255, 0, 255, 0, 255, 0, 255]);

await Dashphrase.encode(bytes);
// "abstract way divert acid useless legend advance theme youth"
```

### Dashphrase.checksum(passphrase)

We all make mistakes. Especially typos.

Running the checksum can't guarantee that the passphrase is correct, but most
typos - such as `brocolli` instead of `broccoli` - will cause it to fail, so
that's a start.

```js
let passphrase = "often delay margin arch ...";
await Dashphrase.checksum(passphrase); // true
```

```js
let passphrase = "often delay margin arch TYPO";
await Dashphrase.checksum(passphrase).catch(function (err) {
  // checksum failed?
  throw err;
});
```

### Dashphrase.decode(words)

Decode an string of space-delimited words from the Base2048 dictionary into a
Uint8Array.

This will throw an error if any non-Base2048-compatible words are used, or if
the checksum does not match.

```js
let words = "abstract way divert acid useless legend advance theme youth";

await Dashphrase.decode(words);
// Uint8Array[12] <0, 255, 0, 255, 0, 255, 0, 255, 0, 255, 0, 255>
```

### Dashphrase.toSeed(passphraseMnemonic, saltPassword)

Generate a private key seed or encryption key based on the passphrase (mnemonic
word list) and some other string - whether a salt, a password, another
passphrase or secret, or an id of some kind.

```js
await Dashphrase.toSeed(passphraseMnemonic, saltPassword || ""); // Uint8Array[64]
```

### Dashphrase.base2048.includes(word)

Check if a given word exists in the base2048 dictionary.

```js
Dashphrase.base2048.includes("broccoli"); // true
```

```js
Dashphrase.base2048.includes("brocolli"); // false
```

#### Get all misspelled words

```js
"hammer spoon brocolli zoo".split(" ").filter(function (word) {
  return word && !Dashphrase.base2048.includes(word);
});
// [ "brocolli" ]
```

## Compatibility Testing

- Passes Trezor's
  [python-mnemonic](https://github.com/trezor/python-mnemonic/blob/master/vectors.json)
  tests

```sh
npm run test
```

## LICENSE

Copyright 2023 Dash Incubator \
 (forked from [therootcompany/passphrase.js][passphrase-js], re-license as MIT
with permission) \
Copyright 2021 AJ ONeal (MPL-2.0 License) \
Copyright 2021 Root, LLC (MPL-2.0 License)

[passphrase-js]: https://github.com/therootcompany/passphrase.js
