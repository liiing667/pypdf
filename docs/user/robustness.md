# Robustness and strict=False

PDF is [specified in various versions](https://pdfa.org/resource/pdf-specification-archive/).
The specification of PDF 2.0 has 1003 pages. This length makes it hard to get
everything right. As a consequence, a lot of PDF files are not strictly following the
specification.

If a PDF file does not follow the specification, it is not always possible to
be certain what the intended effect would be. Think of the following broken
Python code as an example:

```{testcode}
# Broken
function (foo, bar):

# Potentially intended:
def function(foo, bar):
    ...

# Also possible:
function = (foo, bar)
```

```{testoutput}
:hide:

Traceback (most recent call last):
    ...
SyntaxError: invalid syntax
```

Writing a parser, you can go two paths: Either you try to be forgiving and try
to figure out what the user intended, or you are strict and just tell the user
that they should fix their stuff.

pypdf gives you the option to be strict or not.

pypdf has two core objects:

* {class}`~pypdf.PdfReader`
* {class}`~pypdf.PdfWriter`

PdfReader and PdfWriter both have a `strict` parameter.

Choosing `strict=True` means that pypdf will raise an exception if a PDF does
not follow the specification.

Choosing `strict=False` means that pypdf will try to be forgiving and do
something reasonable, but it will log a warning message. It is a best-effort
approach.

## Error recovery in PdfReader

With `strict=False` (the default), `PdfReader` attempts to recover from the
following kinds of damage before giving up:

* A missing or truncated `%%EOF` marker or `startxref` pointer: the
  cross-reference table is rebuilt by scanning the file for indirect objects.
  If no trailer dictionary survives, the document catalog is located via its
  `/Type /Catalog` entry and used as `/Root`.
* A `startxref` offset pointing outside the file: the cross-reference table
  is rebuilt as described above.
* Wrong object offsets in the cross-reference table: entries are repaired
  from the objects found in the file (the last occurrence of each object
  wins); only objects that cannot be found at all are dropped.
* A stream object without a `/Length` entry: the stream data is read up to
  the next `endstream` or `endobj` marker.

If none of this yields a usable document, `PdfReader` raises
{class}`~pypdf.errors.PdfReadError`.

With `strict=True`, none of these repairs are attempted and the first
violation raises an exception.
