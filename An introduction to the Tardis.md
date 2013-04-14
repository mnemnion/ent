#An introduction to tardis#

tardis is a new approach to code creation. It is (will be) an editor that works on parse trees, rather than on files, and registers all changes as 
[operational transformations][1]. It does so through the medium of familiar code files, but these may be thought of as an interface to the code, and as a product of it, similar to the executable binaries produced by a compiler. 

[1]: http://www.codecommit.com/blog/java/understanding-and-applying-operational-transformation

It's worthwhile to visit the link on operational transformation, even if you're familiar with the concept. It's at the heart of what tardis will do, and why it can be so phenomenally powerful. 

When reading it, whenever you see the word 'client', think 'the present'. when you see the word 'server', think 'the past'. While OT can be used for simultaneous editing of a single code base, and this has implications for tardis, it is this handling of past state which is half of what makes tardis worth creating. 

