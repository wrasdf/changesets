﻿# changesets
This is library allows you to build text-based concurrent multi-user applications.

It was built with the following requirements in mind:
* intention preservation
* reversibility/invertibility (undo effect)
* convergence

Easily create changesets and apply them on all sites of a distributed system using Operational Transformation.

Note: While, at the current stage of development, this library only implements a text-based changeset solution, I intend to add functionality for tree-based data and maybe even images.

## Install
`npm install changesets`

In your code, `require('changesets')`

## Usage
```
var cs = require('changesets')
```

### Constructing and applying changesets
Construct a changeset between two texts:
```js
var changes = cs.text.constructChangeset(text1, text2)
```
You get a `cs.Changeset` object containing multiple `cs.Operation`s. The changeset can be applied to a text as follows:
```js
var finalText = changes.apply(text1)

finalText == text2 // true
```

### Operational transformation
*Inclusion Transformation* as well as *Exclusion Transformation* are supported.

#### Inclusion Transformation
Say, for instance, you give a text to two different people. Each of them makes some changes and hands them back to you.
```js
var text = "Hello adventurer!"
  , textA = "Hello treasured adventurer!"
  , textB = "Good day adventurers, y'all!"
```
Now, what do you do? As a human you're certainly able to determine the changes and apply them both on the original text, but a machine is hard put to do so without proper guidance. And this is where this library comes in. Firstly, you'll need to extract the changes in each version.
```js
var csA = cs.text.constructChangeset(text, textA)
var csB = cs.text.constructChangeset(text, textB)
```
The problem is that at least one changeset becomes invalid when we try to apply it on a text that was created by applying the other changeset, because they both still assume the original context:
```js
csA.apply(textB) // -> "Good dtreasured ay adventurer!"
csB.apply(textA) // -> "Good day treasured advs, y'allenturer!"
```
Since we can at least safely apply one of them, we'll apply changeset A first on the original text. Now, in order to be able to apply changeset B, which still assumes the original context, we need to adjust it, based on the changes of changeset A, so that it still has the same effect on the text that was originally intended.
```js
var csB_new = csB.transformAgainst(csA)

textA = csA.apply(text)
csB_new.apply(textA)
// "Good day treasured adventurers, y'all!"
```
In this scenario we employed *Inclusion Transformation*, which adjusts a changeset in a way so that it assumes the changes of another changeset already happened.

#### Exclusion Transformation
Imagine a text editor, where users that allows users to undo any edit they've ever done to a document. Naturally, one will choose to store all edits as a list of changesets, where each applied on top of the other results in the currently visible document.
```js
var versions =
[ ""
, "a"
, "ab"
, "abc"
]

var edits = []
for (var i=1; i < versions.length; i++) {
  edits.push( cs.text.constructChangeset(text[i-1], text[i]) )
}
```
Now, if we want to undo a certain edit in the document's history without undoing the following edits, we need to construct the inverse changeset of the given one.
```js
var inverse = edits[1].invert()
```
Now we need to transform all following edits against this inverse changeset and in turn transform it against the previously iterated edits.
```
var newEdits = [""]
for (var i=1; i < edits.length; i++) {
  newEdits[i] = edits[i].transformAgainst(inverse)
  inverse = inverse.transformAgainst(newEdits[i])
}
```
This way we effectively exclude the given changes from all following changesets.

# More information
Anyone interested may want to start with [Wikipedia's entry on Operational Transformation](https://en.wikipedia.org/wiki/Operational_transformation) and a compehensive list of [Frequently asked questions concerning OT](http://www3.ntu.edu.sg/home/czsun/projects/otfaq) (quite extensive!).

If anyone is interested, under the hood *changesets* makes use of Neil Frasers amazing [*diff-match-patch* library](https://code.google.com/p/google-diff-match-patch/) for generating the diff between two texts.

# Todo
* Add tests for Exclusion Transformation
* make Changeset#substract() more sane
* What happens, if you apply the same CS multiple times, with or without transforming it?
* Perhaps add text length diff to `Operation`s in order to be able validate them
* Add a `pack()`/`unpack()`method to changesets

# License
MIT