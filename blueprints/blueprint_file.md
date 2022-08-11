# Blueprint Syntax
The core issue here, is that a file is not only chunks, but also each has an indentation pairing (and sometimes a newline suffix). 

## Basics
I propose that we encode the indentation and line return metadata directly into the structure of this file. So if opened in a text editor, it matches the 'shape' of the final document.

Code would be:
```
{Chunk hash} //chunk 0, 0 indent
	{Chunk hash} //chunk 1, 1 indent
{Chunk hash} //chunk 2, 0 indent
```

Whereas two sentences (with appropriate chunker) would be:
```
{Chunk hash}	{Chunk hash} //1 tab in-line = subLineSpaces
	{Chunk hash}	{Chunk hash} //1 tab start of line =indentationUnit (i.e. paragraph indentation)
```

So we have two literal characters at the moment: \t = IndentUnit and \n = newLine. The \t is interpreted based on where it is found. If we wanted we could add a third literal char ' ' for the subLineSpaces whitespace for differentiation. This could make it easier? TBD.

## Linking
We have a [link URI](./uri.md#linking) so I think we should allow it to be in a blueprint file. This would allow for assembling of synthetic documents from other documents within a repo. I'm not sure the precise application yet, but I feel like if we have URIs then we should be able to use them!

So we could do:
```
{Chunk hash} //chunk 0, 0 indent
	bpl:/path/to/file.ext#0xHashOfBluePrintFile
{Chunk hash} //chunk 2, 0 indent
```
Basically, we are wanting to inline this entire "file.ext" from our repo (with one additional indent) between the other two chunks listed. This is a pure concatenation operation, so this other file would need compliant syntax for whatever you are doing. This can be a whole file, chunk, or subset of a chunk.

## Blueprint: File
I think we want a bit more info in the blueprint file, but not too much! We want the hash of the blueprint file to only change if *content* in the file changes. Here is a proposed layout:
```
type: File
ext: "js"
name: "human-machine-readable-file"
chunker: 0xHashOfRegexFnUsedToChunk
⸻
{Chunk hash} //chunk 0, 0 indent
	{Chunk hash} //chunk 1, 1 indent
{Chunk hash} //chunk 2, 0 indent
```
This does mean that if you change the name of the file, it will be marked as 'edited'. Without the name, the blueprint cannot stand alone. 

Is this important? I think it is for the networked layer. We want the ability for anyone to 'snapshot' a document to prevent broken links. We can embed a local URI into a network URL to help in orchestrating this 'save a local copy' feature. Allowing files to stand alone can allow other systems to build on these ideas. Ultimately Plank is built off this lower level 'blueprint' specification.

We could allow additional metadata in the header — however, if used incorrectly it will 'break' the usefulness of our content addressing of this whole blueprint file. So this is something to discuss. Metadata *inside* the blueprint file is really *only* for using blueprint files *directly*. Ideally a larger system should encode metadata *alongside* the blueprint.

## Blueprint: Dir
In unix, everything is a file, so we probably need a corresponding Dir(ectory) blueprint as well.
```
type: Dir
name: "someDir" (note: no '/' allowed)
⸻
0xBlueprintHash
0xBlueprintHash
0xBlueprintHash
...
```

# Specification
If stored in a filesystem, the extension is '.bp'. 

Blueprints are UTF-8 text documents.

For the moment all hashes, regardless of what they represent, are lowercase hex encoded with a '0x' prefix.

The tentative hash algorithm is [Blake3](https://github.com/BLAKE3-team/BLAKE3/). Git uses a (now broken) cryptographic hash, so I don't see why we shouldn't go with the latest and greatest (cryptographic) hash. We could truncate the hash to 160bits (20 bytes). I think this is safe, and will save pretty significantly on storage costs. Ethereum uses Keccak-256 and takes the lower 160bits. If that is enough security for ETH, then I'm fine with 160b (same bit length as SHA1). **Do we need to include this in the spec??** If we are concerned with the blueprint spec being an interoperable exchange format... **TBD**

The separating character between the header and the template is currently a [Three-Em Dash](https://www.compart.com/en/unicode/U+2E3B) (U+2E3B) "⸻". I think I want a single character and something that feels like a division marker. I'm flexible to change it. It is 3 bytes, but I think the only single byte char that is dash-like is going to be '-' or '~'. The advantage of '-' is that it is on most keyboards given math is everywhere and it represents subtraction. I'm open for discussion here, so TBD.

