# Leveraging URIs
Finally getting around to the whole reason I started this project, COMMENTS!

# Links
As we change the repo, our URIs might change, but we want all things pointing to that URI to 'move' with it. So we need a layer of indirection to make this work:

```
links/bpl:/path/to/file.ext?chunk=0xChunkHash,idxOfHashCollision&subset=offsetInChunk,len#0xHashOfBluePrintFile
```
This is simply 'links/' + the blueprint link URI in its entirety. If *anything* in this URI must change, we need to 'delete' this key, and then re-insert the associated value with the 'new' URI. Or, if it is no longer, move it as-is to the archive.

What is the value? I think it is some sort of 'lookup' file that would contain things like:
```
{
    threads:[threadID,...],
    refsInBps: [bpl:/path/to/file.ext.bp, ...]
    commentRefs:[commentID]
}
```
- threads: Some sort of unique identifier that sorts nicely for querying. Probably something to do with created time as the ID.
    - Not sure it needs to be an array, but I suppose a thread might have a 'topic' and so more than one thread could be talking about, say, a whole file, or a directory?
    - This is used to state *this is the home* of this thread. More or less, where should we render this in a UI.
- refsInBps: For link migration.
    - When a bp is committed we scan it for bpls and record them here. 
    - Can migrate links without rescanning the entire database for references.
- commentRefs: Same as refsInBps, but for links *inside* of a comment. (either in a thread or issue)

There might be more things to put here, but the idea is this should be a pretty low velocity file. We need to read the whole blob into memory to make even a small change. However, keeping it a small blob makes moving to a new key very easy. There is no need to 'grab all keys' in the subtree or anything like that.

# Comments
These are used in two places, Threads and Issues, since both are logically the same.

These are stored:
```
comments/ISO8601_COMMENT_CREATED_TIME_UTC
```
The ISO date should sort nicely and we can do range queries for 'recent' comments.

We could extend the key for 'dual linking' and put '/thread/threadID' or '/issue/issueID' to allow us to assemble 'views' either way. The only reason to do this is to allow some tiny bit of indexing so we can use the Plank DB with pretty good performance without needing to index in a separate database.

# Threads
Comment on *all the things!* Well, not literally. I think you can only comment on things in the '/' VCFS.

Threads will be stored in:
```
threads
```
Since these really aren't 'tied' to anything, we simply create a key based on the date created. Since we would like to easily get 'recent' threads, or organize things by time, I think a thread ID will simply be:
```
threads/ISO8601_THREAD_CREATED_TIME_UTC
```
The value at this key will be a string representing the 'Topic'.

For comments we extend the key, to allow range queries:

```
threads/ISO8601_THREAD_CREATED_TIME_UTC/ISO8601_COMMENT_CREATED_TIME_UTC
```
The value at this key is simply a boolean, in practice, the value is never read. We are using our Trie as a query/index structure.

# Issues
These are the same as threads, but they don't have a 'home' in the VCFS. These are simply 'threads' with a 'topic'. Much like how Github issues work.

```
issues/ISO8601_ISSUE_CREATED_TIME_UTC/ISO8601_COMMENT_CREATED_TIME_UTC
```
We treat this key the same as threads, it is simply a way to do a range query.

