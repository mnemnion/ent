#An introduction to ent#

ent is a new approach to code creation. It is (will be) an editor and library that works on parse trees, rather than on files, and registers all changes as [operational transformations][1]. It does so through the medium of familiar code files, but these may be thought of as an interface to the code, and as a product of it, similar to the executable binaries produced by a compiler. 

[1]: http://www.codecommit.com/blog/java/understanding-and-applying-operational-transformation

### Parse Aware Editing of Structured Files
ent's major rationale is parse-awareness. It will, in general, not allow you to type invalid code, though this can always be overridden. It will parse your code as you create it, storing the resulting file as a series of operational transformations on the parse tree. As a language is more thoroughly defined within ent, this enables REPL-like instant feedback and sophisticated refactoring. 

A simple example in JSON  will get us started. We are editing this file:

```JSON
[  {"foo":"bar"},
   _
]
```

Where '_' represents the cursor. We type '{'. 

Because we are in an Array context, and the only rule that can match { is Object, we get:

```JSON
[  {"foo":"bar"},
   {_:**}
]
```

where "**" represents the target, which is the next place that ent expects us to move. 

We now type 'q'. Because we are in the Key context of an Object, this is not valid. But ent is friendly, so we get this:

```JSON
[  {"foo":"bar"},
   {"q_":**}
]

```
Since JSON expects a string, the "" would actually be auto-inserted after the '{'. This example was somewhat contrived to show how ent can handle erroneous input through parse awareness.

Note, as an aside, that github's JSON lexer identifies ** as an error. This points to the advantage of parse aware editing, which can go far beyond syntax highlighting (as well as getting that task more correct than line-based regexes can).

We continue typing 'ux' to give 'qux'. Either " or the right arrow key closes the string and gets us to our target:

```JSON
[  {"foo":"bar"},
   {"qux":_} **
]
```
Note that the target has moved also.

All of this special magic is enabled by the fact that ent cooperates with a parser to parse, validate, and transform the file as you type. ent will also have a 'permissive' mode where the user may make arbitrary changes to the file; upon returning to opinionated mode, ent will reparse the edited regions and try and make sense of the input. 

This flavor of convenience is well known to IDE users; ent generalizes this, but aims to do so in a way that has deep and far-reaching implications. 

###Continuous Comprehension#

As programmers, we want to stay close to our code as we work with it. The move from batch processing to interactive compile-run cycles was a boon, and the development of REPLS took us further, but we remain in a state where we interact with mutable flat files. 

We would like to be in a state where we interact with immutable trees that embody not only the state of our program's encoding, but every state the program has ever been in. Graphic designers and CAD technicians have had this for decades; one may take the typical Adobe Photoshop file and run it backwards to the very first edit.

What is holding us back is that flat files are the ubiquitous interface between tools in the programming chain, and they are mutable by default (indeed, to a fault). ent aims to do the least possible to allow the move to immutable tree-based code structure while maintaining flat files in a sensible state and allowing other tools to act on and transform those files as input into the code structure. 


###Deep Waters

This is not a trivial move. If we were free to design our own ball game, we could start with immutable data types, define transformations on them, and start snowballing. 

ent isn't that kind of project. The entire towering edifice of computer software is built on transacting mutable text files; with few exceptions, everything running has a canonical form as a collection of such files, which is interacted upon by various tools to generate the running code. 

ent wants to thoroughly change the method for generating that collection. Current best practice is revision control; ent extends that to dizzying heights, so dizzying, in fact, that it uses existing version control to keep matters from getting totally out of hand. 

To say that modern code bases are mutable text file collections is imprecise. In most cases, they are exactly that, backed by a revision control package such as git. This manages those aspects of history and difference which the user has decided to record, and is an improvement. 

ent stores and understands the code base as a series of operational transformations on a parse tree, and treats flat files as the canonical interface to that parse tree. let's break that down some before we continue.

To the user, a flat file is exactly what it was before. ent allows you to do whatever you want to it, in permissive mode, and the restrictions of opinionated mode are there to help; the user experience we're going for is one of free typing, with transformations, reformatting, and opinions offered in real time. Trying to type syntactic nonsense won't work in opinionated mode, which is very much the point.

To ent, proper, edits on the flat file are not seen. ent tracks the cursor position within the parse tree, and reparses the input periodically (specifically when the cursor crosses rule boundaries). It is the result of those parsing actions that ent tracks and stores, as operational transformations on the code base itself.

That is the minimum necessary to interact with a flat file through ent: a parser which meets certain interface criteria and produces a parse tree that contains every character in the original file. We can add more, but we cannot have less.

Even a small amount of structure can be useful. If we divide English into words, sentences, and paragraphs, ent will keep track of each one of these entities as they come into existence, and allow us to do many of the fanciful things ent makes possible, such as correcting a typo in a way that propagates to multiple copy-pasted versions of a sentence. 

Importantly, even a poor or ambiguous grammar will work, as long as the parser's output is deterministic. The guarantee that ent will make you is that any tree it makes, if walked from left to right, will give you your string back. 


###Branch and Merge

Because ent uses operational transformation, it provides great flexibility and control in branching and merging. OT is used in products like Google Docs so that, if network problems cause two user edit streams to diverge, the edits can be merged automatically into a single canonical document as soon as connectivity is restored. 

ent approaches OT differently. Where Docs etc. are concerned with synchronizing multiple versions of a single canonical file in near-real time, ent uses OT to flexibly handle multiple branches and merges of a single code base, which may be replicated elsewhere with optional differences. 

It is the same underlying algorithm, and it leads to a substantially different approach to branching and merging than that embodied in programs like git. Ultimately, ent enables time travel; you may return to any point in the history of your project, make revisions and changes, and propagate them, with control, back to the front of the project.

Let's contrast this with git. In git, to make a branch, you tell git you want to make a branch. git takes a snapshot and starts tracking changes under a different name. If you revert to the original branch, it goes back to the snapshot and tracks a different set of changes. When you merge, if all goes well, all the changes from both branches are reflected in the new file structure.

With ent, the user does not have to decide to branch. They may simply rewind to the point where an alternate path is helpful, create it, and merge. If all goes well, you have a new reality, in which the old edits happened in the past. 

That's why we still use git within ent, for the record; when you start doing time travel, you start to wonder, sometimes, what reality used to look like. git, enslaved to ent, will serenely keep track of all this. 

###Time travel

Here's some unavoidable terminology: in ent world, there are two universes. In Universe A, time is entropic, irreversible, and can only be queried as to prior state (and only through the mechanism of recording that prior state). In Universe B, time is reversible and mutable, with a higher order that immutably tracks the paths of that mutation and can unwind the skein accordingly. 

What? Say I have a file, and I rewind time to rename a function foo() to bar(). I have a path of git revisions that say that at such-and-such a time, my file structure contains certain data. Since I haven't done any time traveling (it's not for the faint of heart), Universe A (git land) is the same as Universe B (ent space). 

So I rewind time, past several git boundaries, and merge. It works. Now, if I go back in time in Universe B, my function is called bar(). If I go back in time in Universe A, my function is called foo(), until the moment that I went back in time, at which point it's called bar(). 

Universe A is reality, as it happened. Universe B is reality as we wish it happened. They are a powerful team. 

###Branch and Merge, again

The model is in principle no different when multiple authors work on one code base. ent tracks who made each change, in addition to what the change is, as part of the atomic transformation. 

To the degree that one ent is aware of another, they may trade branches. Moreover, when changes are propagated up the time stream, they may be offered to such other ents as the propagating ent may be aware. 

That's a lot of maybes. This is code we're talking about; handle with care. The current paradigm is pull-only for revisions; ent can provide notifications that changes are available, and hand those changes off, but pushing code willy-nilly is a bad habit to get into. 









