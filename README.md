# [DashPhrase.js][dashphrasejs]

Secure HD Wallet _Recovery Phrase_ Generator for that works in Node, Bundlers,
and Browsers.

[BIP-39][bip39]-compatible \
<small>uses the standard "mnemonic" dictionary for Base2048 words</small>

Lightweight. Zero dependencies. 20kb (17kb min, 7.4kb gz) ~150 LoC. \
<small>(most of the package weight is due to the base2048 word list)</small>

[bip39]: https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki
[dashphrasejs]: https://github.com/dashhive/dashphrase.js

## Features & Use Cases

- [x] Base2048 (BIP-0039 compliant)
- [x] Easy to retype on different devices
- [x] Seed many, distinct keys from a single recovery phrase
- [x] Keys for AES Encryption & Decryption
- [x] Air Gap security
- [x] Cryptocurrency wallets

| Target Entropy |         Number of Words | Total Bits                             |
| -------------- | ----------------------: | :------------------------------------- |
| 128-bit        | 12 words @ 11 bits each | = 132 bits (128 bits + 4-bit checksum) |
| 160-bit        | 15 words @ 11 bits each | = 165 bits (160 bits + 5-bit checksum) |
| 192-bit        | 18 words @ 11 bits each | = 198 bits (192 bits + 6-bit checksum) |
| 224-bit        | 21 words @ 11 bits each | = 231 bits (224 bits + 7-bit checksum) |
| 256-bit        | 24 words @ 11 bits each | = 264 bits (256 bits + 8-bit checksum) |

## Install

**Node**, **Bun**, & **Bundlers**:

```sh
npm install --save dashphrase@1.2.2
```

```js
"use strict";

let DashPhrase = require("dashphrase");
```

**Browsers**

```html
<script src="https://unpkg.com/dashphrase@1.2.2/dashphrase.js"></script>
<script type="module">
  "use strict";

  let DashPhrase = window.DashPhrase;
  // ...
</script>
```

## Usage

```js
let recoveryPhrase = await DashPhrase.generate(128);
// cat swing flag economy
// stadium alone churn speed
// unique patch report train

let keyBytes = await DashPhrase.toSeed(recoveryPhrase);
// Uint8Array[64] (suitable for use with importKey for AES, etc)

let fooKeyBytes = await DashPhrase.toSeed(recoveryPhrase, "foo");
// Uint8Array[64] (a completely different key, determined by "foo")
```

Values to use for Testing, Demos, Development, and Debugging:

```js
DashPhrase.ZOOMONIC;
DashPhrase.ZECRET;
DashPhrase.ZEED;
DashPhrase.CATMONIC;
```

## Test Fixtures

### Zoomonic

The canonical DASH _Recovery Phrase_ (mnemonic), _Secret Salt_, & _Seed_ test
values are:

```text
zoo zoo zoo zoo zoo zoo zoo zoo zoo zoo zoo wrong
```

That's eleven (11) 'zoo's and one (1) 'wrong'.

If we decode that, we get the "_input entropy_". \
For extra entropy / projection, we can also use a "_secret salt_". \
If we run the appropriate _Key Derivation_ on those we the "_seed_". \
Described as JSON:

With _secret salt_:

```json
{
  "inputEntropy": "ffffffffffffffffffffffffffffffff",
  "recoveryPhrase": "zoo zoo zoo zoo zoo zoo zoo zoo zoo zoo zoo wrong",
  "secretSalt": "TREZOR",
  "seed": "ac27495480225222079d7be181583751e86f571027b0497b5b5d11218e0a8a13332572917f0f8e5a589620c6f15b11c61dee327651a14c34e18231052e48c069"
}
```

Empty _secret salt_:

```json
{
  "inputEntropy": "ffffffffffffffffffffffffffffffff",
  "recoveryPhrase": "zoo zoo zoo zoo zoo zoo zoo zoo zoo zoo zoo wrong",
  "secretSalt": "",
  "seed": "b6a6d8921942dd9806607ebc2750416b289adea669198769f2e15ed926c3aa92bf88ece232317b4ea463e84b0fcd3b53577812ee449ccc448eb45e6f544e25b6"
}
```

### Catmonic

Or, for the times you need something that _looks_ random:

```text
cat swing flag economy stadium alone churn speed unique patch report train
```

## API

- generate
  - encode
