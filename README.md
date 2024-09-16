# `Prefer-No-Speculative-Parsing` explainer

> [!IMPORTANT]
> This proposal has been revised into a [configuration point](https://wicg.github.io/document-policy#configuration-point) in [Document Policy](https://wicg.github.io/document-policy/) header named `expect-no-embedded-resources`. More information can be found on the [expect-no-linked-resources](https://github.com/explainers-by-googlers/expect-no-linked-resources) repository. This repository has been archived.

The `Prefer-No-Speculative-Parsing` HTTP response header allows a developer to request the user agent to not use the default speculative parsing behavior.

## Participate
- https://github.com/explainers-by-googlers/prefer-no-speculative-parsing/issues

## Table of Contents

<!-- Update this table of contents by running `npx doctoc README.md` -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [`Prefer-No-Speculative-Parsing` explainer](#prefer-no-speculative-parsing-explainer)
  - [Participate](#participate)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Example](#example)
  - [Goals](#goals)
  - [Non-goals](#non-goals)
  - [User research](#user-research)
  - [Specification plan](#specification-plan)
  - [Considered alternatives](#considered-alternatives)
    - [A `<meta>` tag version](#a-meta-tag-version)
    - [Use of `Content-Security-Policy` meta tag as a work-around](#use-of-content-security-policy-meta-tag-as-a-work-around)
  - [References \& acknowledgements](#references--acknowledgements)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Introduction

User Agents have implemented [speculative parsing of HTML](https://html.spec.whatwg.org/multipage/parsing.html#speculative-html-parsing) to speculatively fetch resources that are present in the HTML markup, to speed up page loading. For the vast majority of pages on the Web that have resources declared in the HTML markup, the optimization is beneficial and the cost paid in determining such resources is a sound tradeoff. However, the following scenarios might result in a sub-optimal performance tradeoff vs. the explicit time spent parsing HTML for determining sub resources to fetch:

 * Pages that do not have any resources declared in the HTML.
 * Large HTML pages with minimal or no resource loads that could explicitly control preloading resources via other preload mechanisms available.
 * Pages that have privacy concerns around resource loads during the speculative fetch phase.

This proposal introduces a hint response header `Prefer-No-Speculative-Parsing` to explicitly state to the User Agent that it may choose to optimize out the time spent in such sub resource determination.

## Example

A document can request that the default speculative parsing behavior is not used, by using the `Prefer-No-Speculative-Parsing` HTTP header. This header is a HTTP [structured field](https://httpwg.org/http-extensions/draft-ietf-httpbis-header-structure.html) whose value must be Boolean.


```http
HTTP/1.1 200 OK
Content-Type: text/html
...
Prefer-No-Speculative-Parsing: ?1
...
```

The presence of the header, with the structured headers boolean "true" value `?1`, indicates that resulting document’s [active speculative HTML parser](https://html.spec.whatwg.org/multipage/parsing.html#active-speculative-html-parser) may be explicitly set to `null`. Note that this header indicates only a preference from the Document on how the speculative HTML parser may behave, which the user agent may eventually choose to ignore. In terms of observable effects, this means that any fetches originating from speculative HTML parser may be avoided and any time spent in speculative parsing may be reduced. Resources that were being fetched as part of a speculative fetch, will then be fetched as part of the normal document parsing. Behind the scenes, this preference can allow user agents to skip the time spent in speculative parsing, and deallocate any implementation specific resources corresponding to speculative parsing for additional efficiency.


## Goals

 * Allow well informed web developers to assist User Agents to optimize loading sequence better, resulting in a more performant scenario.
 * Allow opting-in per HTTP response, creating a very targeted signal that reduces misuse.
 * Allow User Agents to determine if they should heed to the hint, preventing any binding contracts that prevent future optimization of the speculating parser and prefetch implementation.

## Non-goals

 * Generalize the header to signal more than the intended hint.
 * Allow explicit control of the speculative parsing or preloading scanning mechanism — it’s better this is designed as a hint that a response is supplied with, on which the UA may choose to vary its usage of speculative parsing on.

## User research

An Origin Trial was conducted in Chrome from 125 to 130 that allowed participants to disable the default speculative prefetch behavior of the user agent. More information about the Origin Trial can be found on the [Chrome Platform Status page for the Skip Preload Scanning Feature](https://chromestatus.com/feature/5190976638550016) and the [explainer document](https://docs.google.com/document/d/1wiaTL5TeONTZamycMVMjo76nMcbhHNYznQy7I_zCVRY/edit).

Additional comments and usecases from the community were received on the [tracker issue](https://issues.chromium.org/issues/330802493) that influenced the design of this header.

Participants observed improved performance on pages that do not have external
resource fetches by reducing the time spent towards speculative parsing.

## Specification plan

The specification for this feature in itself consists of two parts.

One part is header parsing, which relies on the [Structured Headers](https://httpwg.org/http-extensions/draft-ietf-httpbis-header-structure.html) specification to do the work. The other is altering behavior of the [active HTML speculative parser](https://html.spec.whatwg.org/multipage/parsing.html#active-speculative-html-parser), which would be done by modifying the HTML Standard's [Speculative HTML parsing](https://html.spec.whatwg.org/multipage/parsing.html#speculative-html-parsing).

Please see the associated [spec proposal](https://explainers-by-googlers.github.io/prefer-no-speculative-parsing) for more information.

## Considered alternatives

### A `<meta>` tag version

While a `meta` element is in some cases easier for web developers to add to their page than a header, it would make the behavior of such a directive unpredictable. It will also add further complexity in determining how such a directive would work given the placement and timing of itself against resources specified in the HTML. This option has been considered and rejected.

### Use of `Content-Security-Policy` meta tag as a work-around

Some user agents may choose to disable active speculative HTML parser partly or completely on encountering a CSP meta tag. A more direct hint from the page is a more precise signal vs. overloading of the CSP header for implicit disabling of active speculative HTML parser.

## References & acknowledgements

Many thanks for valuable feedback and advice from:

- Domenic Denicola
- Yoav Weiss
