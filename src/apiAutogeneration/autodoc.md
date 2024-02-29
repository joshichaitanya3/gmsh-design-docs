# Auto-generating inline documentation

Morpho documentation is written in the Markdown format (see [the first section of this chapter](./skeleton.md#help-file)). The `gen.py` file already provides doc-strings for all the functions. In addition to writing them to our help file, we also write the call-signature and I/O parameters to the help file. This clarifies the order of the inputs since this information is not provided in the doc-strings.

Here is a sample output:

```markdown
## gmshWrite

[taggmshWrite]: # "gmshWrite"

Call signature:
gmshWrite(fileName)

Arguments:
fileName (string)

Returns:
nil

Write a file. The export format is determined by the file extension.
```
