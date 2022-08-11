# Version Control Thoughts
I wanted to do something that Git wasn't conducive for. I wanted to version control my text documents about notes on ideas and problems I'm working on. I also wanted to add metadata for comments/threads at the sentence level.

> Problem 1: I wanted to create comment threads on specific lines of version controlled code that would not be associated with a line number.


> Problem 2: I wanted to split on patterns that were not line endings. 

Solution for problem 1: I need a version-control-native URI scheme to reference a version controlled chunk (sentence/line). Git does not have such a thing (that I could find).

Solution for problem 2: I need to use a regex function to create sub-line version controlled chunks. Git does not allow you to change away from being line-based.


## A new VCS with...
- User defined chunking function
- Native URI scheme that allows for addressing down to a single character in a version controlled file. 
    - Think of this like a *selection* or *highlighting* feature. It should have enough information to highlight a contiguous set of version controlled text (bytes).
- Locally configured spaces vs tabs formatting
    - Extract 'non-information' from beginning of chunks and store as metadata in the VCS.
- Integrated issues,comments/threads,wiki,etc. (metadata) *about* the version controlled files.
    - Due to the nature of namespaces/URI concerns I think for practical purposes we *must* co-locate them. The goal is not the co-locating, but the idea of **having an extensible metadata system**.


The metadata can leverage the URI scheme for creating references *within* the repo.

Like the Git protocol, the low-level VCS would avoid any networking/urls directly in the protocol. It should be similar to regular Git in many ways. I think this new VCS would more likely be a series of functions that can be integrated into something larger. I'm thinking of this more like an ABI and less like an API (so a specification more than any specific implementation). The hope is that there can be many clients and implementations, and much hacking on this system.

## Docs
Tentative name for this project: **Plank** (a nod to replacing planks in the [Ship of Theseus](https://en.wikipedia.org/wiki/Ship_of_Theseus))

There are three layers to making this whole thing 'usable'. This repo is mostly focused on layer 1 and 2.

1. [Blueprints, the core primitive](./blueprints/blueprints.md)
2. [Plank, which builds off and extends blueprints](./plank/plank.md)
3. Network Layer (Frame?) - TBD

-------
# No networking
This project does not want to tackle namespace issues and follows Git's design. However, in order for this to be useful it *will* become networked by higher level applications. We should keep this in mind when building the low level VCS.
## Adding Networking
We live in a much more connected world than when Git was invented. I think consideration should be taken at the low level to allow a higher level wrapper to interact with other computers. Git uses something called a bare repository where the only possible operations are Pushing or Cloning.

What are some consideration for wrapping up a low level VCS with a network? I think there is only one. That is the namespace problem.

### Naming
As soon as you add networking, the repo must be named. Git itself doesn't care what the parent directory name is. However, once you create a bare repository, you must give *the bare repository* a name. So how can we reference other repository "selection" URIs? We would need a namespace. Right now, almost everyone uses DNS, either directly, or indirectly. You can host a Git repo on your own Domain Name, but almost everyone uses github.com.

**How you find your repository is how it is named**. 

If we want to avoid conflicting names, there needs to be a user controlled prefix to the name. This repo currently has a prefix of : github.com/ThinkingJoules/

Any repository with this prefix is 'mine'. What does that mean? Nothing more than that **I assert the control** on who can read and write to anything after the prefix. Notice this is a *network* action. The low level VCS doesn't care. **It is gated by the network.**

So what if I had the same named repo on both Github and Gitlab? I can have a URL to both, but unless you compare the commits and history manually, they don't have to be the 'same'.

Put another way: **There is only ever a single source of truth**

We *can* cheat this a bit. We can say two repos are identical regardless of network location if they have the exact same commit hash. (Note: Git encodes the previous commit hash, thus the 'current' commit encapsulates the history.)

We *can* cheat the authorship problem using a digital signature. But what is a more robust namespace: keeping a Domain Name or not losing a single private key? Since the entire world depends on DNS working it is probably safe to say that it isn't going to suddenly stop working. So hosting your own git repo at your own Domain is probably pretty solid. What about using Github? While convenient and free, using github.com as a namespace means that your 'authorship' only lasts as long as Github (or as long as they allow you use of their system) - still pretty safe, but it is technically two dependencies (DNS + Github) instead of one.

So in some sense, using pure math has no systemic dependencies. The digital signature makes the namespace portable *across networks*. This is also how we can test equivalency of repositories across networks: hashing (more math!). (My long term goal is to build a distributed PKI system, and need help/feedback, and so am building a better version control system to help...)

### So what should we pick?

**The distinction is whether the low level VCS *requires* a specific network stack to work.**

If the low level VCS can recognize and operate on domain names (or any network address), then the low level VCS cannot fully function if you are offline locally.

Personally, I don't want to deal with network namespace questions for this project. That is up to how this low level VCS is wrapped up. This VCS can only integrate deeply with repo scoped URIs. Any additional integration of network address/authorship during implementation just needs to be aware of what you can and can't do while offline.

If I *were* to pick a network stack (until I get my distributed PKI system working), then I would probably use a TOR hidden service. Why TOR? Because the network address *is* a key that can sign. I don't really care about the hidden or onion routing part, it is just that the digital signature namespace is *directly resolvable*. (and you save the cost of paying for a domain name)
