# docs-pdfs

PDFs of the Solo.io open source and enterprise documentation, one file per product
version, for reading offline or printing.

This repository holds no documentation source. It is a distribution point: the
PDFs are attached to [releases](https://github.com/solo-io/docs-pdfs/releases),
and they are rebuilt daily from the documentation sources whenever those
sources change.

> [!NOTE]
> A PDF is a snapshot, not the current documentation. The published site is
> always authoritative. Each PDF prints its build date on the cover page, so you
> can tell how old the copy in your hands is.

## Download a PDF

Every product version has one release, and its asset URL is stable:

```
https://github.com/solo-io/docs-pdfs/releases/download/<product>-<distribution>-<version>/<product>-<distribution>-<version>.pdf
```

`<distribution>` is `enterprise` or `oss`. It is part of the tag because several
products are documented as both: `kgateway`, `agentgateway`, `agentregistry` and
`kagent` each have an enterprise documentation set on docs.solo.io and an open
source one on its own site. Without it the two would share a tag.

A product that splits one version across separate documentation sets adds that
set to the tag as well, between the distribution and the version, such as `agentgateway-enterprise-kubernetes-latest`
and `agentgateway-enterprise-standalone-latest`.

For example:

```sh
curl -LO https://github.com/solo-io/docs-pdfs/releases/download/gloo-mesh-enterprise-enterprise-latest/gloo-mesh-enterprise-enterprise-latest.pdf
```

The URL does not change when the PDF is refreshed, so it is safe to link to or
to script against.

### Available PDFs

| Product | Distribution | Version | Download |
| --- | --- | --- | --- |
| Agentgateway Enterprise (Kubernetes docs) | enterprise | latest (2026.8.2) | [agentgateway-enterprise-kubernetes-latest.pdf](https://github.com/solo-io/docs-pdfs/releases/download/agentgateway-enterprise-kubernetes-latest/agentgateway-enterprise-kubernetes-latest.pdf) |
| Agentgateway Enterprise (Standalone docs) | enterprise | latest (2026.8.2) | [agentgateway-enterprise-standalone-latest.pdf](https://github.com/solo-io/docs-pdfs/releases/download/agentgateway-enterprise-standalone-latest/agentgateway-enterprise-standalone-latest.pdf) |
| Agentregistry Enterprise | enterprise | latest (2026.8.0) | [agentregistry-enterprise-latest.pdf](https://github.com/solo-io/docs-pdfs/releases/download/agentregistry-enterprise-latest/agentregistry-enterprise-latest.pdf) |
| Gloo Gateway | enterprise | 1.22.x | [gateway-enterprise-1.22.x.pdf](https://github.com/solo-io/docs-pdfs/releases/download/gateway-enterprise-1.22.x/gateway-enterprise-1.22.x.pdf) |
| Gloo Mesh Enterprise | enterprise | latest (2.13.x) | [gloo-mesh-enterprise-enterprise-latest.pdf](https://github.com/solo-io/docs-pdfs/releases/download/gloo-mesh-enterprise-enterprise-latest/gloo-mesh-enterprise-enterprise-latest.pdf) |
| Gloo Mesh Gateway | enterprise | latest (2.13.x) | [gloo-mesh-gateway-enterprise-latest.pdf](https://github.com/solo-io/docs-pdfs/releases/download/gloo-mesh-gateway-enterprise-latest/gloo-mesh-gateway-enterprise-latest.pdf) |
| Istio (Solo distribution) | enterprise | 1.30.x | [istio-enterprise-1.30.x.pdf](https://github.com/solo-io/docs-pdfs/releases/download/istio-enterprise-1.30.x/istio-enterprise-1.30.x.pdf) |
| kagent Enterprise | enterprise | latest (0.5.x) | [kagent-enterprise-latest.pdf](https://github.com/solo-io/docs-pdfs/releases/download/kagent-enterprise-latest/kagent-enterprise-latest.pdf) |
| kgateway Enterprise | enterprise | 2.3.x | [kgateway-enterprise-2.3.x.pdf](https://github.com/solo-io/docs-pdfs/releases/download/kgateway-enterprise-2.3.x/kgateway-enterprise-2.3.x.pdf) |

The version in parentheses is what `latest` pointed at when this table was last
updated, on 31 August 2026. The
[releases page](https://github.com/solo-io/docs-pdfs/releases) is always the
complete list, and no open source PDFs are published yet.

> [!TIP]
> A version named `latest` tracks whichever release is currently latest, so its
> contents change over time. If you need a copy of a specific numbered version,
> download it and keep it, or build your own from a tag.


## Lifecycle

| Event | What happens here |
| --- | --- |
| A version starts publishing a PDF | Its release is created on the next run |
| The documentation changes | The release keeps its URL; only the attached file is replaced |
| The documentation does not change | Nothing runs, and the existing PDF stays |
| A version is retired | Its release is frozen and stops being updated |
| A retired version is no longer wanted | Its release is deleted by hand |

Which versions publish a PDF is decided by the documentation build that produces
them. If a version you want is not listed, open an issue here and ask for it. To
get one right now instead, build it yourself with the steps below.

### Why releases instead of committed files

A PDF is an already-compressed stream, so Git cannot store an updated one as a
small delta. Committing a nightly rebuild of every product would add its full
size to the repository history every night, permanently, for files nobody will
ever want an old revision of. Release assets live outside the Git history, so
replacing one leaves nothing behind and deleting one actually frees the space.

This is also why there is no archive of past builds. If you need a specific
historical state of the documentation, build it yourself from the source at that
point in time.

## Build a PDF yourself

The open source documentation sites are public, so you can build their PDFs
yourself. That is useful for a version that is not published here, for an older
release you want to keep, or for checking a change before it ships.

| Site | Repository |
| --- | --- |
| kgateway.dev | [kgateway-dev/kgateway.dev](https://github.com/kgateway-dev/kgateway.dev) |
| agentgateway.dev | [agentgateway/website](https://github.com/agentgateway/website) |
| agentregistry | [agentregistry-dev/website](https://github.com/agentregistry-dev/website) |
| kagent | [kagent-dev/website](https://github.com/kagent-dev/website) |

They all use the same Hugo theme, so the steps below are the same in each one;
only the paths differ. The examples use kgateway.dev.

You need Hugo extended, Python 3, and WeasyPrint's system libraries. On Debian
or Ubuntu:

```sh
sudo apt-get install -y libpango-1.0-0 libpangoft2-1.0-0 libharfbuzz0b \
  fonts-dejavu-core fonts-liberation
pip install weasyprint lxml cssselect pypdf
```

On macOS, `brew install pango` provides the same libraries.

Then, from a checkout of the site:

```sh
git clone https://github.com/kgateway-dev/kgateway.dev
cd kgateway.dev
```

1. **Opt the version you want into the `book` output format.** Add this to that
   version's `_index.md`, for example `content/docs/envoy/latest/_index.md`:

   ```yaml
   outputs: ["html", "book"]
   ```

   Put it on the **version root**, not on the sections beneath it. A page that
   opts in stitches itself plus everything below it, so the version root gives
   you the whole version in one document. kgateway.dev already opts its
   individual sections in, for an older per-section PDF, and leaving that alone
   is fine — it just produces extra `book.html` files you can ignore.

   The site config has to define the format too. kgateway.dev already does, in
   `hugo.yaml`:

   ```yaml
   outputFormats:
     book:
       mediaType: "text/html"
       baseName: "book"
       isHTML: true
   ```

2. **Build the site**, which produces `book.html` alongside that version's
   `index.html`:

   ```sh
   hugo
   ```

   The book lands next to the page that opted in, so for the example above it is
   `public/docs/envoy/latest/book.html`.

3. **Prepare the stitched document.** Heading IDs are only unique per source
   page, so a stitched book repeats them; this step makes them unique and
   rewrites the links so cross-references jump inside the PDF instead of going
   back to the website. `--strict` fails if anything is left unresolved.

   It also splits the book into parts of about 2 MB each, listed in
   `book.parts.txt`. Renderer memory grows with the number of pages produced,
   roughly 1.6 MB per page, so a large book rendered in one pass can need more
   than 10 GB. Rendering a part at a time keeps that bounded.

   The last argument is the site's own origin, used to resolve links that point
   outside the book:

   ```sh
   EXTRAS=$(awk '/solo-io\/docs-theme-extras/ {print $2}' go.mod)
   for f in prepare_book.py merge_book.py; do
     curl -fsSL "https://raw.githubusercontent.com/solo-io/docs-theme-extras/$EXTRAS/scripts/$f" -o "$f"
   done

   BOOK=public/docs/envoy/latest/book.html
   python3 prepare_book.py "$BOOK" "$BOOK" https://kgateway.dev --strict
   ```

   Fetching the scripts at the `docs-theme-extras` version your `go.mod` already
   requires, rather than at `main`, keeps them matched to the book layout that
   produced your HTML.

4. **Render each part, then merge.** Serve the built site first, so that
   root-relative image and stylesheet paths resolve the way they do in
   production. Parts must be rendered in order, because each one is told which
   page number to start at:

   ```sh
   python3 -m http.server 8000 --directory public &

   DIR=public/docs/envoy/latest
   URL=http://127.0.0.1:8000/docs/envoy/latest
   NEXT=1
   for PART in $(cat "$DIR/book.parts.txt"); do
     NAME=$(basename "$PART" .html)
     echo "@page :first { counter-reset: page $NEXT; }" > offset.css
     weasyprint -s offset.css "$URL/$NAME.html" "$NAME.pdf"
     NEXT=$((NEXT + $(python3 -c "import sys;from pypdf import PdfReader;print(len(PdfReader(sys.argv[1]).pages))" "$NAME.pdf")))
   done

   python3 merge_book.py kgateway-docs.pdf book.part*.pdf
   ```

   `merge_book.py` restores the links that point from one part into another, and
   fails if any of them cannot be resolved.

## Reporting a problem

Open an issue on this repository for anything about a PDF itself, such as a
broken link inside it, a diagram that did not render, or a missing version. For
a mistake in the documentation content, open the issue against the product's own
documentation repository instead, since fixing it there is what regenerates the
PDF.
