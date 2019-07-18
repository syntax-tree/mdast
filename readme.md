# ![mdast][logo]

**M**ark**d**own **A**bstract **S**yntax **T**ree.

* * *

**mdast** is a specification for representing Markdown in a [syntax
tree][syntax-tree].
It implements the [**unist**][unist] spec.
It can represent several flavours of [Markdown][], such as [CommonMark][],
and [GitHub Flavored Markdown][gfm] extensions.

This document may not be released.
See [releases][] for released documents.
The latest released version is [`3.0.0`][latest].

## Table of Contents

*   [Introduction](#introduction)
    *   [Where this specification fits](#where-this-specification-fits)
*   [Nodes](#nodes)
    *   [Parent](#parent)
    *   [Literal](#literal)
    *   [Root](#root)
    *   [Paragraph](#paragraph)
    *   [Heading](#heading)
    *   [ThematicBreak](#thematicbreak)
    *   [Blockquote](#blockquote)
    *   [List](#list)
    *   [ListItem](#listitem)
    *   [Table](#table)
    *   [TableRow](#tablerow)
    *   [TableCell](#tablecell)
    *   [HTML](#html)
    *   [Code](#code)
    *   [YAML](#yaml)
    *   [Definition](#definition)
    *   [FootnoteDefinition](#footnotedefinition)
    *   [Text](#text)
    *   [Emphasis](#emphasis)
    *   [Strong](#strong)
    *   [Delete](#delete)
    *   [InlineCode](#inlinecode)
    *   [Break](#break)
    *   [Link](#link)
    *   [Image](#image)
    *   [LinkReference](#linkreference)
    *   [ImageReference](#imagereference)
    *   [Footnote](#footnote)
    *   [FootnoteReference](#footnotereference)
*   [Mixin](#mixin)
    *   [Resource](#resource)
    *   [Association](#association)
    *   [Reference](#reference)
    *   [Alternative](#alternative)
*   [Enumeration](#enumeration)
    *   [alignType](#aligntype)
    *   [referenceType](#referencetype)
*   [Content](#content)
    *   [TopLevelContent](#toplevelcontent)
    *   [BlockContent](#blockcontent)
    *   [FrontmatterContent](#frontmattercontent)
    *   [DefinitionContent](#definitioncontent)
    *   [ListContent](#listcontent)
    *   [TableContent](#tablecontent)
    *   [RowContent](#rowcontent)
    *   [PhrasingContent](#phrasingcontent)
    *   [StaticPhrasingContent](#staticphrasingcontent)
*   [Glossary](#glossary)
*   [List of Utilities](#list-of-utilities)
*   [References](#references)
*   [Security](#security)
*   [Contribute](#contribute)
*   [Acknowledgments](#acknowledgments)
*   [License](#license)

## Introduction

This document defines a format for representing [Markdown][] as an [abstract
syntax tree][syntax-tree].
Development of mdast started in July 2014, in [**remark**][remark], before
[unist][] existed.
This specification is written in a [Web IDL][webidl]-like grammar.

### Where this specification fits

mdast extends [unist][], a format for syntax trees, to benefit from its
[ecosystem of utilities][utilities].

mdast relates to [JavaScript][] in that it has a rich [ecosystem of
utilities][list-of-utilities] for working with compliant syntax trees in
JavaScript.
However, mdast is not limited to JavaScript and can be used in other programming
languages.

mdast relates to the [unified][] and [remark][] projects in that mdast syntax
trees are used throughout their ecosystems.

## Nodes

### `Parent`

```idl
interface Parent <: UnistParent {
  children: [Content]
}
```

**Parent** ([**UnistParent**][dfn-unist-parent]) represents a node in mdast
containing other nodes (said to be [*children*][term-child]).

Its content is limited to only other mdast [**content**][dfn-content].

### `Literal`

```idl
interface Literal <: UnistLiteral {
  value: string
}
```

**Literal** ([**UnistLiteral**][dfn-unist-literal]) represents a node in mdast
containing a value.

Its `value` field is a `string`.

### `Root`

```idl
interface Root <: Parent {
  type: "root"
}
```

**Root** ([**Parent**][dfn-parent]) represents a document.

**Root** can be used as the [*root*][term-root] of a [*tree*][term-tree], never
as a [*child*][term-child].
Its content model is not limited to [**top-level**][dfn-top-level-content]
content, but can contain any [**content**][dfn-content] with the restriction
that all content must be of the same category.

### `Paragraph`

```idl
interface Paragraph <: Parent {
  type: "paragraph"
  children: [PhrasingContent]
}
```

**Paragraph** ([**Parent**][dfn-parent]) represents a unit of discourse dealing
with a particular point or idea.

**Paragraph** can be used where [**block**][dfn-block-content] content is
expected.
Its content model is [**phrasing**][dfn-phrasing-content] content.

For example, the following Markdown:

```markdown
Alpha bravo charlie.
```

Yields:

```js
{
  type: 'paragraph',
  children: [{type: 'text', value: 'Alpha bravo charlie.'}]
}
```

### `Heading`

```idl
interface Heading <: Parent {
  type: "heading"
  depth: 1 <= number <= 6
  children: [PhrasingContent]
}
```

**Heading** ([**Parent**][dfn-parent]) represents a heading of a section.

**Heading** can be used where [**block**][dfn-block-content] content is
expected.
Its content model is [**phrasing**][dfn-phrasing-content] content.

A `depth` field must be present.
A value of `1` is said to be the highest rank and `6` the lowest.

For example, the following Markdown:

```markdown
# Alpha
```

Yields:

```js
{
  type: 'heading',
  depth: 1,
  children: [{type: 'text', value: 'Alpha'}]
}
```

### `ThematicBreak`

```idl
interface ThematicBreak <: Node {
  type: "thematicBreak"
}
```

**ThematicBreak** ([**Node**][dfn-node]) represents a thematic break, such as a
scene change in a story, a transition to another topic, or a new document.

**ThematicBreak** can be used where [**block**][dfn-block-content] content is
expected.
It has no content model.

For example, the following Markdown:

```markdown
***
```

Yields:

```js
{type: 'thematicBreak'}
```

### `Blockquote`

```idl
interface Blockquote <: Parent {
  type: "blockquote"
  children: [BlockContent]
}
```

**Blockquote** ([**Parent**][dfn-parent]) represents a section quoted from
somewhere else.

**Blockquote** can be used where [**block**][dfn-block-content] content is
expected.
Its content model is also [**block**][dfn-block-content] content.

For example, the following Markdown:

```markdown
> Alpha bravo charlie.
```

Yields:

```js
{
  type: 'blockquote',
  children: [{
    type: 'paragraph',
    children: [{type: 'text', value: 'Alpha bravo charlie.'}]
  }]
}
```

### `List`

```idl
interface List <: Parent {
  type: "list"
  ordered: boolean?
  start: number?
  spread: boolean?
  children: [ListContent]
}
```

**List** ([**Parent**][dfn-parent]) represents a list of items.

**List** can be used where [**block**][dfn-block-content] content is expected.
Its content model is [**list**][dfn-list-content] content.

An `ordered` field can be present.
It represents that the items have been intentionally ordered (when true), or
that the order of items is not important (when `false` or not present).

If the `ordered` field is `true`, a `start` field can be present.
It represents the starting number of the node.

A `spread` field can be present.
It represents that any of its items is separated by a blank line from its
[siblings][term-sibling] (when `true`), or not (when `false` or not present).

For example, the following Markdown:

```markdown
1. [x] foo
```

Yields:

```js
{
  type: 'list',
  ordered: true,
  start: 1,
  spread: false,
  children: [{
    type: 'listItem',
    checked: true,
    spread: false,
    children: [{
      type: 'paragraph',
      children: [{type: 'text', value: 'foo'}]
    }]
  }]
}
```

### `ListItem`

```idl
interface ListItem <: Parent {
  type: "listItem"
  checked: boolean?
  spread: boolean?
  children: [BlockContent]
}
```

**ListItem** ([**Parent**][dfn-parent]) represents an item in a
[**List**][dfn-list].

**ListItem** can be used where [**list**][dfn-list-content] content is expected.
Its content model is [**block**][dfn-block-content] content.

A `checked` field can be present.
It represents whether the item is done (when `true`), not done (when `false`),
or indeterminate or not applicable (when `null` or not present).

A `spread` field can be present.
It represents that the item contains two or more [*children*][term-child]
separated by a blank line (when `true`), or not (when `false` or not present).

For example, the following Markdown:

```markdown
* [x] bar
```

Yields:

```js
{
  type: 'listItem',
  checked: true,
  spread: false,
  children: [{
    type: 'paragraph',
    children: [{type: 'text', value: 'bar'}]
  }]
}
```

### `Table`

```idl
interface Table <: Parent {
  type: "table"
  align: [alignType]?
  children: [TableContent]
}
```

**Table** ([**Parent**][dfn-parent]) represents two-dimensional data.

**Table** can be used where [**block**][dfn-block-content] content is expected.
Its content model is [**table**][dfn-table-content] content.

The [*head*][term-head] of the node represents the labels of the columns.

An `align` field can be present.
If present, it must be a list of [**alignType**s][dfn-enum-align-type].
It represents how cells in columns are aligned.

For example, the following Markdown:

```markdown
| foo | bar |
| :-- | :-: |
| baz | qux |
```

Yields:

```js
{
  type: 'table',
  align: ['left', 'center'],
  children: [
    {
      type: 'tableRow',
      children: [
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'foo'}]
        },
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'bar'}]
        }
      ]
    },
    {
      type: 'tableRow',
      children: [
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'baz'}]
        },
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'qux'}]
        }
      ]
    }
  ]
}
```

### `TableRow`

```idl
interface TableRow <: Parent {
  type: "tableRow"
  children: [RowContent]
}
```

**TableRow** ([**Parent**][dfn-parent]) represents a row of cells in a table.

**TableRow** can be used where [**table**][dfn-table-content] content is
expected.
Its content model is [**row**][dfn-row-content] content.

If the node is a [*head*][term-head], it represents the labels of the columns
for its parent [**Table**][dfn-table].

For an example, see [**Table**][dfn-table].

### `TableCell`

```idl
interface TableCell <: Parent {
  type: "tableCell"
  children: [PhrasingContent]
}
```

**TableCell** ([**Parent**][dfn-parent]) represents a header cell in a
[**Table**][dfn-table], if its parent is a [*head*][term-head], or a data
cell otherwise.

**TableCell** can be used where [**row**][dfn-row-content] content is expected.
Its content model is [**phrasing**][dfn-phrasing-content] content.

For an example, see [**Table**][dfn-table].

### `HTML`

```idl
interface HTML <: Literal {
  type: "html"
}
```

**HTML** ([**Literal**][dfn-literal]) represents a fragment of raw [HTML][].

**HTML** can be used where [**block**][dfn-block-content] or
[**phrasing**][dfn-phrasing-content] content is expected.
Its content is represented by its `value` field.

For example, the following Markdown:

```markdown
<div>
```

Yields:

```js
{type: 'html', value: '<div>'}
```

### `Code`

```idl
interface Code <: Literal {
  type: "code"
  lang: string?
  meta: string?
}
```

**Code** ([**Literal**][dfn-literal]) represents a block of preformatted text,
such as ASCII art or computer code.

**Code** can be used where [**block**][dfn-block-content] content is expected.
Its content is represented by its `value` field.

This node relates to the [**phrasing**][dfn-phrasing-content] content concept
[**InlineCode**][dfn-inline-code].

A `lang` field can be present.
It represents the language of computer code being marked up.

If the `lang` field is present, a `meta` field can be present.
It represents custom information relating to the node.

For example, the following Markdown:

```markdown
    foo()
```

Yields:

```js
{
  type: 'code',
  lang: null,
  meta: null,
  value: 'foo()'
}
```

And the following Markdown:

````markdown
```js highlight-line="2"
foo()
bar()
baz()
```
````

Yields:

```js
{
  type: 'code',
  lang: 'javascript',
  meta: 'highlight-line="2"',
  value: 'foo()\nbar()\nbaz()'
}
```

### `YAML`

```idl
interface YAML <: Literal {
  type: "yaml"
}
```

**YAML** ([**Literal**][dfn-literal]) represents a collection of metadata for
the document in the [YAML][] data serialisation language.

**YAML** can be used where [**frontmatter**][dfn-frontmatter-content] content is
expected.
Its content is represented by its `value` field.

For example, the following Markdown:

```markdown
---
foo: bar
---
```

Yields:

```js
{type: 'yaml', value: 'foo: bar'}
```

### `Definition`

```idl
interface Definition <: Node {
  type: "definition"
}

Definition includes Association
Definition includes Resource
```

**Definition** ([**Node**][dfn-node]) represents a resource.

**Definition** can be used where [**definition**][dfn-definition-content]
content is expected.
It has no content model.

**Definition** includes the mixins [**Association**][dfn-mxn-association] and
[**Resource**][dfn-mxn-resource].

**Definition** should be associated with
[**LinkReferences**][dfn-link-reference] and
[**ImageReferences**][dfn-image-reference].

For example, the following Markdown:

```markdown
[Alpha]: https://example.com
```

Yields:

```js
{
  type: 'definition',
  identifier: 'alpha',
  label: 'Alpha',
  url: 'https://example.com',
  title: null
}
```

### `FootnoteDefinition`

```idl
interface FootnoteDefinition <: Parent {
  type: "footnoteDefinition"
  children: [BlockContent]
}

FootnoteDefinition includes Association
```

**FootnoteDefinition** ([**Parent**][dfn-parent]) represents content relating
to the document that is outside its flow.

**FootnoteDefinition** can be used where
[**definition**][dfn-definition-content] content is expected.
Its content model is [**block**][dfn-block-content] content.

**FootnoteDefinition** includes the mixin
[**Association**][dfn-mxn-association].

**FootnoteDefinition** should be associated with
[**FootnoteReferences**][dfn-footnote-reference].

For example, the following Markdown:

```markdown
[^alpha]: bravo and charlie.
```

Yields:

```js
{
  type: 'footnoteDefinition',
  identifier: 'alpha',
  label: 'alpha',
  children: [{
    type: 'paragraph',
    children: [{type: 'text', value: 'bravo and charlie.'}]
  }]
}
```

### `Text`

```idl
interface Text <: Literal {
  type: "text"
}
```

**Text** ([**Literal**][dfn-literal]) represents everything that is just text.

**Text** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content is represented by its `value` field.

For example, the following Markdown:

```markdown
Alpha bravo charlie.
```

Yields:

```js
{type: 'text', value: 'Alpha bravo charlie.'}
```

### `Emphasis`

```idl
interface Emphasis <: Parent {
  type: "emphasis"
  children: [PhrasingContent]
}
```

**Emphasis** ([**Parent**][dfn-parent]) represents stress emphasis of its
contents.

**Emphasis** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is also [**phrasing**][dfn-phrasing-content] content.

For example, the following Markdown:

```markdown
*alpha* _bravo_
```

Yields:

```js
{
  type: 'paragraph',
  children: [
    {
      type: 'emphasis',
      children: [{type: 'text', value: 'alpha'}]
    },
    {type: 'text', value: ' '},
    {
      type: 'emphasis',
      children: [{type: 'text', value: 'bravo'}]
    }
  ]
}
```

### `Strong`

```idl
interface Strong <: Parent {
  type: "strong"
  children: [PhrasingContent]
}
```

**Strong** ([**Parent**][dfn-parent]) represents strong importance, seriousness,
or urgency for its contents.

**Strong** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is also [**phrasing**][dfn-phrasing-content] content.

For example, the following Markdown:

```markdown
**alpha** __bravo__
```

Yields:

```js
{
  type: 'paragraph',
  children: [
    {
      type: 'strong',
      children: [{type: 'text', value: 'alpha'}]
    },
    {type: 'text', value: ' '},
    {
      type: 'strong',
      children: [{type: 'text', value: 'bravo'}]
    }
  ]
}
```

### `Delete`

```idl
interface Delete <: Parent {
  type: "delete"
  children: [PhrasingContent]
}
```

**Delete** ([**Parent**][dfn-parent]) represents contents that are no longer
accurate or no longer relevant.

**Delete** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is also [**phrasing**][dfn-phrasing-content] content.

For example, the following Markdown:

```markdown
~~alpha~~
```

Yields:

```js
{
  type: 'delete',
  children: [{type: 'text', value: 'alpha'}]
}
```

### `InlineCode`

```idl
interface InlineCode <: Literal {
  type: "inlineCode"
}
```

**InlineCode** ([**Literal**][dfn-literal]) represents a fragment of computer
code, such as a file name, computer program, or anything a computer could parse.

**InlineCode** can be used where [**phrasing**][dfn-phrasing-content] content
is expected.
Its content is represented by its `value` field.

This node relates to the [**block**][dfn-block-content] content concept
[**Code**][dfn-code].

For example, the following Markdown:

```markdown
`foo()`
```

Yields:

```js
{type: 'inlineCode', value: 'foo()'}
```

### `Break`

```idl
interface Break <: Node {
  type: "break"
}
```

**Break** ([**Node**][dfn-node]) represents a line break, such as in poems or
addresses.

**Break** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
It has no content model.

For example, the following Markdown:

```markdown
foo··
bar
```

Yields:

```js
{
  type: 'paragraph',
  children: [
    {type: 'text', value: 'foo'},
    {type: 'break'},
    {type: 'text', value: 'bar'}
  ]
}
```

### `Link`

```idl
interface Link <: Parent {
  type: "link"
  children: [StaticPhrasingContent]
}

Link includes Resource
```

**Link** ([**Parent**][dfn-parent]) represents a hyperlink.

**Link** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is [**static phrasing**][dfn-static-phrasing-content] content.

**Link** includes the mixin [**Resource**][dfn-mxn-resource].

For example, the following Markdown:

```markdown
[alpha](https://example.com "bravo")
```

Yields:

```js
{
  type: 'link',
  url: 'https://example.com',
  title: 'bravo',
  children: [{type: 'text', value: 'alpha'}]
}
```

### `Image`

```idl
interface Image <: Node {
  type: "image"
}

Image includes Resource
Image includes Alternative
```

**Image** ([**Node**][dfn-node]) represents an image.

**Image** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
It has no content model, but is described by its `alt` field.

**Image** includes the mixins [**Resource**][dfn-mxn-resource] and
[**Alternative**][dfn-mxn-alternative].

For example, the following Markdown:

```markdown
![alpha](https://example.com/favicon.ico "bravo")
```

Yields:

```js
{
  type: 'image',
  url: 'https://example.com/favicon.ico',
  title: 'bravo',
  alt: 'alpha'
}
```

### `LinkReference`

```idl
interface LinkReference <: Parent {
  type: "linkReference"
  children: [StaticPhrasingContent]
}

LinkReference includes Reference
```

**LinkReference** ([**Parent**][dfn-parent]) represents a hyperlink through
association, or its original source if there is no association.

**LinkReference** can be used where [**phrasing**][dfn-phrasing-content] content
is expected.
Its content model is [**static phrasing**][dfn-static-phrasing-content] content.

**LinkReference** includes the mixin [**Reference**][dfn-mxn-reference].

**LinkReferences** should be associated with a [**Definition**][dfn-definition].

For example, the following Markdown:

```markdown
[alpha][Bravo]
```

Yields:

```js
{
  type: 'linkReference',
  identifier: 'bravo',
  label: 'Bravo',
  referenceType: 'full',
  children: [{type: 'text', value: 'alpha'}]
}
```

### `ImageReference`

```idl
interface ImageReference <: Node {
  type: "imageReference"
}

ImageReference includes Reference
ImageReference includes Alternative
```

**ImageReference** ([**Node**][dfn-node]) represents an image through
association, or its original source if there is no association.

**ImageReference** can be used where [**phrasing**][dfn-phrasing-content]
content is expected.
It has no content model, but is described by its `alt` field.

**ImageReference** includes the mixins [**Reference**][dfn-mxn-reference] and
[**Alternative**][dfn-mxn-alternative].

**ImageReference** should be associated with a [**Definition**][dfn-definition].

For example, the following Markdown:

```markdown
![alpha][bravo]
```

Yields:

```js
{
  type: 'imageReference',
  identifier: 'bravo',
  label: 'bravo',
  referenceType: 'full',
  alt: 'alpha'
}
```

### `Footnote`

```idl
interface Footnote <: Parent {
  type: "footnote"
  children: [PhrasingContent]
}
```

**Footnote** ([**Parent**][dfn-parent]) represents content relating to the
document that is outside its flow.

**Footnote** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is also [**phrasing**][dfn-phrasing-content] content.

For example, the following Markdown:

```markdown
[^alpha bravo]
```

Yields:

```js
{
  type: 'footnote',
  children: [{type: 'text', value: 'alpha bravo'}]
}
```

### `FootnoteReference`

```idl
interface FootnoteReference <: Node {
  type: "footnoteReference"
}

FootnoteReference includes Association
```

**FootnoteReference** ([**Node**][dfn-node]) represents a marker through
association.

**FootnoteReference** can be used where [**phrasing**][dfn-phrasing-content]
content is expected.
It has no content model.

**FootnoteReference** includes the mixin [**Association**][dfn-mxn-association].

**FootnoteReference** should be associated with a
[**FootnoteDefinition**][dfn-footnote-definition].

For example, the following Markdown:

```markdown
[^alpha]
```

Yields:

```js
{
  type: 'footnoteReference',
  identifier: 'alpha',
  label: 'alpha'
}
```

## Mixin

### `Resource`

```idl
interface mixin Resource {
  url: string
  title: string?
}
```

**Resource** represents a reference to resource.

A `url` field must be present.
It represents a URL to the referenced resource.

A `title` field can be present.
It represents  advisory information for the resource, such as would be
appropriate for a tooltip.

### `Association`

```idl
interface mixin Association {
  identifier: string
  label: string?
}
```

**Association** represents an internal relation from one node to another.

An `identifier` field must be present.
It can match an `identifier` field on another node.

A `label` field can be present.
It represents the original value of the normalised `identifier` field.

Whether the value of `identifier` is expected to be a unique identifier or not
depends on the type of node including the **Association**.
An example of this is that `identifier` on [**Definition**][dfn-definition]
should be a unique identifier, whereas multiple
[**LinkReference**][dfn-link-reference]s can have the same `identifier` and be
associated with one definition.

### `Reference`

```idl
interface mixin Reference {
  referenceType: string
}

Reference includes Association
```

**Reference** represents a marker that is [**associated**][dfn-mxn-association]
to another node.

A `referenceType` field must be present.
Its value must be a [**referenceType**][dfn-enum-reference-type].
It represents the explicitness of the reference.

### `Alternative`

```idl
interface mixin Alternative {
  alt: string?
}
```

**Alternative** represents a node with a fallback

An `alt` field should be present.
It represents equivalent content for environments that cannot represent the
node as intended.

## Enumeration

### `alignType`

```idl
enum alignType {
  "left" | "right" | "center" | null
}
```

**alignType** represents how phrasing content is aligned.

*   **left**: See the [`left`][css-left] value of the `text-align` CSS property
*   **right**: See the [`right`][css-right] value of the `text-align` CSS
    property
*   **center**: See the [`center`][css-center] value of the `text-align` CSS
    property
*   **null**: phrasing content is aligned as defined by the host environment

### `referenceType`

```idl
enum referenceType {
  "shortcut" | "collapsed" | "full"
}
```

**referenceType** represents the explicitness of a reference.

*   **shortcut**: the reference is implicit, its identifier inferred from its
    content
*   **collapsed**: the reference is explicit, its identifier inferred from its
    content
*   **full**: the reference is explicit, its identifier explicitly set

## `Content`

```idl
type Content =
  TopLevelContent | ListContent | TableContent | RowContent | PhrasingContent
```

Each node in mdast falls into one or more categories of **Content** that group
nodes with similar characteristics together.

### `TopLevelContent`

```idl
type TopLevelContent = BlockContent | FrontmatterContent | DefinitionContent
```

**Top-level** content represent the sections of document (**block** content),
and metadata such as frontmatter and definitions.

### `BlockContent`

```idl
type BlockContent =
  Paragraph | Heading | ThematicBreak | Blockquote | List | Table | HTML | Code
```

**Block** content represent the sections of document.

### `FrontmatterContent`

```idl
type FrontmatterContent = YAML
```

**Frontmatter** content represent out-of-band information about the document.

If frontmatter is present, it must be limited to one node in the
[*tree*][term-tree], and can only exist as a [*head*][term-head].

### `DefinitionContent`

```idl
type DefinitionContent = Definition | FootnoteDefinition
```

**Definition** content represents out-of-band information that typically
affects the document through [**Association**][dfn-mxn-association].

### `ListContent`

```idl
type ListContent = ListItem
```

**List** content represent the items in a list.

### `TableContent`

```idl
type TableContent = TableRow
```

**Table** content represent the rows in a table.

### `RowContent`

```idl
type RowContent = TableCell
```

**Row** content represent the cells in a row.

### `PhrasingContent`

```idl
type PhrasingContent = StaticPhrasingContent | Link | LinkReference
```

**Phrasing** content represent the text in a document, and its markup.

### `StaticPhrasingContent`

```idl
type StaticPhrasingContent =
  Text | Emphasis | Strong | Delete | HTML | InlineCode | Break | Image |
  ImageReference | Footnote | FootnoteReference
```

**StaticPhrasing** content represent the text in a document, and its
markup, that is not intended for user interaction.

## Glossary

See the [unist glossary][glossary].

## List of Utilities

See the [unist list of utilities][utilities] for more utilities.

<!--lint disable list-item-spacing-->

*   [`mdast-util-assert`](https://github.com/syntax-tree/mdast-util-assert)
    — Assert nodes
*   [`mdast-add-list-metadata`](https://gitlab.com/staltz/mdast-add-list-metadata)
    — Enhances the metadata of list and listItem nodes
*   [`mdast-builder`](https://github.com/mike-north/mdast-builder)
    — Build mdast structures with composable functions
*   [`mdast-comment-marker`](https://github.com/syntax-tree/mdast-comment-marker)
    — Parse a comment marker
*   [`mdast-util-compact`](https://github.com/syntax-tree/mdast-util-compact)
    — Make a tree compact
*   [`mdast-util-definitions`](https://github.com/syntax-tree/mdast-util-definitions)
    — Find definition nodes
*   [`mdast-flatten-listitem-paragraphs`](https://gitlab.com/staltz/mdast-flatten-listitem-paragraphs)
    — Flatten listItem and (nested) paragraph into one listItem node
*   [`mdast-flatten-nested-lists`](https://gitlab.com/staltz/mdast-flatten-nested-lists)
    — Transforms a tree to avoid lists inside lists
*   [`mdast-util-heading-range`](https://github.com/syntax-tree/mdast-util-heading-range)
    — Markdown heading as ranges
*   [`mdast-util-heading-style`](https://github.com/syntax-tree/mdast-util-heading-style)
    — Get the style of a heading node
*   [`mdast-util-inject`](https://github.com/anandthakker/mdast-util-inject)
    — Inject a tree into another at a given heading
*   [`mdast-util-phrasing`](https://github.com/syntax-tree/mdast-util-phrasing)
    — Check if a node is phrasing content
*   [`mdast-util-to-string`](https://github.com/syntax-tree/mdast-util-to-string)
    — Get the plain text content of a node
*   [`mdast-flatten-image-paragraphs`](https://gitlab.com/staltz/mdast-flatten-image-paragraphs)
    — Flatten paragraph and image into one image node
*   [`mdast-move-images-to-root`](https://gitlab.com/staltz/mdast-move-images-to-root)
    — Moves image nodes up the tree until they are strict children of the root
*   [`mdast-normalize-headings`](https://github.com/syntax-tree/mdast-normalize-headings)
    — Ensure at most one top-level heading is in the document
*   [`mdast-squeeze-paragraphs`](https://github.com/syntax-tree/mdast-squeeze-paragraphs)
    — Remove empty paragraphs
*   [`mdast-util-toc`](https://github.com/syntax-tree/mdast-util-toc)
    — Generate a Table of Contents from a tree
*   [`mdast-util-to-hast`](https://github.com/syntax-tree/mdast-util-to-hast)
    — Transform to hast
*   [`mdast-util-to-nlcst`](https://github.com/syntax-tree/mdast-util-to-nlcst)
    — Transform to nlcst
*   [`mdast-zone`](https://github.com/syntax-tree/mdast-zone)
    — HTML comments as ranges or markers

## References

*   **unist**:
    [Universal Syntax Tree][unist].
    T. Wormer; et al.
*   **Markdown**:
    [Markdown][].
    J. Gruber.
*   **CommonMark**:
    [CommonMark][].
    J. MacFarlane; et al.
*   **GFM**:
    [GitHub Flavored Markdown][gfm].
    GitHub.
*   **HTML**:
    [HTML Standard][html],
    A. van Kesteren; et al.
    WHATWG.
*   **CSSTEXT**:
    [CSS Text][css-text],
    CSS Text, E. Etemad, K. Ishii.
    W3C.
*   **JavaScript**:
    [ECMAScript Language Specification][javascript].
    Ecma International.
*   **YAML**:
    [YAML Ain’t Markup Language][yaml],
    O. Ben-Kiki, C. Evans, I. döt Net.
*   **Web IDL**:
    [Web IDL][webidl],
    C. McCormack.
    W3C.

## Security

As mdast can contain HTML and be used to represent HTML, and improper use of
HTML can open you up to a [cross-site scripting (XSS)][xss] attack, improper use
of mdast is also unsafe.
When transforming to HTML (typically through [**hast**][hast]), always be
careful with user input and use [`hast-util-santize`][sanitize] to make the hast
tree safe.

## Contribute

See [`contributing.md`][contributing] in [`syntax-tree/.github`][health] for
ways to get started.
See [`support.md`][support] for ways to get help.
Ideas for new utilities and tools can be posted in [`syntax-tree/ideas`][ideas].

A curated list of awesome syntax-tree, unist, hast, mdast, and nlcst resources
can be found in [awesome syntax-tree][awesome].

This project has a [Code of Conduct][coc].
By interacting with this repository, organisation, or community you agree to
abide by its terms.

## Acknowledgments

The initial release of this project was authored by
[**@wooorm**](https://github.com/wooorm).

Special thanks to [**@eush77**](https://github.com/eush77) for their work,
ideas, and incredibly valuable feedback!

Thanks to
[**@anandthakker**](https://github.com/anandthakker),
[**@arobase-che**](https://github.com/arobase-che),
[**@BarryThePenguin**](https://github.com/BarryThePenguin),
[**@chinesedfan**](https://github.com/chinesedfan),
[**@ChristianMurphy**](https://github.com/ChristianMurphy),
[**@craftzdog**](https://github.com/craftzdog),
[**@d4rekanguok**](https://github.com/d4rekanguok),
[**@detj**](https://github.com/detj),
[**@dominictarr**](https://github.com/dominictarr),
[**@gkatsev**](https://github.com/gkatsev),
[**@Hamms**](https://github.com/Hamms),
[**@Hypercubed**](https://github.com/Hypercubed),
[**@ikatyang**](https://github.com/ikatyang),
[**@izumin5210**](https://github.com/izumin5210),
[**@jasonLaster**](https://github.com/jasonLaster),
[**@Justineo**](https://github.com/Justineo),
[**@justjake**](https://github.com/justjake),
[**@KyleAMathews**](https://github.com/KyleAMathews),
[**@laysent**](https://github.com/laysent),
[**@macklinu**](https://github.com/macklinu),
[**@mike-north**](https://github.com/mike-north),
[**@Murderlon**](https://github.com/Murderlon),
[**@nevik**](https://github.com/nevik),
[**@Rokt33r**](https://github.com/Rokt33r),
[**@rhysd**](https://github.com/rhysd),
[**@rubys**](https://github.com/rubys),
[**@Sarah-Seo**](https://github.com/Sarah-Seo),
[**@sethvincent**](https://github.com/sethvincent),
[**@silvenon**](https://github.com/silvenon),
[**@simov**](https://github.com/simov),
[**@staltz**](https://github.com/staltz),
[**@stefanprobst**](https://github.com/stefanprobst),
[**@tmcw**](https://github.com/tmcw),
and [**@vhf**](https://github.com/vhf)
for contributing to mdast and related
projects!

## License

[CC-BY-4.0][license] © [Titus Wormer][author]

<!-- Definitions -->

[health]: https://github.com/syntax-tree/.github

[contributing]: https://github.com/syntax-tree/.github/blob/master/contributing.md

[support]: https://github.com/syntax-tree/.github/blob/master/support.md

[coc]: https://github.com/syntax-tree/.github/blob/master/code-of-conduct.md

[awesome]: https://github.com/syntax-tree/awesome-syntax-tree

[ideas]: https://github.com/syntax-tree/ideas

[license]: https://creativecommons.org/licenses/by/4.0/

[author]: https://wooorm.com

[logo]: https://raw.githubusercontent.com/syntax-tree/mdast/79672c0/logo.svg?sanitize=true

[releases]: https://github.com/syntax-tree/mdast/releases

[latest]: https://github.com/syntax-tree/mdast/releases/tag/3.0.0

[dfn-node]: https://github.com/syntax-tree/unist#node

[dfn-unist-parent]: https://github.com/syntax-tree/unist#parent

[dfn-unist-literal]: https://github.com/syntax-tree/unist#literal

[dfn-parent]: #parent

[dfn-literal]: #literal

[dfn-code]: #code

[dfn-inline-code]: #inlinecode

[dfn-list]: #list

[dfn-table]: #table

[dfn-link-reference]: #linkreference

[dfn-image-reference]: #imagereference

[dfn-footnote-reference]: #footnotereference

[dfn-definition]: #definition

[dfn-footnote-definition]: #footnotedefinition

[term-tree]: https://github.com/syntax-tree/unist#tree

[term-child]: https://github.com/syntax-tree/unist#child

[term-sibling]: https://github.com/syntax-tree/unist#sibling

[term-root]: https://github.com/syntax-tree/unist#root

[term-head]: https://github.com/syntax-tree/unist#head

[dfn-mxn-resource]: #resource

[dfn-mxn-association]: #association

[dfn-mxn-reference]: #reference

[dfn-mxn-alternative]: #alternative

[dfn-enum-align-type]: #aligntype

[dfn-enum-reference-type]: #referencetype

[dfn-content]: #content

[dfn-top-level-content]: #toplevelcontent

[dfn-block-content]: #blockcontent

[dfn-frontmatter-content]: #frontmattercontent

[dfn-definition-content]: #frontmattercontent

[dfn-list-content]: #listcontent

[dfn-table-content]: #tablecontent

[dfn-row-content]: #rowcontent

[dfn-phrasing-content]: #phrasingcontent

[dfn-static-phrasing-content]: #staticphrasingcontent

[list-of-utilities]: #list-of-utilities

[unist]: https://github.com/syntax-tree/unist

[syntax-tree]: https://github.com/syntax-tree/unist#syntax-tree

[yaml]: https://yaml.org

[html]: https://html.spec.whatwg.org/multipage/

[css-text]: https://drafts.csswg.org/css-text/

[css-left]: https://drafts.csswg.org/css-text/#valdef-text-align-left

[css-right]: https://drafts.csswg.org/css-text/#valdef-text-align-right

[css-center]: https://drafts.csswg.org/css-text/#valdef-text-align-center

[javascript]: https://www.ecma-international.org/ecma-262/9.0/index.html

[webidl]: https://heycam.github.io/webidl/

[markdown]: https://daringfireball.net/projects/markdown/

[commonmark]: https://commonmark.org

[gfm]: https://github.github.com/gfm/

[glossary]: https://github.com/syntax-tree/unist#glossary

[utilities]: https://github.com/syntax-tree/unist#list-of-utilities

[unified]: https://github.com/unifiedjs/unified

[remark]: https://github.com/remarkjs/remark

[xss]: https://en.wikipedia.org/wiki/Cross-site_scripting

[hast]: https://github.com/syntax-tree/hast

[sanitize]: https://github.com/syntax-tree/hast-util-sanitize
