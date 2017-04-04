# markdown-toc [![NPM version](https://img.shields.io/npm/v/markdown-toc.svg?style=flat)](https://www.npmjs.com/package/markdown-toc) [![NPM monthly downloads](https://img.shields.io/npm/dm/markdown-toc.svg?style=flat)](https://npmjs.org/package/markdown-toc)  [![NPM total downloads](https://img.shields.io/npm/dt/markdown-toc.svg?style=flat)](https://npmjs.org/package/markdown-toc) [![Linux Build Status](https://img.shields.io/travis/jonschlinkert/markdown-toc.svg?style=flat&label=Travis)](https://travis-ci.org/jonschlinkert/markdown-toc)

> Generate a markdown TOC (table of contents) with Remarkable.

## Table of Contents

- [Install](#install)
- [CLI](#cli)
- [Highights](#highights)
- [Usage](#usage)
- [API](#api)
  * [toc.plugin](#tocplugin)
  * [toc.json](#tocjson)
  * [toc.insert](#tocinsert)
  * [Utility functions](#utility-functions)
- [Options](#options)
  * [options.append](#optionsappend)
  * [options.filter](#optionsfilter)
  * [options.slugify](#optionsslugify)
  * [options.bullets](#optionsbullets)
  * [options.maxdepth](#optionsmaxdepth)
  * [options.firsth1](#optionsfirsth1)
  * [options.stripHeadingTags](#optionsstripheadingtags)
- [About](#about)

_(TOC generated by [verb](https://github.com/verbose/verb) using [markdown-toc](https://github.com/jonschlinkert/markdown-toc))_

## Install

Install with [npm](https://www.npmjs.com/):

```sh
$ npm install --save markdown-toc
```

## CLI

```
Usage: markdown-toc [--json] [-i] <input>

  input:  The markdown file to parse for table of contents,
          or "-" to read from stdin.

  --json: Print the TOC in json format

  -i:     Edit the <input> file directly, injecting the TOC at <!-- toc -->
          (Without this flag, the default is to print the TOC to stdout.)
```

## Highlights

**Features**

* Can optionally be used as a [remarkable](https://github.com/jonschlinkert/remarkable) plugin
* Returns an object with the rendered TOC (on `content`), as well as a `json` property with the raw TOC object, so you can generate your own TOC using templates or however you want
* Works with [repeated headings](https://gist.github.com/jonschlinkert/ac5d8122bfaaa394f896)
* Uses sane defaults, so no customization is necessary, but you can if you need to.
* [Filters](#filter-headings) out headings you don't want
* [Improves](#titleize) the headings you do want
* Uses a custom [slugify](#optionsslugify) function to change how links are created

**Safe!**

* Won't mangle markdown in code examples in gfm code blocks that other TOC generators mistake as being actual headings (this happens when markdown headings are show in _examples_, meaning they arent' actually headings that should be in the toc. Also happens with yaml and coffee-script comments, or any comments that use `#`)
* Won't mangle front-matter, or mistake front-matter properties for headings like other TOC generators

## Usage

```js
var toc = require('markdown-toc');

toc('# One\n\n# Two').content;
// Results in:
// - [One](#one)
// - [Two](#two)
```

To allow customization of the output, an object is returned with the following properties:

* `content` **{String}**: The generated table of contents. Unless you want to customize rendering, this is all you need.
* `highest` **{Number}**: The highest level heading found. This is used to adjust indentation.
* `tokens` **{Array}**: Headings tokens that can be used for custom rendering

## API

### toc.plugin

Use as a [remarkable](https://github.com/jonschlinkert/remarkable) plugin.

```js
var Remarkable = require('remarkable');
var toc = require('markdown-toc');

function render(str, options) {
  return new Remarkable()
    .use(toc.plugin(options)) // <= register the plugin
    .render(str);
}
```

**Usage example**

```js
var results = render('# AAA\n# BBB\n# CCC\nfoo\nbar\nbaz');
```

Results in:

```
- [AAA](#aaa)
- [BBB](#bbb)
- [CCC](#ccc)
```

### toc.json

Object for creating a custom TOC.

```js
toc('# AAA\n## BBB\n### CCC\nfoo').json;

// results in
[ { content: 'AAA', slug: 'aaa', lvl: 1 },
  { content: 'BBB', slug: 'bbb', lvl: 2 },
  { content: 'CCC', slug: 'ccc', lvl: 3 } ]
```

### toc.insert

Insert a table of contents immediately after an _opening_ `<!-- toc -->` code comment, or replace an existing TOC if both an _opening_ comment and a _closing_ comment (`<!-- tocstop -->`) are found.

_(This strategy works well since code comments in markdown are hidden when viewed as HTML, like when viewing a README on GitHub README for example)._

**Example**

```
<!-- toc -->
- old toc 1
- old toc 2
- old toc 3
<!-- tocstop -->

## abc
This is a b c.

## xyz
This is x y z.
```

Would result in something like:

```
<!-- toc -->
- [abc](#abc)
- [xyz](#xyz)
<!-- tocstop -->

## abc
This is a b c.

## xyz
This is x y z.
```

### Utility functions

As a convenience to folks who wants to create a custom TOC, markdown-toc's internal utility methods are exposed:

```js
var toc = require('markdown-toc');
```

* `toc.bullets()`: render a bullet list from an array of tokens
* `toc.linkify()`: linking a heading `content` string
* `toc.slugify()`: slugify a heading `content` string
* `toc.strip()`: strip words or characters from a heading `content` string

**Example**

```js
var result = toc('# AAA\n## BBB\n### CCC\nfoo');
var str = '';

result.json.forEach(function(heading) {
  str += toc.linkify(heading.content);
});
```

## Options

### options.append

Append a string to the end of the TOC.

```js
toc(str, {append: '\n_(TOC generated by Verb)_'});
```

### options.filter

Type: `Function`

Default: `undefined`

Params:

* `str` **{String}** the actual heading string
* `ele` **{Objecct}** object of heading tokens
* `arr` **{Array}** all of the headings objects

**Example**

From time to time, we might get junk like this in our TOC.

```
[.aaa([foo], ...) another bad heading](#-aaa--foo--------another-bad-heading)
```

Unless you like that kind of thing, you might want to filter these bad headings out.

```js
function removeJunk(str, ele, arr) {
  return str.indexOf('...') === -1;
}

var result = toc(str, {filter: removeJunk});
//=> beautiful TOC
```

### options.slugify

Type: `Function`

Default: Basic non-word character replacement.

**Example**

```js
var str = toc('# Some Article', {slugify: require('uslug')});
```

### options.bullets

Type: `String|Array`

Default: `*`

The bullet to use for each item in the generated TOC. If passed as an array (`['*', '-', '+']`), the bullet point strings will be used based on the header depth.

### options.maxdepth

Type: `Number`

Default: `6`

Use headings whose depth is at most maxdepth.

### options.firsth1

Type: `Boolean`

Default: `true`

Exclude the first h1-level heading in a file. For example, this prevents the first heading in a README from showing up in the TOC.

### options.stripHeadingTags

Type: `Boolean`

Default: `true`

Strip extraneous HTML tags from heading text before slugifying. This is similar to GitHub markdown behavior.

## About

### Related projects

* [gfm-code-blocks](https://www.npmjs.com/package/gfm-code-blocks): Extract gfm (GitHub Flavored Markdown) fenced code blocks from a string. | [homepage](https://github.com/jonschlinkert/gfm-code-blocks "Extract gfm (GitHub Flavored Markdown) fenced code blocks from a string.")
* [markdown-link](https://www.npmjs.com/package/markdown-link): Micro util for generating a single markdown link. | [homepage](https://github.com/jonschlinkert/markdown-link "Micro util for generating a single markdown link.")
* [markdown-utils](https://www.npmjs.com/package/markdown-utils): Micro-utils for creating markdown snippets. | [homepage](https://github.com/jonschlinkert/markdown-utils "Micro-utils for creating markdown snippets.")
* [pretty-remarkable](https://www.npmjs.com/package/pretty-remarkable): Plugin for prettifying markdown with Remarkable using custom renderer rules. | [homepage](https://github.com/jonschlinkert/pretty-remarkable "Plugin for prettifying markdown with Remarkable using custom renderer rules.")
* [remarkable](https://www.npmjs.com/package/remarkable): Markdown parser, done right. 100% Commonmark support, extensions, syntax plugins, high speed - all in… [more](https://github.com/jonschlinkert/remarkable) | [homepage](https://github.com/jonschlinkert/remarkable "Markdown parser, done right. 100% Commonmark support, extensions, syntax plugins, high speed - all in one.")

### Contributing

Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](../../issues/new).

### Contributors

| **Commits** | **Contributor** | 
| --- | --- |
| 190 | [jonschlinkert](https://github.com/jonschlinkert) |
| 4 | [stefanwalther](https://github.com/stefanwalther) |
| 3 | [Marsup](https://github.com/Marsup) |
| 2 | [dvcrn](https://github.com/dvcrn) |
| 2 | [maxogden](https://github.com/maxogden) |
| 2 | [twang2218](https://github.com/twang2218) |
| 2 | [angrykoala](https://github.com/angrykoala) |
| 2 | [zeke](https://github.com/zeke) |
| 1 | [Vortex375](https://github.com/Vortex375) |
| 1 | [owzim](https://github.com/owzim) |
| 1 | [chendaniely](https://github.com/chendaniely) |
| 1 | [Feder1co5oave](https://github.com/Feder1co5oave) |
| 1 | [garygreen](https://github.com/garygreen) |
| 1 | [TehShrike](https://github.com/TehShrike) |
| 1 | [citizenmatt](https://github.com/citizenmatt) |
| 1 | [rafaelsteil](https://github.com/rafaelsteil) |
| 1 | [RichardBradley](https://github.com/RichardBradley) |
| 1 | [sethvincent](https://github.com/sethvincent) |
| 1 | [lu22do](https://github.com/lu22do) |

### Building docs

_(This document was generated by [verb-generate-readme](https://github.com/verbose/verb-generate-readme) (a [verb](https://github.com/verbose/verb) generator), please don't edit the readme directly. Any changes to the readme must be made in [.verb.md](.verb.md).)_

To generate the readme and API documentation with [verb](https://github.com/verbose/verb):

```sh
$ npm install -g verb verb-generate-readme && verb
```

### Running tests

Install dev dependencies:

```sh
$ npm install -d && npm test
```

### Author

**Jon Schlinkert**

* [github/jonschlinkert](https://github.com/jonschlinkert)
* [twitter/jonschlinkert](https://twitter.com/jonschlinkert)

### License

Copyright © 2017, [Jon Schlinkert](https://github.com/jonschlinkert).
Released under the [MIT license](LICENSE).

***

_This file was generated by [verb-generate-readme](https://github.com/verbose/verb-generate-readme), v0.4.1, on January 15, 2017._
