# Password Generator

A password generator that runs entirely in the tab. Pick a length, toggle which
character classes to include, generate, copy.

**[Live demo →](https://vette1123.github.io/Passowrd-Generator/)**

## Features

- Length input
- Independent toggles for uppercase, lowercase, numbers and symbols
- One-click copy to clipboard
- Returns an empty string when every class is switched off, rather than
  generating something that satisfies nothing

## Stack

Plain HTML, CSS and JavaScript. No dependencies, no build step — a password
tool is precisely the wrong place to pull in a supply chain.

## How generation works

The obvious implementation draws every character from one merged pool, which
can return a password satisfying **none** of the classes you asked for. This
one walks the selected types in a cycle instead, appending one character from
each per pass, then trims to the requested length:

```js
function getRandomPassowrd(lower, upper, number, symbol, length) {
  let generatedPassword = ''
  const typesCount = lower + upper + number + symbol
  const typesArray = [{ lower }, { upper }, { number }, { symbol }].filter(
    (item) => Object.values(item)[0]
  )
  if (typesCount === 0) return ''

  for (let i = 0; i < length; i += typesCount) {
    typesArray.forEach((type) => {
      const funcName = Object.keys(type)[0]
      generatedPassword += randomFunc[funcName]()
    })
  }
  return generatedPassword.slice(0, length)
}
```

Every enabled class is therefore guaranteed to appear, which is what satisfies
"must contain a symbol" style rules reliably.

## Known limitations

Being honest about these, because they matter more in a security tool than
anywhere else:

- **The classes land in a fixed repeating pattern.** Because the cycle order
  never varies, a four-class password is always upper, lower, number, symbol,
  upper, lower… That is structure an attacker can assume, and it shrinks the
  real search space below what the length suggests. A Fisher–Yates shuffle of
  the result characters before returning would remove it in three lines.
- **`Math.random()` is not cryptographically secure.** It is not seeded from a
  secure source and its output is predictable given enough samples.
  `crypto.getRandomValues()` is available in every browser this runs in and is
  the correct primitive here.
- **Truncation can drop a guaranteed class.** `slice(0, length)` cuts the last
  pass short, so a length that is not a multiple of the number of enabled
  classes can lose one — length 5 with four classes yields upper, lower,
  number, symbol, upper, and no second symbol was ever the problem, but the
  guarantee only holds when `length >= typesCount`.
- **Copy uses `document.execCommand('copy')`** with a hidden textarea — the
  approach that predated the async Clipboard API, now deprecated. `navigator.
  clipboard.writeText()` is the modern equivalent.

## Running locally

```bash
git clone https://github.com/Vette1123/Passowrd-Generator.git
cd Passowrd-Generator
# open index.html — no build, no server needed
```

---

Built by [Mohamed Gado](https://mohamedgado.com)
