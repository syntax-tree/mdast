# ![MDAST][logo]

**M**ark**d**own **A**bstract **S**yntax **T**ree.

***

**MDAST** discloses markdown as an abstract syntax tree.  _Abstract_
means not all information is stored in this tree and an exact replica
of the original document cannot be re-created.  _Syntax Tree_ means syntax
**is** present in the tree, thus an exact syntactic document can be
re-created.

**MDAST** is a subset of [unist][], and implemented by [remark][].

This document describes version 2.0.0 of **MDAST**. [Changelog »][changelog].

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
[`InlineCode`][inlinecode] for code spans).  `Code` sports a language
tag (when using GitHub Flavoured Markdown fences with a flag, `null`
otherwise).

```idl
interface Code <: Text {
  type: "code";
  lang: string | null;
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

`List` ([`Parent`][parent]) contains [`ListItem`s][listitem].

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

## Related

*   [remark][]
*   [unist][]
*   [nlcst][]
*   [vfile][]

## License

MIT © [Titus Wormer](http://wooorm.com)

<!-- Definitions -->

[logo]: https://cdn.rawgit.com/wooorm/mdast/master/logo.svg

[unist]: https://github.com/wooorm/unist

[remark]: https://github.com/wooorm/remark

[nlcst]: https://github.com/wooorm/nlcst

[vfile]: https://github.com/wooorm/vfile

[changelog]: https://github.com/wooorm/mdast/releases

[node]: https://github.com/wooorm/unist#node

[parent]: https://github.com/wooorm/unist#parent

[text]: https://github.com/wooorm/unist#text

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
