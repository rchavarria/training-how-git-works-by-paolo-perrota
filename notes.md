# Git is not what you think

Git is an onion

Git is a Distributed Version Control System

Git is a Content Tracker

Git is a Map, a Persistent map in your disk

Git stores everything under `.git` directory

Git is a map, where keys are SHA1 hashes and values are objects, objects stored in `.git/objects`

To see what an object contains, use `git cat-file` passing at least one argument, the hash code of the object.

That is the way to see Git as a Map

Let's see it as a content tracker

Objects can be a commit, a tree, a blob,... (you can type `git cat-file -t <hash>` to know the type of an object). All **objects are stored as text, simple text**.

A commit usually contains a tree. A tree usually contains more trees and blobs. A blob contains **content**

### Versioning

Commits are linked. They usually have one parent commit.

In the commit object, git stores everything. But, if a file or a directory (tree object) didn't change, the old object is reused. But, the commit object references both old and new objects.

You can use `git count-objects` to count how many objects are stored in the repo

### Tags

Tags are objects too

### Recap

Git can store these types of objects:

- blob: arbitrary content
- tree: much like a dir
- commit
- tag: annotated tag, just a link and a name to a commit

Git looks a lot like a file system: it has blobs (content), trees (directories). And they can be linked, even with different names, as soft/hard links in Linux, or shortcuts in Windows.

# Branches demystified

Start with this module