- verify (checksum)
  - decode
- toSeed
- base2048.includes
- Errors
  - `E_UNKNOWN_WORD`
  - `E_BAD_CHECKSUM`
- CATMONIC
- ZOOMONIC
- ZECRET
- ZEED

### DashPhrase.generate(bitlen)

Generate a "Base2048" recovery phrase.\
(each word represents 11 bits of entropy)

```js
await DashPhrase.generate(bitLen); // *128*, 160, 192, 224, or 256
```

### DashPhrase.encode(bytes)

Encode an array of 16, 20, 24, 28, or 32 bytes (typically a `Uint8Array`) into a
recovery phrase using the Base2048 word list dictionary.

```js
let bytes = Uint8Array.from([0, 255, 0, 255, 0, 255, 0, 255, 0, 255, 0, 255]);

await DashPhrase.encode(bytes);
// "abstract way divert acid useless legend advance theme youth"
```

### DashPhrase.verify(recoveryPhrase)

We all make mistakes. Especially typos.

Running the checksum can't guarantee that the recovery phrase is correct, but
most typos - such as `brocolli` instead of `broccoli` - will cause it to fail,
so that's a start. \
(although this _does_ check the checksum as well)

```js
let recoveryPhrase = "often delay margin arch ...";
await DashPhrase.verify(recoveryPhrase); // true
```

```js
let recoveryPhrase = "often delay margin arch TYPO";
await DashPhrase.verify(recoveryPhrase).catch(function (err) {
  // checksum failed?
  throw err;
});
```

### DashPhrase.decode(words, { verify: true })

Decode an string of space-delimited words from the Base2048 dictionary into a
Uint8Array.

This will throw an error if any non-Base2048-compatible words are used, or if
the checksum does not match.

```js
let words = "abstract way divert acid useless legend advance theme youth";

let entropy = await DashPhrase.decode(words);
// Uint8Array[12] <0, 255, 0, 255, 0, 255, 0, 255, 0, 255, 0, 255>
```

### DashPhrase.toSeed(recoveryPhrase, saltPassword, { verify: true })

Generate a private key seed or encryption key based on the recovery phrase
(mnemonic) and some other string - whether a salt, a password, another recovery
phrase or secret, or an id of some kind.

```js
await DashPhrase.toSeed(recoveryPhrase, saltPassword || ""); // Uint8Array[64]
```

If you'd like to skip the word and checksum checks, pass `{ verify: false }`.

### DashPhrase.base2048.includes(word)

Check if a given word exists in the base2048 dictionary.

```js
DashPhrase.base2048.includes("broccoli"); // true
```

```js
DashPhrase.base2048.includes("brocolli"); // false
```

#### Get all misspelled words

```js
"hammer spoon brocolli zoo".split(" ").filter(function (word) {
  return word && !DashPhrase.base2048.includes(word);
});
// [ "brocolli" ]
```

### Errors

- `E_UNKNOWN_WORD` - at least one the given words is not in the word list
- `E_BAD_CHECKSUM` - words were good, but checksum failed

### DashPhrase.CATMONIC

A secondary Recovery Phrase Mnemonic for documentation, examples, testing, etc.
\
(use when you need something that appears more random)

```text
cat swing flag economy stadium alone churn speed unique patch report train

TREZOR

deb5f45449e615feff5640f2e49f933ff51895de3b4381832b3139941c57b59205a42480c52175b6efcffaa58a2503887c1e8b363a707256bdd2b587b46541f5
```

(included in canonical test vector, chosen for being friendly & pleasant) \
(and still fits in 80 characters) \
(and everybody loves cats) \
(except me)

### DashPhrase.ZOOMONIC

The Recovery Phrase Mnemonic to use for documentation, examples, testing, etc.

```text
zoo zoo zoo zoo zoo zoo zoo zoo zoo zoo zoo wrong
```

(chosen for being easy to remember, and funny - the checksum is "wrong")

### DashPhrase.ZECRET

The Secret Salt to use for documentation examples, testing & debugging, etc.

```text
TREZOR
```

### DashPhrase.ZEED

The Seed to use for documentation examples, testing & debugging, etc.

```text
ac27495480225222079d7be181583751e86f571027b0497b5b5d11218e0a8a13332572917f0f8e5a589620c6f15b11c61dee327651a14c34e18231052e48c069
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
