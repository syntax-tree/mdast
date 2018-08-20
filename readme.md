# ![MDAST][logo]

**M**ark**d**own **A**bstract **S**yntax **T**ree.

* * *

**MDAST** discloses markdown as an abstract syntax tree.  _Abstract_
means not all information is stored in this tree and an exact replica
of the original document cannot be re-created.  _Syntax Tree_ means syntax
**is** present in the tree, thus an exact syntactic document can be
re-created.

**MDAST** is a subset of [unist][], and implemented by [remark][].

This document may not be released. See [releases][] for released
documents. The latest released version is [`2.2.0`][latest].

## Table of Contents

*   [AST](#ast)
    *   [Root](#root)
    *   [Paragraph](#paragraph)
    *   [Blockquote](#blockquote)
    *   [Heading](#heading)
    *   [Code](#code)
    *   [InlineCode](#inlinecode)
    *   [YAML](#yaml)
    *   [HTML](#html)
    *   [List](#list)
    *   [ListItem](#listitem)
    *   [Table](#table)
    *   [TableRow](#tablerow)
    *   [TableCell](#tablecell)
    *   [ThematicBreak](#thematicbreak)
    *   [Break](#break)
    *   [Emphasis](#emphasis)
    *   [Strong](#strong)
    *   [Delete](#delete)
    *   [Link](#link)
    *   [Image](#image)
    *   [Footnote](#footnote)
    *   [LinkReference](#linkreference)
    *   [ImageReference](#imagereference)
    *   [FootnoteReference](#footnotereference)
    *   [Definition](#definition)
    *   [FootnoteDefinition](#footnotedefinition)
    *   [TextNode](#textnode)
*   [List of Utilities](#list-of-utilities)
*   [Related](#related)
*   [Contribute](#contribute)
*   [Acknowledgments](#acknowledgments)
*   [License](#license)

## AST

### `Root`

`Root` ([`Parent`][parent]) houses all nodes.

```idl
interface Root <: Parent {
  type: "root";
}
```

### `Paragraph`

`Paragraph` ([`Parent`][parent]) represents a unit of discourse dealing
with a particular point or idea.

```idl
interface Paragraph <: Parent {
  type: "paragraph";
}
```

For example, the following markdown:

```md
Alpha bravo charlie.
```

Yields:

```json
{
  "type": "paragraph",
  "children": [{
    "type": "text",
    "value": "Alpha bravo charlie."
  }]
}
```

### `Blockquote`

`Blockquote` ([`Parent`][parent]) represents a quote.

```idl
interface Blockquote <: Parent {
  type: "blockquote";
}
```

For example, the following markdown:

```md
> Alpha bravo charlie.
```

Yields:

```json
{
  "type": "blockquote",
  "children": [{
    "type": "paragraph",
    "children": [{
      "type": "text",
      "value": "Alpha bravo charlie."
    }]
  }]
}
```

### `Heading`

`Heading` ([`Parent`][parent]), just like with HTML, with a level greater
than or equal to 1, lower than or equal to 6.

```idl
interface Heading <: Parent {
  type: "heading";
  depth: 1 <= uint32 <= 6;
}
```

For example, the following markdown:

```md
# Alpha
```

Yields:

```json
{
  "type": "heading",
  "depth": 1,
  "children": [{
    "type": "text",
    "value": "Alpha"
  }]
}
```

### `Code`

`Code` ([`Text`][text]) occurs at block level (see
[`InlineCode`][inlinecode] for code spans).  `Code` supports an
info string and a language tag (when the line with the opening fence
contains some text, it is stored as the info string, the first word
following the fence is stored as the language tag, the rest of the
line is stored as the info string, both are null if missing)

```idl
interface Code <: Text {
  type: "code";
  lang: string | null;
  info: string | null;
}
```

For example, the following markdown:

```md
    foo()
```

Yields:

```json
{
  "type": "code",
  "lang": null,
  "info": null,
  "value": "foo()"
}
```

### `InlineCode`

`InlineCode` ([`Text`][text]) occurs inline (see [`Code`][code] for
blocks). Inline code does not sport a `lang` attribute.

```idl
interface InlineCode <: Text {
  type: "inlineCode";
}
```

For example, the following markdown:

```md
`foo()`
```

Yields:

```json
{
  "type": "inlineCode",
  "value": "foo()"
}
```

### `YAML`

`YAML` ([`Text`][text]) can occur at the start of a document, and
contains embedded YAML data.

```idl
interface YAML <: Text {
  type: "yaml";
}
```

> **Note**: YAML used to be available through the core of remark and thus
> is specified here.  Support for it now moved to
> [`remark-frontmatter`][frontmatter], and the definition here may be removed
> in the future.

For example, the following markdown:

```md
---
foo: bar
---
```

Yields:

```json
{
  "type": "yaml",
  "value": "foo: bar"
}
```

### `HTML`

`HTML` ([`Text`][text]) contains embedded HTML.

```idl
interface HTML <: Text {
  type: "html";
}
```

For example, the following markdown:

```md
<div>
```

Yields:

```json
{
  "type": "html",
  "value": "<div>"
}
```

### `List`

`List` ([`Parent`][parent]) contains [`ListItem`s][listitem].  No other nodes
may occur in lists.

The `start` property contains the starting number of the list when
`ordered: true`; `null` otherwise.

When all list items have `loose: false`, the list’s `loose` property is also
`false`.  Otherwise, `loose: true`.

```idl
interface List <: Parent {
  type: "list";
  ordered: true | false;
  start: uint32 | null;
  loose: true | false;
}
```

For example, the following markdown:

```md
1. [x] foo
```

Yields:

```json
{
  "type": "list",
  "ordered": true,
  "start": 1,
  "loose": false,
  "children": [{
    "type": "listItem",
    "loose": false,
    "checked": true,
    "children": [{
      "type": "paragraph",
      "children": [{
        "type": "text",
        "value": "foo",
      }]
    }]
  }]
}
```

### `ListItem`

`ListItem` ([`Parent`][parent]) is a child of a [`List`][list].

Loose `ListItem`s often contain more than one block-level elements.

A checked property exists on `ListItem`s, set to `true` (when checked),
`false` (when unchecked), or `null` (when not containing a checkbox).
See [Task Lists on GitHub][task-list] for information.

```idl
interface ListItem <: Parent {
  type: "listItem";
  loose: true | false;
  checked: true | false | null;
}
```

For an example, see the definition of [`List`][list].

### `Table`

`Table` ([`Parent`][parent]) represents tabular data, with alignment.
Its children are [`TableRow`][tablerow]s, the first of which acts as
a table header row.

`table.align` represents the alignment of columns.

```idl
interface Table <: Parent {
  type: "table";
  align: [alignType];
}
```

```idl
enum alignType {
  "left" | "right" | "center" | null;
}
```

For example, the following markdown:

```md
| foo | bar |
| :-- | :-: |
| baz | qux |
```

Yields:

```json
{
  "type": "table",
  "align": ["left", "center"],
  "children": [
    {
      "type": "tableRow",
      "children": [
        {
          "type": "tableCell",
          "children": [{
            "type": "text",
            "value": "foo"
          }]
        },
        {
          "type": "tableCell",
          "children": [{
            "type": "text",
            "value": "bar"
          }]
        }
      ]
    },
    {
      "type": "tableRow",
      "children": [
        {
          "type": "tableCell",
          "children": [{
            "type": "text",
            "value": "baz"
          }]
        },
        {
          "type": "tableCell",
          "children": [{
            "type": "text",
            "value": "qux"
          }]
        }
      ]
    }
  ]
}
```

### `TableRow`

`TableRow` ([`Parent`][parent]).  Its children are always
[`TableCell`][tablecell].

```idl
interface TableRow <: Parent {
  type: "tableRow";
}
```

For an example, see the definition of `Table`.

### `TableCell`

`TableCell` ([`Parent`][parent]).  Contains a single tabular field.

```idl
interface TableCell <: Parent {
  type: "tableCell";
}
```

For an example, see the definition of [`Table`][table].

### `ThematicBreak`

A `ThematicBreak` ([`Node`][node]) represents a break in content,
often shown as a horizontal rule, or by two HTML section elements.

```idl
interface ThematicBreak <: Node {
  type: "thematicBreak";
}
```

For example, the following markdown:

```md
***
```

Yields:

```json
{
  "type": "thematicBreak"
}
```

### `Break`

`Break` ([`Node`][node]) represents an explicit line break.

```idl
interface Break <: Node {
  type: "break";
}
```

For example, the following markdown (interpuncts represent spaces):

```md
foo··
bar
```

Yields:

```json
{
  "type": "paragraph",
  "children": [
    {
      "type": "text",
      "value": "foo"
    },
    {
      "type": "break"
    },
    {
      "type": "text",
      "value": "bar"
    }
  ]
}
```

### `Emphasis`

`Emphasis` ([`Parent`][parent]) represents slight emphasis.

```idl
interface Emphasis <: Parent {
  type: "emphasis";
}
```

For example, the following markdown:

```md
*alpha* _bravo_
```

Yields:

```json
{
  "type": "paragraph",
  "children": [
    {
      "type": "emphasis",
      "children": [{
        "type": "text",
        "value": "alpha"
      }]
    },
    {
      "type": "text",
      "value": " "
    },
    {
      "type": "emphasis",
      "children": [{
        "type": "text",
        "value": "bravo"
      }]
    }
  ]
}
```

### `Strong`

`Strong` ([`Parent`][parent]) represents strong emphasis.

```idl
interface Strong <: Parent {
  type: "strong";
}
```

For example, the following markdown:

```md
**alpha** __bravo__
```

Yields:

```json
{
  "type": "paragraph",
  "children": [
    {
      "type": "strong",
      "children": [{
        "type": "text",
        "value": "alpha"
      }]
    },
    {
      "type": "text",
      "value": " "
    },
    {
      "type": "strong",
      "children": [{
        "type": "text",
        "value": "bravo"
      }]
    }
  ]
}
```

### `Delete`

`Delete` ([`Parent`][parent]) represents text ready for removal.

```idl
interface Delete <: Parent {
  type: "delete";
}
```

For example, the following markdown:

```md
~~alpha~~
```

Yields:

```json
{
  "type": "delete",
  "children": [{
    "type": "text",
    "value": "alpha"
  }]
}
```

### `Link`

`Link` ([`Parent`][parent]) represents the humble hyperlink.

```idl
interface Link <: Parent {
  type: "link";
  title: string | null;
  url: string;
}
```

For example, the following markdown:

```md
[alpha](http://example.com "bravo")
```

Yields:

```json
{
  "type": "link",
  "title": "bravo",
  "url": "http://example.com",
  "children": [{
    "type": "text",
    "value": "alpha"
  }]
}
```

### `Image`

`Image` ([`Node`][node]) represents the figurative figure.

```idl
interface Image <: Node {
  type: "image";
  title: string | null;
  alt: string | null;
  url: string;
}
```

For example, the following markdown:

```md
![alpha](http://example.com/favicon.ico "bravo")
```

Yields:

```json
{
  "type": "image",
  "title": "bravo",
  "url": "http://example.com",
  "alt": "alpha"
}
```

### `Footnote`

`Footnote` ([`Parent`][parent]) represents an inline marker, whose
content relates to the document but is outside its flow.

```idl
interface Footnote <: Parent {
  type: "footnote";
}
```

For example, the following markdown:

```md
[^alpha bravo]
```

Yields:

```json
{
  "type": "footnote",
  "children": [{
    "type": "text",
    "value": "alpha bravo"
  }]
}
```

### `LinkReference`

`LinkReference` ([`Parent`][parent]) represents a humble hyperlink,
its `url` and `title` defined somewhere else in the document by a
[`Definition`][definition].

`referenceType` is needed to detect if a reference was meant as a
reference (`[foo][]`) or just unescaped brackets (`[foo]`).

```idl
interface LinkReference <: Parent {
  type: "linkReference";
  identifier: string;
  referenceType: referenceType;
}
```

```idl
enum referenceType {
  "shortcut" | "collapsed" | "full";
}
```

For example, the following markdown:

```md
[alpha][bravo]
```

Yields:

```json
{
  "type": "linkReference",
  "identifier": "bravo",
  "referenceType": "full",
  "children": [{
    "type": "text",
    "value": "alpha"
  }]
}
```

### `ImageReference`

`ImageReference` ([`Node`][node]) represents a figurative figure,
its `url` and `title` defined somewhere else in the document by a
[`Definition`][definition].

`referenceType` is needed to detect if a reference was meant as a
reference (`![foo][]`) or just unescaped brackets (`![foo]`).
See [`LinkReference`][linkreference] for the definition of `referenceType`.

```idl
interface ImageReference <: Node {
  type: "imageReference";
  identifier: string;
  referenceType: referenceType;
  alt: string | null;
}
```

For example, the following markdown:

```md
![alpha][bravo]
```

Yields:

```json
{
  "type": "imageReference",
  "identifier": "bravo",
  "referenceType": "full",
  "alt": "alpha"
}
```

### `FootnoteReference`

`FootnoteReference` ([`Node`][node]) is like [`Footnote`][footnote],
but its content is already outside the documents flow: placed in a
[`FootnoteDefinition`][footnotedefinition].

```idl
interface FootnoteReference <: Node {
  type: "footnoteReference";
  identifier: string;
}
```

For example, the following markdown:

```md
[^alpha]
```

Yields:

```json
{
  "type": "footnoteReference",
  "identifier": "alpha"
}
```

### `Definition`

`Definition` ([`Node`][node]) represents the definition (i.e., location
and title) of a [`LinkReference`][linkreference] or an
[`ImageReference`][imagereference].

```idl
interface Definition <: Node {
  type: "definition";
  identifier: string;
  title: string | null;
  url: string;
}
```

For example, the following markdown:

```md
[alpha]: http://example.com
```

Yields:

```json
{
  "type": "definition",
  "identifier": "alpha",
  "title": null,
  "url": "http://example.com"
}
```

### `FootnoteDefinition`

`FootnoteDefinition` ([`Parent`][parent]) represents the definition
(i.e., content) of a [`FootnoteReference`][footnotereference].

```idl
interface FootnoteDefinition <: Parent {
  type: "footnoteDefinition";
  identifier: string;
}
```

For example, the following markdown:

```md
[^alpha]: bravo and charlie.
```

Yields:

```json
{
  "type": "footnoteDefinition",
  "identifier": "alpha",
  "children": [{
    "type": "paragraph",
    "children": [{
      "type": "text",
      "value": "bravo and charlie."
    }]
  }]
}
```

### `TextNode`

`TextNode` ([`Text`][text]) represents everything that is just text.
Note that its `type` property is `text`, but it is different from
[`Text`][text].

```idl
interface TextNode <: Text {
  type: "text";
}
```

For example, the following markdown:

```md
Alpha bravo charlie.
```

Yields:

```json
{
  "type": "text",
  "value": "Alpha bravo charlie."
}
```

## List of Utilities

<!--lint disable list-item-spacing-->

*   [`mdast-util-assert`](https://github.com/syntax-tree/mdast-util-assert)
    — Assert MDAST nodes
*   [`mdast-add-list-metadata`](https://gitlab.com/staltz/mdast-add-list-metadata)
    — Enhances the metadata of list and listItem nodes
*   [`mdast-comment-marker`](https://github.com/syntax-tree/mdast-comment-marker)
    — Parse a comment marker
*   [`mdast-util-compact`](https://github.com/syntax-tree/mdast-util-compact)
    — Make an MDAST tree compact
*   [`mdast-util-definitions`](https://github.com/syntax-tree/mdast-util-definitions)
    — Find definition nodes
*   [`mdast-flatten-listitem-paragraphs`](https://gitlab.com/staltz/mdast-flatten-listitem-paragraphs)
    — Flatten listItem and (nested) paragraph into one listItem node 
*   [`mdast-flatten-nested-lists`](https://gitlab.com/staltz/mdast-flatten-nested-lists)
    — Transforms an MDAST tree to avoid lists inside lists
*   [`mdast-util-heading-range`](https://github.com/syntax-tree/mdast-util-heading-range)
    — Markdown heading as ranges
*   [`mdast-util-heading-style`](https://github.com/syntax-tree/mdast-util-heading-style)
    — Get the style of a heading node
*   [`mdast-util-inject`](https://github.com/anandthakker/mdast-util-inject)
    — Inject a tree into another at a given heading
*   [`mdast-util-to-string`](https://github.com/syntax-tree/mdast-util-to-string)
    — Get the plain text content of a node
*   [`mdast-flatten-image-paragraphs`](https://gitlab.com/staltz/mdast-flatten-image-paragraphs)
    — Flatten paragraph and image into one image node 
*   [`mdast-move-images-to-root`](https://gitlab.com/staltz/mdast-move-images-to-root)
    — Moves image nodes up the tree until they are strict children of the root
*   [`mdast-normalize-headings`](https://github.com/eush77/mdast-normalize-headings)
    — Ensure at most one top-level heading is in the document
*   [`mdast-squeeze-paragraphs`](https://github.com/eush77/mdast-squeeze-paragraphs)
    — Remove empty paragraphs
*   [`mdast-util-toc`](https://github.com/BarryThePenguin/mdast-util-toc)
    — Generate a Table of Contents from a tree
*   [`mdast-util-to-hast`](https://github.com/syntax-tree/mdast-util-to-hast)
    — Transform MDAST to HAST
*   [`mdast-util-to-nlcst`](https://github.com/syntax-tree/mdast-util-to-nlcst)
    — Transform MDAST to NLCST
*   [`mdast-zone`](https://github.com/syntax-tree/mdast-zone)
    — HTML comments as ranges or markers

## Related

*   [remark][]
*   [unist][]
*   [nlcst][]
*   [vfile][]

## Contribute

**mdast** is built by people just like you!  Check out
[`contributing.md`][contributing] for ways to get started.

This project has a [Code of Conduct][coc].  By interacting with this repository,
organisation, or community you agree to abide by its terms.

Want to chat with the community and contributors?  Join us in [Gitter][chat]!

Have an idea for a cool new utility or tool?  That’s great!  If you want
feedback, help, or just to share it with the world you can do so by creating
an issue in the [`syntax-tree/ideas`][ideas] repository!

## Acknowledgments

The initial release of this project was authored by
[**@wooorm**](https://github.com/wooorm).

Special thanks to [**@eush77**](https://github.com/eush77) for their work,
ideas, and incredibly valuable feedback!

Thanks to
[**@anandthakker**](https://github.com/anandthakker),
[**@BarryThePenguin**](https://github.com/BarryThePenguin),
[**@izumin5210**](https://github.com/izumin5210),
[**@jasonLaster**](https://github.com/jasonLaster),
[**@justjake**](https://github.com/justjake),
[**@KyleAMathews**](https://github.com/KyleAMathews),
[**@Rokt33r**](https://github.com/Rokt33r),
[**@rhysd**](https://github.com/rhysd),
[**@Sarah-Seo**](https://github.com/Sarah-Seo),
[**@sethvincent**](https://github.com/sethvincent), and
[**@simov**](https://github.com/simov) for contributing commits since!

## License

[CC-BY-4.0][license] © [Titus Wormer][author]

<!-- Definitions -->

[logo]: https://cdn.rawgit.com/syntax-tree/mdast/6205835/logo.svg

[unist]: https://github.com/syntax-tree/unist

[remark]: https://github.com/wooorm/remark

[nlcst]: https://github.com/syntax-tree/nlcst

[vfile]: https://github.com/vfile/vfile

[releases]: https://github.com/syntax-tree/mdast/releases

[latest]: https://github.com/syntax-tree/mdast/releases/tag/2.2.0

[node]: https://github.com/syntax-tree/unist#node

[parent]: https://github.com/syntax-tree/unist#parent

[text]: https://github.com/syntax-tree/unist#text

[task-list]: https://help.github.com/articles/writing-on-github/#task-lists

[inlinecode]: #inlinecode

[code]: #code

[list]: #list

[listitem]: #listitem

[table]: #table

[tablerow]: #tablerow

[tablecell]: #tablecell

[definition]: #definition

[linkreference]: #linkreference

[imagereference]: #imagereference

[footnote]: #footnote

[footnotereference]: #footnotereference

[footnotedefinition]: #footnotedefinition

[frontmatter]: https://github.com/wooorm/remark-frontmatter

[contributing]: contributing.md

[coc]: code-of-conduct.md

[ideas]: https://github.com/syntax-tree/ideas

[chat]: https://gitter.im/wooorm/remark

[license]: https://creativecommons.org/licenses/by/4.0/

[author]: http://wooorm.com
