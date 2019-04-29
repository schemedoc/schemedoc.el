# Overview

This document contains a summary of a few related threads from
the [srfi-discuss](https://srfi-email.schemers.org/srfi-discuss/) mailing list around the idea of adding structural
information to existing and new [Scheme SRFIs](https://srfi.schemers.org/) in order to potentially support various
mostly documentation-related tools. Namely:

- Summary of the thread "Proposal to add HTML class attributes to SRFIs to aid machine-parsing",
  see [here](https://srfi-email.schemers.org/srfi-discuss/msg/10613509). The initial post from Lassi Kortela has already
  a very detailed summary of the status until that thread started.

- There is also the related thread "Format of S-expression metadata for SRFI documents", with the first
  post [here](https://srfi-email.schemers.org/srfi-discuss/msg/10667443). Again this initial post from Lassi Kortela
  provides a summary of the discussion status by then. I also merged the relevant information into this assembly.

- Eventually the thread "Format of S-expression metadata for SRFI documents" became "SRFI server infrastructure"
  with [this post](https://srfi-email.schemers.org/srfi-discuss/msg/10785159). As for a start a very simple
  infrastructure has been chosen, not much of the info from that track is listed here.

This document does not contain the pros and cons for (and the back and forth around) many discussed options, the main
thread of the discussion eventually did lead to the creation a Github repository, where the editors maintain the tools and
content as agreed upon by iterative refinement. Approach the mailing case if you're interested in particpating.

## Interesting links

- [Existing template](https://srfi.schemers.org/srfi-template.html)

- Earlier threads on similar topics are
  [Is there an index of symbols defined by the various SRFI's?](https://srfi-email.schemers.org/srfi-discuss/msg/8163553)
  and [Describing Scheme libraries (and thus SRFI's and R7RS) in a "machine readable" format ...](https://srfi-email.schemers.org/srfi-discuss/msg/8932119)

- [Lassi Kortela's SRFI symbol index](https://schemedoc.herokuapp.com/)

- [Ciprian Craciun's SRFI symbol index](https://github.com/cipriancraciun/scheme-srfi-index)

- [Sample for generated output](https://vonuvoli.volution.ro/documentation/libraries-html/_libraries.html)

# Summary

## Task description

Gradually add a standardized set of HTML class attributes to the SRFI source documents.

- Encode metadata that can be used to index all symbols defined in SRFIs.

- General information about the SRFI (date, author, abstract, status, license, etc.).

These additional meta data can then be used in order to

- generate indexes of all the names defined by each SRFI, also allowing back-referencing to the indexed elements

- generate reference documentation incl. all procedures and syntax along with their arguments, argument types, return
  types, and descriptions; that documentation should also be workable in documentation browsers (such
  as [devdocs.io](http://devdocs.io/))

- to model type signature information detailed enough to permit static analysis of the code (see
  Erlang's [Dialyzer](https://learnyousomeerlang.com/dialyzer)

- support language servers

The relevant meta data categories listed for that purpose would be (as mainly defined
by [this post](https://srfi-email.schemers.org/srfi-discuss/msg/10628469)):

- "global" SRFI related meta-data like author, title, identifier and other "bibliographic" items

- "indexing" items, like `<span class="def-prog">make-array</span>`

- textual "descriptive" elements related to various definitions like for example:
  - textual description of a procedure / macro
  - example code snippets
  - test snippets
  - id attribute for links from external documents into the SRFI

- procedure or syntax definition in a way that is machine readable and from which "signatures" could be extracted

## Requirements and boundary conditions

- The overall HTML structure of the existing SRFI documents is somewhat variable. HTML tags are used differently in
  different SRFIs.

- There exist 166 SRFI documents (and counting), from which 128 are in status "final".

- There are 58 authors in total. About 5 different generator programs have been used by these authors. Authors are
  required to provide HTML directly. The HTML document sources are not always included with the SRFI.

- Some authors write their documents in another markup language (e.g. TeX), then convert them to HTML. The sources in
  those other markup language might no longer be available (e.g. SRFI 67).

- It is been considered mandatory to keep the effort required by an author to comply to the infrastructure discussed
  here minimal, and to keep the "friction" low. The proposed workflow to acchieve that would be to split the
  SRFI-process:

  Let the authors work on the description and add sub-sequent step where maintainers add the markup required for the
  documentation purpose.

  And as long as the text of the SRFIs doesn't change, the APIs won't change, so the changes can be done without
  bothering the authors.

## Options discussed

### Source formats

- Having an extra file per SRFI holding S-expressions for meta data
  (like [here](https://github.com/cipriancraciun/scheme-srfi-index/blob/master/syntax-specification.txt))

- Using HTML class attribute (like [here](https://github.com/lassik/srfi-markup-proposal) or as already applied to
  SRFI-1 [here](https://srfi-email.schemers.org/srfi-discuss/msg/10633861) and then some more
  SRFIs [here](https://srfi-email.schemers.org/srfi-discuss/msg/10635289))

  For the HTML class system both the use of multiple classes (`def proc`, `def var`) as well as the use of mutually
  exclusive classes (`def-proc`, `def-var`) have been discussed.

- Starting with Markdown-based documentation, then converting to HTML or XHTML
  (see [here](https://srfi-email.schemers.org/srfi-discuss/msg/10614555)); alternatively starting with
  reStructuredText-based documentation

- Embed HTML comments in the SRFIs and parse them

- Custom XML (e.g. [IETF](https://xml2rfc.tools.ietf.org/))

- HTML micro-format
  (e.g. [microformats2](http://microformats.org/wiki/microformats2), [h-card](http://microformats.org/wiki/h-card))

- Adding the additional meta data as visible or invisible elements to the HTML

- Skribe or Racket's Scribble; Latex or TeX

- Various options have been discussed on where to define the additional meta data:

  - Agree on one "external" index format and have more than one tool to make use of that data or have one SRFI source
    document format (without "external" index) and one API server based on that (with the optional step to auto-generate
    indexes from the source documents; API samples are
    given [here](https://srfi-email.schemers.org/srfi-discuss/msg/10629035).

  - Or have a mixed approach where as much as possible is been read from the (X)HTML source and potentially "cached" in
    an external file, if such an external meta data is ot yet available, otherwise the source extract is cross-checked
    against the external file (detailed out a bit more [here](https://srfi-email.schemers.org/srfi-discuss/msg/10641251)
    and [here](https://srfi-email.schemers.org/srfi-discuss/msg/10649211).

  - Also this step can be supported by using a merge of the introspection facilities of some Scheme implementations,
    which provide means to describe bound identifiers.

  - For both approaches agreement seems to be that the interesting parts to potentially move out of the markup-document
    and into an extra file with extra syntax (e.g. S-expressions) would be detailed type signatures e.g. for for static
    type inference. Still there also seems to be agreement that additional meta data could also reasonably be added to
    that external file. The meta data would then fall more into the category of data extracted from the source documents
    where the type signatures would probably be information exclusively defined in an extra file.

  - Some more details on naming conventions, file content and workflow are given
    in [this](https://srfi-email.schemers.org/srfi-discuss/msg/10675263)
    and [this](https://srfi-email.schemers.org/srfi-discuss/msg/10686585) post; then later detailed out
    nicely [here](https://srfi-email.schemers.org/srfi-discuss/msg/10780913).

### Target document formats

- HTML4, HTML5, XHTML or "polyglot markup" (both valid HTML and XML).

- Switch to XHTML and use separate XML namespace for metadata

  A detailed example, where a structured XHTML is manually built from an existing HTML and then again transformed to
  HTML - but with the desired meta data - is given [here](https://srfi-email.schemers.org/srfi-discuss/msg/10662889).

### Technical options

- Tool options: pup ([HTML command line parser](https://github.com/ericchiang/pup), jq
  ([JSON command line processor](https://stedolan.github.io/jq/), mf2py
  ([Microformats2 parser in Python](https://github.com/microformats/mf2py)),
  [check for unbalanced or missing tags](https://www.jwz.org/hacks/validate.pl)

- Programming language: should this be done in Scheme (which one) or another language - or is it just fine to use
  whatever fits best for a given sub-task? That question is shortly discussed
  in [this post](https://srfi-email.schemers.org/srfi-discuss/msg/10686531).

## Proposals for next steps

- The overall task [has to be split](https://srfi-email.schemers.org/srfi-discuss/msg/10617365) into:

  - "new" SRFI documents that have to be "structured", "annotated" and "indexed"

  - "old" SRFI documents that can be just "back-referenced" starting from an existing "index" (the one "crawled" by
    Ciprian)

  - "transforming" the old SRFI documents so that they are in-line with the newly proposed format

- When working on "new" SRFI documents, the proposed workflow would be
  ([1](https://srfi-email.schemers.org/srfi-discuss/msg/10617365),
  [2](https://srfi-email.schemers.org/srfi-discuss/msg/10617511),
  [3](https://srfi-email.schemers.org/srfi-discuss/msg/10687827),
  [4](https://srfi-email.schemers.org/srfi-discuss/msg/10693839))

  - the editors come-up with some "document format"

  - the author writes the SRFI as he wants (and is accepted by the editor)

  - for those authors that prefer HTML / LaTeX or something else, the editors just take the final version of their SRFI,
    and manually convert it (meaning: more "actively" formats it)

  - the editor together with the author finalize the document in the "free-form" HTML format;

  - from here on the "volunteers" (or if time permits the editor and/or author) would take the HTML, reformat it and
    augment it as needed, and add the signatures;

  - (nothing prevents the author to start with this last form, but it is not part of the SRFI requirements /
    guidelines;)

That workflow pertains three categories of document changes/augmentation: markup, meta data, type signatures; where at
least the type signatures can be processed in a separate step. Again various options as how to maintain a status
overview are given in [this post](https://srfi-email.schemers.org/srfi-discuss/msg/10693985).)
