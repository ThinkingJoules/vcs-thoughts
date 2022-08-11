# Trimming
The goal of this is to allow people to render their code using tabs or spaces, in any number, regardless of author's preference.
## Tabs vs Spaces
In code we have the eternal indentation character argument. Is a single indent a single tab, a single space, 2 tabs, 4 spaces, etc.? Indentation is basically the standard method all human-machine readable formats take to improve human readability. Indentation itself doesn't really carry any logical information. I think some formats use the whitespace as part of the machine information, but I don't think there are very many of those, and I imagine they are very forgiving when they parse the code. 

## Sentences start with one space or two spaces.
Less of a debate is [starting a sentence with two spaces](https://english.stackexchange.com/questions/519320/where-and-when-did-the-practice-of-using-two-spaces-in-the-beginning-of-each-sen). Again it is purely for making a document easier to read and carries no information.

# Let's end this madness!
These both have the commonality of white space characters at the *beginning of the chunk*. More or less *formatting*, not content. Why can't we process the chunks and extract this information, thus shrinking the chunk to be *actual content*. Let's call this a **trimming function** or 'trimmer' for short. 

Do we care about trailing whitespace? This is less a matter of 'counting' and more a matter of removing trailing whitespace altogether. Probably just a 'yes/no' boolean? TBD.

## Code Files
In a code file, we need to run a detection algorithm over the file to 'learn' the source document indentation type. We need to figure out two things, which character is used (SPACE or TAB) as well as the number of those characters that is considered for a 'single' indent. So we need a **detectIndent function** that returns a proper indent regex. We can use this regex function to then give us the indentation count for any particular chunk.

## Sentences
For human sentences, we really just need to remove all the whitespace and insert a single 'indent' character in our blueprint. There is no nesting of indentation, so no detection is really needed. Then on document building, we can insert a document-viewer-defined number of spaces (or tabs!) at the beginning of the sentence.