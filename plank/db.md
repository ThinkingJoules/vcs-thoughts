# Database
I like the idea of a database, but I like to be able to read the stored data myself. So while this is going to be a 'database', it will be stored in a single append-only file.

To me, the most obvious candidate for the structure of this database is a copy-on-write tree of some sort. 

Since we are working with a filesystem in the abstract, using a prefix trie (PATRICIA is my preference) makes a lot of sense. It naturally emulates a hierarchy, but in a tree form (inner nodes can have values, and you 'build' your key as you walk down the trie). It has the added benefit of being compact on disk. 

What sort of degree or radix might this trie be? If we are using this exclusively for text-based (utf-8) documents then we could do a 256 radix trie. This requires keys to be byte aligned (no 4 or 6-bit keys). Utf-8 chars should always be whole bytes, so this is fine. So each node can have up to 256 children. This would represent one child for each 'next byte' in the key. 

If we wanted to merklize this structure to prove a value, then 256-ary trie will cause a large proof size compared to the best case 2-ary (bi-nary). This would be a consideration if the 'authorship' model took the digital signature route (determined in the network layer). Otherwise the 'aryness' of the tree doesn't make a ton of difference. A 256-ary trie would probably be a 'better' node size for reading off disk more efficiently than a binary trie. If you wanted to compromise, you could turn the keys into hex, and use a hex-ary trie. It would provide decent proof size, but not suck so bad reading from disk. This should really only be explored if the repo trie itself needs merklizing.

The file can be 'compacted' and 'unofficial' versions discarded. During this process, the repo is unusable. We pay for our various write amortizations in one shot.

# Generalize?
I would like to generalize the append-only-log-tree-database idea. I think it could be helpful to store many things using the same 'understanding' of how it works. 

If we use it extensively internally, there is no reason it can't be generalized and used elsewhere. It won't be high-performance (?maybe it could?), but at least in lower velocity applications like ours, it could be super simple and easy to reason about (if you are someone like me who wants to *know* how a thing works before using it).

The key thing would be to allow cross-file-crash-consistency. I think this could only be accomplished if we generalized the idea to be a 'configurable' database-as-a-file concept. This could allow us to store ACID transactions across files. That way the 'parent' database could read through the child databases and find a recovery point that was written to all files successfully.

The downside is that compaction gets more complicated. However, I think even for Plank, I want two files. One for the main copy on write VCS system, and a second tree to store all the commits and checkpoints. This is needed because during compaction, we need a list of 'roots' that anchor the trie nodes. Any root not on the list can be deleted, and anything below it will (potentially) be freed. You also have the problem of trying to save an internal reference to the tree itself. You always have to 'double' commit. Once for the data, and the second to commit the commit. So having two trees is still two commits, but it also separates concerns. 

Since we have crash consistency, we could spin up a database *just to do compaction*. This would allow for a super-robust-never-lose-data system. 

I think generalizing this seems like a really useful tool for dealing with data in a 'safe' manner. Crash consistency and anything that touches a hard drive scares me. Between the OS and the Disk Controller, you have very few 'guarantees'. Depending on the disk driver, a successful return from fsync can still result in data loss. This is because the disk drive is caching it and lies to the OS.
