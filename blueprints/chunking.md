# Chunking
Git is strictly line-based. I want to allow regex within a line. So we can define a sub-line-chunking regex. We will simply call this a 'chunker'.

So for code, we really don't need a chunker, since the line-based first step of chunking is already done for us.

For the chunker we will use regex. I propose [Perl Compatible Regular Expressions (PCRE)](https://en.wikipedia.org/wiki/Regular_expression#Perl_and_PCRE). It seems the mostly widely used flavor, if nothing else it is used in javascript and javascript is everywhere. 

## Sentence Chunker
I think the sub-line chunker is almost always going to work on human syntax sentences. Because of this,we might have a lot of different flavors, and potentially quite a complex regex fn. Most sentences are pretty easy, but I bet there are a lot of edge cases for things that are not sentences but have similar patterns.

We have to be careful that any incorrect splits work fine with our trimming function to ensure we don't remove whitespace that is needed. I assume the complexity will end up in one regex function or the other, else it is going to get confusing.
