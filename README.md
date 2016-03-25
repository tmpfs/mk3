# Renderers

[![Build Status](https://travis-ci.org/mkdoc/mkout.svg?v=3)](https://travis-ci.org/mkdoc/mkout)
[![npm version](http://img.shields.io/npm/v/mkout.svg?v=3)](https://npmjs.org/package/mkout)
[![Coverage Status](https://coveralls.io/repos/mkdoc/mkout/badge.svg?branch=master&service=github&v=3)](https://coveralls.io/github/mkdoc/mkout?branch=master)

> Render documents to Markdown, Text, HTML, XML, YAML and JSON

Output renderers for [commonmark][].

## Install

```
npm i mkout --save
```

For the command line interface install [mkdoc][] globally (`npm i -g mkdoc`).

## Usage

Create the stream and write a [commonmark][] document:

```javascript
var out = require('mkcat')
  , ast = require('mkast');
ast.src('# Heading')
  .pipe(out())
  .pipe(process.stdout);
```

## Example

Print as markdown:

```shell
mkcat README.md | mkout
```

Print as HTML:

```shell
mkcat README.md | mkout --html
```

Print as XML:

```shell
mkcat README.md | mkout --xml
```

Print as plain text:

```shell
mkcat README.md | mkout --text
```

YAML is particularly useful to get a compact view of the tree:

```shell
mkcat README.md | mkout -y
```

For more detailed YAML use:

```shell
mkcat README.md | mkout -Y
```

Print as JSON:

```shell
mkcat README.md | mkout -j
```

Pass through the input newline-delimited JSON:

```shell
mkcat README.md | mkout --noop
```

## Renderer Implementation

This section briefly describes the handling of the various [commonmark][] types, see the [api docs](https://github.com/mkdoc/mkout/blob/master/API.md) for more detail.

### Markdown

The markdown renderer renders all basic markdown types as expected however it is not yet [commonmark][] compliant which is work in progress.

### Text

The text renderer inherits from the markdown renderer so you can choose which types to preserve as markdown, by default headings and lists are preserved. Headings are preserved to maintain the document structure but you may disable them; list rendering is always performed using the underlying markdown renderer.

Block and inline HTML is normalized to text but may be preserved.

Links are converted to indexed references in the form `example[1]` and the appropriate references are appended to the document: `[1]:http://example.com`. Duplicate link destinations are resolved.

If you wanted to preserve some inline elements in addition to headings and lists you could pass the options:

```javascript
{preserve:{heading: true, code: true, emph: true, strong: true}}
```

Soft line breaks are removed but you can preserve them. Thematic breaks (`---`) are rendered as a series of 80 hyphens which may be customised.

Code blocks are indented with two spaces, the info string if present is not preserved.

### JSON

The JSON renderer allows serializing a node tree such that it could be passed between processes or pushed to a remote queue for further processing.

Circular references are resolved and the document has enough information to recreate a node tree with a 1:1 correlation with the original.

### YAML

The YAML renderer is designed to provide a compact view of the tree which is easy to read but can also include the node properties for an extended view of the document.

Whilst it would be possible to recreate a node tree from a YAML document it is optimized for legibility over processing; use the JSON renderer for serialization requirements.

Compact output for a simple document:

```yaml
---
- document: 
  - paragraph: 
    - text: 'Generated by '
    - link: 
      - text: 'mkdoc'
---
```

## Help

```
mkout [options]

Render an abstract syntax tree.

  -H, --html           Set output renderer to HTML
  -x, --xml            Set output renderer to XML
  -m, --man            Set output renderer to MAN
  -y, --yaml           Set output renderer to YAML
  -Y, --yaml-full      Do not compact YAML output
  -t, --text           Set output renderer to TEXT
  -j, --json           Set output renderer to JSON
  -n, --noop           Pass through input JSON
  -o, --output=[FILE]  Write output to FILE (default: stdout)
  -h, --help           Display this help and exit
  --version            Print the version and exit

Report bugs to https://github.com/mkdoc/mkout/issues
```

## API

### out

```javascript
out([opts][, cb])
```

Print via a renderer to an output stream.

Returns an output stream.

* `opts` Object processing options.
* `cb` Function callback function.

#### Options

* `type` String output type.
* `input` Readable=process.stdin input stream.
* `output` Writable=process.stdout output stream.
* `render` Object renderer options.

### JsonRenderer

```javascript
new JsonRenderer([opts])
```

Renders an abstract syntax tree to JSON.

By default prints a compact JSON document, pass `indent` for indented
output:

```javascript
{indent: 1}
```

But be careful the tree can be very deep so it is not recommended you set
`indent` to greater than two.

* `opts` Object processing options.

#### Options

* `indent` Number=0 number of spaces to indent the JSON.

### ManRenderer

```javascript
new ManRenderer([opts])
```

Renders an abstract syntax tree to a ROFF man page.

* `opts` Object processing options.

### MarkdownRenderer

```javascript
new MarkdownRenderer([opts])
```

Renders an abstract syntax tree to markdown.

Eventually the aim is to make the output of this renderer fully
[commonmark][] compliant, at the moment it's output has not been
completely tested for compliance.

* `opts` Object processing options.

### normalize

```javascript
normalize(text)
```

Strips HTML tags from a string and collapses whitespace similar to how
XML text nodes are normalized.

This is designed for the TEXT and MAN output formats so we are not
concerned with XSS attacks, use `striptags` or another library if you
need to strip tags destined for HTML output.

Returns the normalized text.

* `text` String input text.

### TextRenderer

```javascript
new TextRenderer([opts])
```

Renders an abstract syntax tree to a plain text view.

By default this implementation preserves the `heading` type as markdown
so that the structure of the document is maintained.

With the exception of the PARAGRAPH, LIST and ITEM node types all
other markdown formatting is removed. For the aforementioned exceptions
setting a `preserve` option will have no effect as they are always
preserved according to the rules for markdown rendering.

If you wish to preserve some other aspects of the markdown formatting, you
can specify options such as:

```javascript
{preserve:{heading: true, emph: true}}
```

Which would preserve emphasis in addition to the default formatting that
is preserved. If you don't want to preserve any markdown formatting pass
the empty object:

```javascript
{preserve: {}}
```

Code blocks (when not preserved) are indented by the whitespace specified
with the `indent` option, default is two spaces.

Block quotes are indented according to `indent` and then prefixed with a
vertical pipe (|), you can change this prefix with the `blockquote` option.

Unless `autolink` is disabled links are removed and appended to the end
of the document such that the input:

```markdown
[Commonmark](http://commonmark.org)
```

Is converted to:

```
Commonmark[1]

[1]: http://commomark.org
```

Soft line breaks are removed unless preserved and a single space is
injected when necessary.

Thematic breaks are rendered as the hyphen (-) repeated 80 times. You may
change this output with the `hr` option.

HTML is normalized to text unless the `html` option is given in which case
it is passed through untouched.

* `opts` Object processing options.

#### Options

* `autolink` Boolean=true create automatic links by index.
* `indent` String amount of whitespace indentation for code blocks.
* `preserve` Object map of node types that should be preserved.

### YamlRenderer

```javascript
new YamlRenderer([opts])
```

Renders an abstract syntax tree to YAML.

This implementation is designed to provide an easy to read view of the
tree; whilst it would certainly be possible to recreate the tree from the
YAML output it has not been optimized for that use case.

By default a compact view of the tree is rendered, if you also want to
inspect the node properties disable `compact`:

```javascript
{compact: false}
```

* `opts` Object processing options.

#### Options

* `compact` Boolean=true create compact YAML documents.

## License

MIT

---

Created by [mkdoc](https://github.com/mkdoc/mkdoc).

[mkdoc]: https://github.com/mkdoc/mkdoc
[node]: http://nodejs.org
[npm]: http://www.npmjs.org
[commonmark]: http://commonmark.org
[jshint]: http://jshint.com
[jscs]: http://jscs.info

