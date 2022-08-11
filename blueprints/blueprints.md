# Turning a File into a Blueprint
To illustrate how this works, let's walk through how to take a new file and create the indirection file, the *blueprint*.

(Note: some of these steps can be combined, just walking through one at a time for illustration purposes)
## Step 1: Normalization
For the blueprints I propose we store the files in a normalized line-ending manner. I think the computer world is trending towards simply using \n to represent line returns. So if things are \r or \r\n then we need to clean that up first. So the first pass through a document is pretty simple and easy.

```
File -> NormalizedFile
```
## Step 2: Line Based Chunking
Now we must extract all the lines.

```
SplitOnNewline(NormalizedFile) -> ChunkList
[
    Chunk0{
        firstByteOffset: Number, //index-0 of normalized file.
        chunkLen: NumberOfBytesInChunkWithoutNewLine
        endsWithNewLine:true
    },
    Chunk1{
    ...
]
```

So now we have a bunch of slices of a single file.
## Step 3: Sub-line Chunking
[Chunking Doc](./chunking.md). **tl;dr we define a regex fn that we call the 'chunker' to determine where to split within a line**

```
Chunker(ChunkList,NormalizedFile) -> ChunkList[
    Chunk0{
        firstByteOffset: Number,
        chunkLen: Number(BytesInChunkWithoutNewLineChar)
        endsWithNewLine: true || false
    },
    Chunk1{
    ...
]
```
The Chunker alters the chunkList by 'splitting' the newline chunks. The new chunks are spliced in before the 'endsWithNewLine:true' chunk, with the newline chunk getting a new offset and length value. The spliced-in (new) chunks will have 'endsWithNewLine:false'

## Step 4: Trim Chunks
[Trimming Doc](./trimming.md)  **tl;dr we try to remove formatting information from chunks and store leading whitespace as metadata**

```
ChunkList = (from step 3)

trimmerStart = detectIndent(ChunkList,NormalizedFile)
trimTrailingWhiteSpace = t/f

TrimChunks(ChunkList,NormalizedFile,trimmerStart,trimTrailingWhiteSpace) -> 
[
    Chunk0{
        firstByteOffset: Number,
        chunkLen: Number(BytesInChunkWithoutNewLineChar),
        endsWithNewLine: true || false,
        indentations: Number,

    },
    Chunk1{
    ...
]
```
The TrimChunks step may adjust both the firstByteOffset as well as the chunkLen depending on settings. If the firstByteOffset is changed then the number of 'indentations' will be non-zero. At the moment, we do not record any information about the discarded whitespace at the end of the chunk.

## Step 5: Process Chunks
Now we can hash the exact byte slice that has the content we want to version control.
```
ChunkList = (from step 4)

ProcessChunks(ChunkList,NormalizedFile) -> 
[
    Chunk0{
        firstByteOffset: Number, //index-0 byte offset from start of normalized file.
        chunkLen: Number(BytesInChunkWithoutNewLineChar),
        endsWithNewLine: true || false,
        indentations: Number,
        hash: Hash
    },
    Chunk1{
    ...
]
```

## Step 6: Create Blueprint
[Rationale on shape and syntax of file.](./blueprint_file.md) **tl;dr Blueprint has a 'header' and below that is the template section. The template section encodes the indentationUnit as a \t (tab character), and the \n (newline characters) are literal**

Here is a proposed layout (//comments are not actually allowed, only used to convey meaning in this explanation):
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
## Step 7: Store Chunk Hashes
This is covered in another document, but for the most part we have some sort of content addressed store that could contain chunks from many blueprint documents (for deduplication). We just need to send a PUT(hash, NormalizedFile\[chunkStart..chunkStart+chunkLen])

# From Blueprint to File
So now we run the process in reverse, without the cleaning steps, and with viewer-defined-variables.

## Step 1: Get Chunks
```
chunks = {}
for each Hash in Blueprint.template{
    data = ChunkStore.get(Hash)
    chunks[hash] = data
}
```
## Step 2: Build Bytes
```
viewerIndentUnit = getIndentUnitFor(Blueprint.header.ext)
viewerLineReturn = getLineReturn()//should be environment specific? Maybe pass the file extension?

document = Assemble(chunkDict,Blueprint,viewerIndentUnit,viewerLineReturn)
```
The two get functions for the viewer's preferences can come from a global config setting, or if in the browsers, a user session. The lineReturn might be omitted in the browser since it probably has its own spec.

Assemble basically goes through the template and simply does a few replaceAll calls:
1. replaceAll(\t,viewerIndentUnit)
    - viewerIndentUnit could be any string that does not contain \n
2. replaceAll(\n,viewerLineReturn)
    - viewerLineReturn must strictly be one of \r, \r\n, or \n
3. forEach(Chunk).replaceAll(hexEncodedChunkHash,chunkDict\[hexEncodedChunkHash])

Running replaceAll is purely for illustrative purposes. The template would probably be processed in serial manner, and bytes are appended to a buffer as we progress.

