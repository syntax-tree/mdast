# ![mdast][logo]

**M**ark**d**own **A**bstract **S**yntax **T**ree.

***

**mdast** is a specification for representing markdown in a [syntax
tree][syntax-tree].
It implements **[unist][]**.
It can represent several flavours of [markdown][], such as [CommonMark][]
and [GitHub Flavored Markdown][gfm].

This document may not be released.
See [releases][] for released documents.
The latest released version is [`4.0.0`][latest].

## Contents

*   [Introduction](#introduction)
    *   [Where this specification fits](#where-this-specification-fits)
*   [Types](#types)
*   [Nodes](#nodes)
    *   [`Parent`](#parent)
    *   [`Literal`](#literal)
    *   [`Root`](#root)
    *   [`Paragraph`](#paragraph)
    *   [`Heading`](#heading)
    *   [`ThematicBreak`](#thematicbreak)
    *   [`Blockquote`](#blockquote)
    *   [`List`](#list)
    *   [`ListItem`](#listitem)
    *   [`HTML`](#html)
    *   [`Code`](#code)
    *   [`Definition`](#definition)
    *   [`Text`](#text)
    *   [`Emphasis`](#emphasis)
    *   [`Strong`](#strong)
    *   [`InlineCode`](#inlinecode)
    *   [`Break`](#break)
    *   [`Link`](#link)
    *   [`Image`](#image)
    *   [`LinkReference`](#linkreference)
    *   [`ImageReference`](#imagereference)
*   [Mixin](#mixin)
    *   [`Resource`](#resource)
    *   [`Association`](#association)
    *   [`Reference`](#reference)
    *   [`Alternative`](#alternative)
*   [Enumeration](#enumeration)
    *   [`referenceType`](#referencetype)
*   [Content model](#content-model)
    *   [`FlowContent`](#flowcontent)
    *   [`Content`](#content)
    *   [`ListContent`](#listcontent)
    *   [`PhrasingContent`](#phrasingcontent)
    *   [`StaticPhrasingContent`](#staticphrasingcontent)
    *   [`TransparentContent`](#transparentcontent)
*   [Extensions](#extensions)
    *   [GFM](#gfm)
    *   [Frontmatter](#frontmatter)
    *   [Footnotes](#footnotes)
    *   [MDX](#mdx)
*   [Glossary](#glossary)
*   [List of utilities](#list-of-utilities)
*   [References](#references)
*   [Security](#security)
*   [Related](#related)
*   [Contribute](#contribute)
*   [Acknowledgments](#acknowledgments)
*   [License](#license)

## Introduction

This document defines a format for representing [markdown][] as an [abstract
syntax tree][syntax-tree].
Development of mdast started in July 2014, in **[remark][]**, before [unist][]
existed.
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

## Types

If you are using TypeScript, you can use the unist types by installing them
with npm:

```sh
npm install @types/mdast
```

## Nodes

### `Parent`

```idl
interface Parent <: UnistParent {
  children: [MdastContent]
}
```

**Parent** ([**UnistParent**][dfn-unist-parent]) represents an abstract
interface in mdast containing other nodes (said to be [*children*][term-child]).

Its content is limited to only other [**mdast content**][dfn-mdast-content].

### `Literal`

```idl
interface Literal <: UnistLiteral {
  value: string
}
```

**Literal** ([**UnistLiteral**][dfn-unist-literal]) represents an abstract
interface in mdast containing a value.

Its `value` field is a `string`.

### `Root`

```idl
interface Root <: Parent {
  type: 'root'
}
```

**Root** ([**Parent**][dfn-parent]) represents a document.

**Root** can be used as the [*root*][term-root] of a [*tree*][term-tree], never
as a [*child*][term-child].
Its content model is **not** limited to [**flow**][dfn-flow-content] content,
but instead can contain any [**mdast content**][dfn-mdast-content] with the
restriction that all content must be of the same category.

### `Paragraph`

```idl
interface Paragraph <: Parent {
  type: 'paragraph'
  children: [PhrasingContent]
}
```

**Paragraph** ([**Parent**][dfn-parent]) represents a unit of discourse dealing
with a particular point or idea.

**Paragraph** can be used where [**content**][dfn-content] is expected.
Its content model is [**phrasing**][dfn-phrasing-content] content.

For example, the following markdown:

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
  type: 'heading'
  depth: 1 <= number <= 6
  children: [PhrasingContent]
}
```

**Heading** ([**Parent**][dfn-parent]) represents a heading of a section.

**Heading** can be used where [**flow**][dfn-flow-content] content is expected.
Its content model is [**phrasing**][dfn-phrasing-content] content.

A `depth` field must be present.
A value of `1` is said to be the highest rank and `6` the lowest.

For example, the following markdown:

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
  type: 'thematicBreak'
}
```

**ThematicBreak** ([**Node**][dfn-node]) represents a thematic break, such as a
scene change in a story, a transition to another topic, or a new document.

**ThematicBreak** can be used where [**flow**][dfn-flow-content] content is
expected.
It has no content model.

For example, the following markdown:

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
  type: 'blockquote'
  children: [FlowContent]
}
```

**Blockquote** ([**Parent**][dfn-parent]) represents a section quoted from
somewhere else.

**Blockquote** can be used where [**flow**][dfn-flow-content] content is
expected.
Its content model is also [**flow**][dfn-flow-content] content.

For example, the following markdown:

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
  type: 'list'
  ordered: boolean?
  start: number?
  spread: boolean?
  children: [ListContent]
}
```

**List** ([**Parent**][dfn-parent]) represents a list of items.

**List** can be used where [**flow**][dfn-flow-content] content is expected.
Its content model is [**list**][dfn-list-content] content.

An `ordered` field can be present.
It represents that the items have been intentionally ordered (when `true`), or
that the order of items is not important (when `false` or not present).

A `start` field can be present.
It represents, when the `ordered` field is `true`, the starting number of the
list.

A `spread` field can be present.
It represents that one or more of its children are separated with a blank line
from its [siblings][term-sibling] (when `true`), or not (when `false` or not
present).

For example, the following markdown:

```markdown
1. foo
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
  type: 'listItem'
  spread: boolean?
  children: [FlowContent]
}
```

**ListItem** ([**Parent**][dfn-parent]) represents an item in a
[**List**][dfn-list].

**ListItem** can be used where [**list**][dfn-list-content] content is expected.
Its content model is [**flow**][dfn-flow-content] content.

A `spread` field can be present.
It represents that the item contains two or more [*children*][term-child]
separated by a blank line (when `true`), or not (when `false` or not present).

For example, the following markdown:

```markdown
* bar
```

Yields:

```js
{
  type: 'listItem',
  spread: false,
  children: [{
    type: 'paragraph',
    children: [{type: 'text', value: 'bar'}]
  }]
}
```

### `HTML`

```idl
interface HTML <: Literal {
  type: 'html'
}
```

**HTML** ([**Literal**][dfn-literal]) represents a fragment of raw [HTML][].

**HTML** can be used where [**flow**][dfn-flow-content] or
[**phrasing**][dfn-phrasing-content] content is expected.
Its content is represented by its `value` field.

HTML nodes do not have the restriction of being valid or complete HTML
([\[HTML\]][html]) constructs.

For example, the following markdown:

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
  type: 'code'
  lang: string?
  meta: string?
}
```

**Code** ([**Literal**][dfn-literal]) represents a block of preformatted text,
such as ASCII art or computer code.

**Code** can be used where [**flow**][dfn-flow-content] content is expected.
Its content is represented by its `value` field.

This node relates to the [**phrasing**][dfn-phrasing-content] content concept
[**InlineCode**][dfn-inline-code].

A `lang` field can be present.
It represents the language of computer code being marked up.

If the `lang` field is present, a `meta` field can be present.
It represents custom information relating to the node.

For example, the following markdown:

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

And the following markdown:

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

### `Definition`

```idl
interface Definition <: Node {
  type: 'definition'
}

Definition includes Association
Definition includes Resource
```

**Definition** ([**Node**][dfn-node]) represents a resource.

**Definition** can be used where [**content**][dfn-content] is expected.
It has no content model.

**Definition** includes the mixins [**Association**][dfn-mxn-association] and
[**Resource**][dfn-mxn-resource].

**Definition** should be associated with
[**LinkReferences**][dfn-link-reference] and
[**ImageReferences**][dfn-image-reference].

For example, the following markdown:

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

### `Text`

```idl
interface Text <: Literal {
  type: 'text'
}
```

**Text** ([**Literal**][dfn-literal]) represents everything that is just text.

**Text** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content is represented by its `value` field.

For example, the following markdown:

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
  type: 'emphasis'
  children: [TransparentContent]
}
```

**Emphasis** ([**Parent**][dfn-parent]) represents stress emphasis of its
contents.

**Emphasis** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is [**transparent**][dfn-transparent-content] content.

For example, the following markdown:

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
  type: 'strong'
  children: [TransparentContent]
}
```

**Strong** ([**Parent**][dfn-parent]) represents strong importance, seriousness,
or urgency for its contents.

**Strong** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is [**transparent**][dfn-transparent-content] content.

For example, the following markdown:

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

### `InlineCode`

```idl
interface InlineCode <: Literal {
  type: 'inlineCode'
}
```

**InlineCode** ([**Literal**][dfn-literal]) represents a fragment of computer
code, such as a file name, computer program, or anything a computer could parse.

**InlineCode** can be used where [**phrasing**][dfn-phrasing-content] content
is expected.
Its content is represented by its `value` field.

This node relates to the [**flow**][dfn-flow-content] content concept
[**Code**][dfn-code].

For example, the following markdown:

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
  type: 'break'
}
```

**Break** ([**Node**][dfn-node]) represents a line break, such as in poems or
addresses.

**Break** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
It has no content model.

For example, the following markdown:

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
  type: 'link'
  children: [StaticPhrasingContent]
}

Link includes Resource
```

**Link** ([**Parent**][dfn-parent]) represents a hyperlink.

**Link** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is [**static phrasing**][dfn-static-phrasing-content] content.

**Link** includes the mixin [**Resource**][dfn-mxn-resource].

For example, the following markdown:

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
  type: 'image'
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

For example, the following markdown:

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
  type: 'linkReference'
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

For example, the following markdown:

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
  type: 'imageReference'
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

For example, the following markdown:

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
It can match another node.
`identifier` is a source value: character escapes and character references are
*not* parsed.
Its value must be normalized.

A `label` field can be present.
`label` is a string value: it works just like `title` on a link or a `lang` on
code: character escapes and character references are parsed.

To normalize a value, collapse markdown whitespace (`[\t\n\r ]+`) to a space,
trim the optional initial and/or final space, and perform case-folding.

Whether the value of `identifier` (or normalized `label` if there is no
`identifier`) is expected to be a unique identifier or not depends on the type
of node including the **Association**.
An example of this is that they should be unique on
[**Definition**][dfn-definition], whereas multiple
[**LinkReference**][dfn-link-reference]s can be non-unique to be associated with
one definition.

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

### `referenceType`

```idl
enum referenceType {
  'shortcut' | 'collapsed' | 'full'
}
```

**referenceType** represents the explicitness of a reference.

*   **shortcut**: the reference is implicit, its identifier inferred from its
    content
*   **collapsed**: the reference is explicit, its identifier inferred from its
    content
*   **full**: the reference is explicit, its identifier explicitly set

## Content model

```idl
type MdastContent = FlowContent | ListContent | PhrasingContent
```

Each node in mdast falls into one or more categories of **Content** that group
nodes with similar characteristics together.

### `FlowContent`

```idl
type FlowContent =
  Blockquote | Code | Heading | HTML | List | ThematicBreak | Content
```

**Flow** content represent the sections of document.

### `Content`

```idl
type Content = Definition | Paragraph
```

**Content** represents runs of text that form definitions and paragraphs.

### `ListContent`

```idl
type ListContent = ListItem
```

**List** content represent the items in a list.

### `PhrasingContent`

```idl
type PhrasingContent = Link | LinkReference | StaticPhrasingContent
```

**Phrasing** content represent the text in a document, and its markup.

### `StaticPhrasingContent`

```idl
type StaticPhrasingContent =
  Break | Emphasis | HTML | Image | ImageReference | InlineCode | Strong | Text
```

**StaticPhrasing** content represent the text in a document, and its
markup, that is not intended for user interaction.

### `TransparentContent`

The **transparent** content model is derived from the content model of its
[parent][dfn-parent].
Effectively, this is used to prohibit nested links (and link references).

## Extensions

Markdown syntax is often extended.
It is not a goal of this specification to list all possible extensions.
However, a short list of frequently used extensions are shown below.

### GFM

The following interfaces are found in [GitHub Flavored Markdown][gfm].

#### `FootnoteDefinition`

```idl
interface FootnoteDefinition <: Parent {
  type: 'footnoteDefinition'
  children: [FlowContent]
}

FootnoteDefinition includes Association
```

**FootnoteDefinition** ([**Parent**][dfn-parent]) represents content relating
to the document that is outside its flow.

**FootnoteDefinition** can be used where [**flow**][dfn-flow-content] content is
expected.
Its content model is also [**flow**][dfn-flow-content] content.

**FootnoteDefinition** includes the mixin
[**Association**][dfn-mxn-association].

**FootnoteDefinition** should be associated with
[**FootnoteReferences**][dfn-footnote-reference].

For example, the following markdown:

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

#### `FootnoteReference`

```idl
interface FootnoteReference <: Node {
  type: 'footnoteReference'
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

For example, the following markdown:

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

#### `Table`

```idl
interface Table <: Parent {
  type: 'table'
  align: [alignType]?
  children: [TableContent]
}
```

**Table** ([**Parent**][dfn-parent]) represents two-dimensional data.

**Table** can be used where [**flow**][dfn-flow-content] content is expected.
Its content model is [**table**][dfn-table-content] content.

The [*head*][term-head] of the node represents the labels of the columns.

An `align` field can be present.
If present, it must be a list of [**alignType**s][dfn-enum-align-type].
It represents how cells in columns are aligned.

For example, the following markdown:

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

#### `TableRow`

```idl
interface TableRow <: Parent {
  type: 'tableRow'
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

#### `TableCell`

```idl
interface TableCell <: Parent {
  type: 'tableCell'
  children: [PhrasingContent]
}
```

**TableCell** ([**Parent**][dfn-parent]) represents a header cell in a
[**Table**][dfn-table], if its parent is a [*head*][term-head], or a data
cell otherwise.

**TableCell** can be used where [**row**][dfn-row-content] content is expected.
Its content model is [**phrasing**][dfn-phrasing-content] content excluding
[**Break**][dfn-break] nodes.

For an example, see [**Table**][dfn-table].

#### `ListItem` (GFM)

```idl
interface ListItemGfm <: ListItem {
  checked: boolean?
}
```

In GFM, a `checked` field can be present.
It represents whether the item is done (when `true`), not done (when `false`),
or indeterminate or not applicable (when `null` or not present).

#### `Delete`

```idl
interface Delete <: Parent {
  type: 'delete'
  children: [TransparentContent]
}
```

**Delete** ([**Parent**][dfn-parent]) represents contents that are no longer
accurate or no longer relevant.

**Delete** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is [**transparent**][dfn-transparent-content] content.

For example, the following markdown:

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

#### `alignType`

```idl
enum alignType {
  'left' | 'right' | 'center' | null
}
```

**alignType** represents how phrasing content is aligned
([\[CSSTEXT\]][css-text]).

*   **`'left'`**: See the [`left`][css-left] value of the `text-align` CSS
    property
*   **`'right'`**: See the [`right`][css-right] value of the `text-align`
    CSS property
*   **`'center'`**: See the [`center`][css-center] value of the `text-align`
    CSS property
*   **`null`**: phrasing content is aligned as defined by the host environment

#### `FlowContent` (GFM)

```idl
type FlowContentGfm = FootnoteDefinition | Table | FlowContent
```

#### `TableContent`

```idl
type TableContent = TableRow
```

**Table** content represent the rows in a table.

#### `RowContent`

```idl
type RowContent = TableCell
```

**Row** content represent the cells in a row.

#### `ListContent` (GFM)

```idl
type ListContentGfm = ListItemGfm
```

#### `StaticPhrasingContent` (GFM)

```idl
type StaticPhrasingContentGfm =
  FootnoteReference | Delete | StaticPhrasingContent
```

### Frontmatter

The following interfaces are found with YAML.

#### `YAML`

```idl
interface YAML <: Literal {
  type: 'yaml'
}
```

**YAML** ([**Literal**][dfn-literal]) represents a collection of metadata for
the document in the YAML ([\[YAML\]][yaml]) data serialisation language.

**YAML** can be used where [**frontmatter**][dfn-frontmatter-content] content is
expected.
Its content is represented by its `value` field.

For example, the following markdown:

```markdown
---
foo: bar
---
```

Yields:

```js
{type: 'yaml', value: 'foo: bar'}
```

#### `FrontmatterContent`

```idl
type FrontmatterContent = YAML
```

**Frontmatter** content represent out-of-band information about the document.

If frontmatter is present, it must be limited to one node in the
[*tree*][term-tree], and can only exist as a [*head*][term-head].

#### `FlowContent` (frontmatter)

```idl
type FlowContentFrontmatter = FrontmatterContent | FlowContent
```

### Footnotes

The following interfaces are found with footnotes (pandoc).
Note that pandoc also uses [**FootnoteReference**][dfn-footnote-reference]
and [**FootnoteDefinition**][dfn-footnote-definition], but since
[GFM now supports footnotes][gfm-footnote], their definitions were moved to the
[GFM][gfm-section] section

#### `Footnote`

```idl
interface Footnote <: Parent {
  type: 'footnote'
  children: [PhrasingContent]
}
```

**Footnote** ([**Parent**][dfn-parent]) represents content relating to the
document that is outside its flow.

**Footnote** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is also [**phrasing**][dfn-phrasing-content] content.

For example, the following markdown:

```markdown
^[alpha bravo]
```

Yields:

```js
{
  type: 'footnote',
  children: [{type: 'text', value: 'alpha bravo'}]
}
```

#### `StaticPhrasingContent` (footnotes)

```idl
type StaticPhrasingContentFootnotes = Footnote | StaticPhrasingContent
```

### MDX

See [`remark-mdx`](https://mdxjs.com/packages/remark-mdx/#syntax-tree).

## Glossary

See the [unist glossary][glossary].

## List of utilities

See the [unist list of utilities][utilities] for more utilities.

<!--lint disable list-item-spacing-->

*   [`mdast-add-list-metadata`](https://gitlab.com/staltz/mdast-add-list-metadata)
    — enhance the metadata of `list` and `listItem` nodes
*   [`mdast-util-assert`](https://github.com/syntax-tree/mdast-util-assert)
    — assert nodes
*   [`mdast-builder`](https://github.com/mike-north/mdast-builder)
    — build mdast structures with composable functions
*   [`mdast-comment-marker`](https://github.com/syntax-tree/mdast-comment-marker)
    — parse a comment marker
*   [`mdast-util-compact`](https://github.com/syntax-tree/mdast-util-compact)
    — make a tree compact
*   [`mdast-util-definitions`](https://github.com/syntax-tree/mdast-util-definitions)
    — find definition nodes
*   [`mdast-util-directive`](https://github.com/syntax-tree/mdast-util-directive)
    — parse and serialize directives
*   [`mdast-util-find-and-replace`](https://github.com/syntax-tree/mdast-util-find-and-replace)
    — find and replace text
*   [`mdast-flatten-image-paragraphs`](https://gitlab.com/staltz/mdast-flatten-image-paragraphs)
    — flatten `paragraph` and `image` into one `image` node
*   [`mdast-flatten-listitem-paragraphs`](https://gitlab.com/staltz/mdast-flatten-listitem-paragraphs)
    — flatten `listItem` and (nested) paragraph into one listItem node
*   [`mdast-flatten-nested-lists`](https://gitlab.com/staltz/mdast-flatten-nested-lists)
    — transform a tree to avoid lists in lists
*   [`mdast-util-from-adf`](https://github.com/bitcrowd/mdast-util-from-adf)
    — build mdast syntax tree from Atlassian Document Format (ADF)
*   [`mdast-util-from-markdown`](https://github.com/syntax-tree/mdast-util-from-markdown)
    — parse markdown
*   [`mdast-util-frontmatter`](https://github.com/syntax-tree/mdast-util-frontmatter)
    — parse and serialize frontmatter
*   [`mdast-util-gfm`](https://github.com/syntax-tree/mdast-util-gfm)
    — parse and serialize GFM
*   [`mdast-util-gfm-autolink-literal`](https://github.com/syntax-tree/mdast-util-gfm-autolink-literal)
    — parse and serialize GFM autolink literals
*   [`mdast-util-gfm-footnote`](https://github.com/syntax-tree/mdast-util-gfm-footnote)
    — parse and serialize GFM footnotes
*   [`mdast-util-gfm-strikethrough`](https://github.com/syntax-tree/mdast-util-gfm-strikethrough)
    — parse and serialize GFM strikethrough
*   [`mdast-util-gfm-table`](https://github.com/syntax-tree/mdast-util-gfm-table)
    — parse and serialize GFM tables
*   [`mdast-util-gfm-task-list-item`](https://github.com/syntax-tree/mdast-util-gfm-task-list-item)
    — parse and serialize GFM task list items
*   [`mdast-util-gridtables`](https://github.com/syntax-tree/mdast-util-gridtables)
    — parse and serialize gridtables
*   [`mdast-util-heading-range`](https://github.com/syntax-tree/mdast-util-heading-range)
    — markdown heading as ranges
*   [`mdast-util-heading-style`](https://github.com/syntax-tree/mdast-util-heading-style)
    — get the style of a heading node
*   [`mdast-util-hidden`](https://github.com/Xunnamius/unified-utils/tree/main/packages/mdast-util-hidden)
    — prevent nodes from being seen by transformers.
*   [`mdast-util-math`](https://github.com/syntax-tree/mdast-util-math)
    — parse and serialize math
*   [`mdast-util-mdx`](https://github.com/syntax-tree/mdast-util-mdx)
    — parse and serialize MDX
*   [`mdast-util-mdx-expression`](https://github.com/syntax-tree/mdast-util-mdx-expression)
    — parse and serialize MDX expressions
*   [`mdast-util-mdx-jsx`](https://github.com/syntax-tree/mdast-util-mdx-jsx)
    — parse and serialize MDX JSX
*   [`mdast-util-mdxjs-esm`](https://github.com/syntax-tree/mdast-util-mdxjs-esm)
    — parse and serialize MDX ESM
*   [`mdast-move-images-to-root`](https://gitlab.com/staltz/mdast-move-images-to-root)
    — move image nodes up the tree until they are direct children of the root
*   [`mdast-normalize-headings`](https://github.com/syntax-tree/mdast-normalize-headings)
    — ensure at most one top-level heading is in the document
*   [`mdast-util-phrasing`](https://github.com/syntax-tree/mdast-util-phrasing)
    — check if a node is phrasing content
*   [`mdast-squeeze-paragraphs`](https://github.com/syntax-tree/mdast-squeeze-paragraphs)
    — remove empty paragraphs
*   [`mdast-util-toc`](https://github.com/syntax-tree/mdast-util-toc)
    — generate a table of contents from a tree
*   [`mdast-util-to-hast`](https://github.com/syntax-tree/mdast-util-to-hast)
    — transform to hast
*   [`mdast-util-to-markdown`](https://github.com/syntax-tree/mdast-util-to-markdown)
    — serialize markdown
*   [`mdast-util-to-nlcst`](https://github.com/syntax-tree/mdast-util-to-nlcst)
    — transform to nlcst
*   [`mdast-util-to-string`](https://github.com/syntax-tree/mdast-util-to-string)
    — get the plain text content of a node
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

## Related

*   [hast](https://github.com/syntax-tree/hast)
    — Hypertext Abstract Syntax Tree format
*   [nlcst](https://github.com/syntax-tree/nlcst)
    — Natural Language Concrete Syntax Tree format
*   [xast](https://github.com/syntax-tree/xast)
    — Extensible Abstract Syntax Tree

## Contribute

See [`contributing.md`][contributing] in [`syntax-tree/.github`][health] for
ways to get started.
See [`support.md`][support] for ways to get help.
Ideas for new utilities and tools can be posted in [`syntax-tree/ideas`][ideas].

A curated list of awesome syntax-tree, unist, mdast, hast, xast, and nlcst
resources can be found in [awesome syntax-tree][awesome].

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to
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

[contributing]: https://github.com/syntax-tree/.github/blob/HEAD/contributing.md

[support]: https://github.com/syntax-tree/.github/blob/HEAD/support.md

[coc]: https://github.com/syntax-tree/.github/blob/HEAD/code-of-conduct.md

[awesome]: https://github.com/syntax-tree/awesome-syntax-tree

[ideas]: https://github.com/syntax-tree/ideas

[license]: https://creativecommons.org/licenses/by/4.0/

[author]: https://wooorm.com

[logo]: https://raw.githubusercontent.com/syntax-tree/mdast/e6b43aa/logo.svg?sanitize=true

[releases]: https://github.com/syntax-tree/mdast/releases

[latest]: https://github.com/syntax-tree/mdast/releases/tag/4.0.0

[dfn-node]: https://github.com/syntax-tree/unist#node

[dfn-unist-parent]: https://github.com/syntax-tree/unist#parent

[dfn-unist-literal]: https://github.com/syntax-tree/unist#literal

[dfn-parent]: #parent

[dfn-literal]: #literal

[dfn-code]: #code

[dfn-inline-code]: #inlinecode

[dfn-list]: #list

[dfn-table]: #table

[dfn-break]: #break

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

[dfn-mdast-content]: #content-model

[dfn-flow-content]: #flowcontent

[dfn-frontmatter-content]: #frontmattercontent

[dfn-content]: #content

[dfn-list-content]: #listcontent

[dfn-table-content]: #tablecontent

[dfn-row-content]: #rowcontent

[dfn-phrasing-content]: #phrasingcontent

[dfn-static-phrasing-content]: #staticphrasingcontent

[dfn-transparent-content]: #transparentcontent

[gfm-section]: #gfm

[gfm-footnote]: https://github.blog/changelog/2021-09-30-footnotes-now-supported-in-markdown-fields/

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
