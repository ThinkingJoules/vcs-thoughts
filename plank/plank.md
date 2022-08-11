# Plank
This is more or less a new Git. However, we are building off the core primitive: [blueprint](../blueprints/blueprints.md)s. 

Plank works primarily with the File blueprint and can *generate* a Dir blueprint. The Dir blueprint is not used internally in Plank. 

This is simply *my take* on a new VCS. However, if others worked from the raw blueprint spec, there could be many different VCS, with the blueprint spec as a common repo exchange format. 

My goal is to do more than Git. Personally, I want to collocate most of Github within the repo, to allow for intra-repo links. This will allow for comments/threads and other metadata to not muddle up source code, but have the ability to 'overlay' this information. This could also be stored locally to have all the information of the Github repo offline, if you so choose. 

My main goal with Plank, is threads with comments.

## Like Git...
This system stores the files in its own format. Git unpacks the repo into the local file systems so everything 'feels' like normal files. However, they are logically stored in the .git folder. This convention makes sense. We will need a .plank folder locally that will 'compile' documents and present them to the file system.

Like Git, we can extract or get additional information from this directory.

Like Git we will have some global configs installed that will be used to display documents to your preference instead of the author's. These would be things that we could feed into the blueprint assembler like: indentUnit, subLineSpaces, or lineEnding.

## Unlike Git
### External Content
As I will go into more detail later, I want to remove all content addressed data from the core version control trie. This allows two things. First the trie is much smaller and easier to work with if all the data is removed. Second, because hashed content is deterministic we can de-duplicate locally if we had many repositories. Probably not going to save a ton, but if you scaled up to a Github level, then the savings would probably be quite large. 

Dealing with content addressed data is relatively straightforward. You only need to know the hash function. So any 'values' that are hashed in the following descriptions will have a *slight* networking dependency on that content. Again, being hashed data there is no concern of authorship, and you can cache as much or as little of a repo as you want locally. 

Either you are creating the chunks locally or you are getting the repo from a network address, who *should* also have all the chunks you need. 

I know I was anti-network in the readme, but that was in regards to the naming of the repo itself. Content addressed stuff, has no naming or authorship problems. So, while it is part of the implementation, it is not really part of the *specification*. 

If we are building from the blueprint and (*if*) that spec specifies a hash function, then we will use the same function. If it doesn't specify a hash, then maybe we shouldn't either? The network layer is where interoperability is going to happen. So perhaps it should be at that level that the hash function is picked?

Either way, we do not specify here, how to get the chunks that we externalize, just that there needs to be a mechanism to store and get them.
### DB vs FS
(Note: I don't know how Git works under the hood, but I suspect it is file-based)

What is the difference between a database and a filesystem? I think the only distinction we care about here, is that you can emulate a filesystem inside of a database, but not the other way around. My current plan is to treat Plank as a database storing a filesystem.

Why do we care? Because if affects how easily it is to add metadata to a document, as well as how easy it is for anyone to extend and add their own metadata.

# Current Design
## Database
[Idea for a 'simple' database.](./db.md) **tl;dr Use an append only file for simple crash consistency, that stores a copy-on-write tree (specifically a PATRICIA Trie)**

## Plank File System (PFS)
Since we are emulating the version controlled filesystem, I think we extend that idea to our Plank database. We will structure our database keys as a pseudo-filesystem. We shall call this the PFS to differentiate from our version-controlled file structure (VCFS)
### VCFS Keys
We need a directory to store the VCFS. Since this is the entire point of what we are building, it should simply reside at '/'. So if you have a single file in your repo 'readme.md' then in the PFS we would have a key in the database of '/readme.md'. 

However, since this file *doesn't really exist as .md*, we really don't have any data at this *exact* key. Instead I propose that *all files* in the '/' folder get the extension of '.bp' (blueprint) appended, and the value of that key is *a hash of our blueprint file*.

Using a hash for indirection accomplishes two things. It allows us to keep the PFS as small as possible regardless of blueprint size, and this hash will always line up with *external systems*.

So we really have:
```
/readme.md.bp = 0xHashOfBlueprintFile
```
I think appending an extension shows intent. I don't think the blueprint file is metadata. It *represents* the actual file. There is a bidirectional translation between .md <-> .bp.

So let's talk about both file and directory metadata, and what those might look like.

### VCFS Values/metadata
I think it would be nice to *be able to* dereference the *entire VCFS* and view it directly in the filesystem (no translation). It won't be perfect, but I want it to be possible. This means that any metadata we add *within* the PFS, *must not collide with actual data in the VCFS*. We can't know for sure it won't collide, but we can lower the collision chances by prefixing everything inside of the PFS.

I think we can use the hidden file and directory naming convention and start the prefix with ".". So we could say all metadata *inside of PFS* will have a prefix of '.pfs_'.

So it would look something like:
```
/readme.md.bp = 0xHashOfBlueprintFile
/readme.md.bp/.pfs_lastModified = ISO8601
```
Directories are technically encoded into the trie structure, but we still have metadata if we want to properly 'fake' the OS file system by changing the created date to be well in the past of when we just created it.

I think we could do something like this:
```
/someDir/.pfs_createdDate = ISO8601
/someDir/.pfs_lastModifiedDate = ISO8601
```
I think the directory will probably not store any additional or duplicate info. We could always *generate* a Dir blueprint if we wanted to, so there is no need to store that information in here.

If we did export all of our blueprints we would be creating a hash-linked structure that is 'timeless' (no modified data or anything in it) snapshot of an entire repo that can be used outside of the Plank system ('compiles' down to blueprints). 

With this hash-linked structure we wouldn't *have to* merklize our prefix storage trie, as we could just hand out the VCFS root folder's ('/') blueprint and anyone can verify the retrieved content (gotten from anywhere).

The downside, is that there is no 'succinct' way to prove any *particular* file is part of a *particular* snapshot. Our blueprints form a hash structure, but it isn't really setup for proving things, it is more for linking things. So if we need 'simple' ex/inclusion proofs of a deeply nested file, then we would need to merklize our trie and send that. The efficiency tradeoff depends on whether a directory has more sub-directories than the chosen radix of the underlying PATRICIA Trie. This would need some research.

### PFS Keys
Since our repo starts with '/', any key that *doesn't* start with '/' is a PFS directory or file.

What are some keys? Well if we want to store threads then we might have a 'threads' prefix, or 'commenter' (user), etc. The values are obviously going to be Plank specific. 

Perhaps there will also be an extensible directory for all non-plank things. Maybe all things that aren't the repo are simply 'non plank' things. 

What is Plank? 

I want comments, but just because I'm the author I get a special namespace? Not sure how to 'safely' modularize Plank... I feel like if someone wanted to make a large change that I disagreed with, they would build up from the blueprint level.

For more info on Plank 'features' [read this document](./uri_usages.md)

# Summary
FEEDBACK! I want to rough out what a network layer might look like with a combination of these two lower levels but there is a lot that is open to discussion.