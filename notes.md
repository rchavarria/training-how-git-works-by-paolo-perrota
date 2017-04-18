# How GIT Works

Autor: [Paolo Perrotta](https://app.pluralsight.com/profile/author/paolo-perrotta "PluralSight profile"), [course link](https://app.pluralsight.com/library/courses/how-git-works/table-of-contents "How git works")

## Git is not what you think

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

| Porcelain commands | Plumbing commands (advanced) |
|------------------- | ----------------- |
| add, commit, push, pull, branch, merge, checkout... | cat-file, hash-object, count-objects... |
	
**Do not learn commands, learn the model**
	
Git is a map of key-value parameters. For every value, Git calculates a hashcode by [SHA1](https://en.wikipedia.org/wiki/SHA-1 "SHA1 algorithm") algorithm. Every piece of content has its own hash: 20 bites in hexadecimal format.

> `echo "example value" | git hash-object --stdin`:  printout in console the hash value for "example value".    
	 `38830df35f015f8f0f348a6806d8d765dd46d580` 

What if the hash code collides? Two values with the same hash?, it is unlikely to happen, but it could make a mess in your project in case of collide.  

**Git is not just a map, but a persistent map.**

When we initialize a Git project `git init` Git creates a hidden directory *.git/*. Lets have a look:
	
+ **.git/objects/**: this is the object database, the place where Git saves all its objects.  
    - **info/**
    - **pack/**
    - **38/**: "38" is the first 2 hexadecimal digits of the hashcode of the objects. Inside are files whose names are the remain digits of the hashcode of the object. The content of this files is compressed and we cannot open the file as a normal file.
     To see the content of the files we can use the `cat-file` command: 
       
      `git cat-file`: takes the hash of the object and one argument  
        - `git cat-file <hash> -t`: prints the type of the content.
        `git cat-file -t 38830df35f015f8f0f348a6806d8d765dd46d580`

		        `blob` 	   
	    - `git cat-file <hash> -p` pretty printing. Git unzips the object, removes the header and prints out the content "example value".  
	        `git cat-file -p 38830df35f015f8f0f348a6806d8d765dd46d580`  
		
				`example value`
            
### Git is also a "Content Tracker"	
If we have a repository, imagine we want to check a file hashcode and the file is already committed:

- `git log -1`: shows last commit done. 

		`commit c7141b524051805b0b2439dd78923fc29e125e46`      
		`Author: autorName <autor@email.com>`   
		`Date:   Mon Apr 3 13:37:12 2017 +0200`   

The first 2 digits of the commit "c7" is the folder inside *.git/objects/* where our commit is compressed.

- `git cat-file -p c7141b524051805b0b2439dd78923fc29e125e46`  

		`tree e25e243d1b3a26366b6fd20217ee94725c9e87ba`  
		`parent 0c13ed3b382bad2a2751d6b43b4fc961a85f4b2c`  
		`author autorName <autor@email.com> 1491219432 +0200`  
		`committer autorName <autor@email.com> 1491219432 +0200`  
		
This is what a **commit** is, a simple piece of text. Contains all metadata about the commit.  `tree` is a directory of another file. Again, the first 2 digits "e2" is the folder where the file is. Like commits,
a tree is a tinny piece of text and contains a list of the content inside the directory:  

- `git cat-file -p e25e243d1b3a26366b6fd20217ee94725c9e87ba`
		
		`100644 blob 7d11a11a074b256077bd6e1b817db27965959690    README.md`  
		`040000 tree a2c0f673e3bf12ae9f2ee87dbcb7707183b330dd    coursesNotes`  
		`160000 commit 7c9030e456db6064b821dbd878196774cfeb7c92  source`  

	The structure is: `<access permissions>	<type> <value>	<filename>`    
- `git cat-file -p a2c0f673e3bf12ae9f2ee87dbcb7707183b330dd`  
		
		`100644 blob 6b19c6c697851138715d213e96c0740dd951fa74 how_git_works.md`

If we do the same with the blob file, then we will get the content of the *how_git_works.md* file. 
		
### Versioning

Commits are linked. They usually have one parent commit.

In the commit object, git stores everything. But, if a file or a directory (tree object) didn't change, the old object is reused. But, the commit object references both old and new objects.

You can use `git count-objects` to count how many objects are stored in the repo

We modify a file and commit it.

- `git cat-file -p <new commit Hashcode>`
		
		`tree 10348310cc1bebf0a8e25c9a80097153a6944baf`  
		`parent c7141b524051805b0b2439dd78923fc29e125e46`  
		`author  autorName <autor@email.com> 1491220803 +0200`  
		`committer  autorName <autor@email.com> 1491220803 +0200`  
		`<commit message>`

Now we have a new parameter called `parent`, this is the previous commit. **Commits are linked**.
If the content of a file or directory haven't changed, then the hashcode will remain the same and **Git reuses** it!. Git do not duplicate objects that haven't changed. This is one of the reasons why git is efficient.

- `git count-objects` prints out the number of objects and the size of all them together.

Another great point is that Git can store file differences instead of the hole file in the "blob" object. It means, if we modify just 1 line in a huge file, Git will create a new blob object with just the difference (the line modified), not the hole content of the file.

### Tags

Tags are objects too

Tags are labels for the current state of the project, there are two types of tags:

+ **Regular tags**
+ **Annotated tags**, which come with a message. We will talk about this one.

Tags are also objects:

- `git tag -a firstTag -m "first tag with the git fundamentals course`  
`git tag`

		`firstTag`
- `git cat-file -p firstTag`

		`object 31a2e5bbf2cf20f3611fc72ec12c30345d20c17a`  
		`type commit`  
		`tag firstTag`  
		`tagger autorName <autor@email.com> 1491225246 +0200`  
		`first tag with the git fundamentals course`  


The **tag is a a commit attached to an object** (linked).

### Recap

Git can store these types of objects:

- blob: arbitrary content
- tree: much like a dir
- commit
- tag: annotated tag, just a link and a name to a commit

Git looks a lot like a file system: it has blobs (content), trees (directories). And they can be linked, even with different names, as soft/hard links in Linux, or shortcuts in Windows.

## Branches demystified

Git normally put branches inside `.git/refs/heads/`, if we check the content of the current branch (master) saved in the file `.git/refs/heads/master`, in plain text (with `cat`), there is only one hashcode, which is the hash of the last commit `31a2e5bbf2cf20f3611fc72ec12c30345d20c17a`

Let's look at the content of that object:

```
$ git cat-file -p 31a2e5bbf2cf20f3611fc72ec12c30345d20c17a
tree 10348310cc1bebf0a8e25c9a80097153a6944baf
parent c7141b524051805b0b2439dd78923fc29e125e46
author autorName <autor@email.com> 1491220803 +0200
committer autorName <autor@email.com> 1491220803 +0200
```

**Branches are just simple references**. So, `master` is a reference to the latest commit, for example.

Git knows always in which branch we are, the file *.git/HEAD* contains a line that defines our current branch:  
`ref: refs/heads/master`

**HEAD is just a reference to a branch**

To change our branch we have to make a checkout: `git checkout <branchName>`
	
`git checkout` makes 2 things (move head and update working area):  

1. Git changes HEAD and link it to <branchName> branch.
2. Our working area changes to the state where the new branch was pointing at.	

Merge is a commit, but it's a commit with two parent commits. Of course, both parent commits are the commits we want to merge.

Fast-forward merge: it happens when git detects it can do a merge withuot creating new objects, because there are merge commits that contain the changes we want to merge.

Three rules in git:

1. The current branch tracks new commits.
2. When you move to another commit, Git updates your working directory.
3. Unreachable objects are **garbage collected** (commits done in detached HEAD). Git checks for commits that are no longer accessible, it means, no other element points to its hashcode (branch or tag)

## Rebasing made simple

`git rebase <branch-to-rebase>`:

1. Rebase looks for the first commit in <brach-to-rebase> that is also referenced in our current branch, all previous commits are shared in both branches. It means, it looks for the base commit for both branches
2. Git detach the current branch from the common commit and attach to the top commit of <branch-to-rebase> branch.
3. Now our current branch contains all the commits done while working on it PLUS the commits done in the <branch-to-rebase> branch.

But this is not 100% true. **Objects in Git are unmutable**, so if we detach a branch and attach to another commit, the content of the branch changes so its hashcode had to change as well, and this is not possible in Git because our current branch still points to the same commit.

So what Rebase really does is:  

1. Copy commits from current branch till the common commit of both branches and modify its parents, this way, the first commit of the current branch will be copied but the parent will be the last commit of the <branch-to-rebase> branch. This means new hashcode = **NEW COMMIT**.
2. When all new commits are created, Git moves the branch to the copy of the las commit.
3. Git leaves the original commits in its state with no branch referencing them.

**¡REBASE CREATES NEW COMMITS!**

Here comes the **garbage collector**. Now, original commits are almost impossible to reach again. Git takes some time to check unreachable objects like commits and blobs and delete them. 

Why do we have 2 methods to get the content together?

**merge**:

- pro :) preserves history, and this is very important, because git history can be seen graphically, and it will tell us the real story of the project.
- con :( merge can result less simple when working with big projects with lots of branches.

**rebase**:

- pro :) project looks simpler, like a single time line
- pro :) refactor history making it looking better.
- con :( history is not real
- con :( can cause some issues when working distributedly, we'll see later

If we have doubts between merge and rebase, **use merge**.

#### Tags, part 2

Tags are saved at *.git/refs/tags/* folder.

- **non-annotated** (regular) tags: `git tag <tagName>`: if we open any, we could check that a tag is a reference to an object, like branches. If we move this file to the *.git/refs/heads/*, we transform the tag into a branch. I mean, this command creates a tag that is a file which content is the hash of a commit.
- **annotated** tags: `git tag -a <tagName>`: similar but the file contains the hashCode of a tag object, and that object is a reference to a commit. It creates a git object.
    
What is the difference between branches and tags? A tag is like a branch that does not move. So if we make new commits, the branch will point to the latest commit but the tag will remain pointing to its original commit.
		
## Distribute version control

When we use "clone" command, Git adds a few lines to our configuration file for the repository *.git/config*.
Each Git repo can remember info about other copies which we call **remote**.  Git defines a default remote repo **origin**, and also a branch called **master** that maps over the master branch of the remote.

- `git branch --all` shows all branches

*.git/refs/remotes/origin/* contains remote branches and also the ***HEAD*** file that says where is HEAD pointing at. But sometimes, as an improvement feature of Git, only the HEAD file exists and packs all remote branches inside *.git/packed-refs* file. This can happen in both the local and remote branches.

- `git show-ref <branchName>` shows the commit which the branch is pointing at. If we write "master", Git will printout al branches that contains "master" (local and remote).

#### Synchronizing repositories

When we clone, git adds a *remote*. That's a reference to the remote repo. It just clone the default branch. Other branches are kept in `.git/packed_refs`, just in case you need to pull them. Local branches live at `.git/refs/heads`, remote ones live at `.git/refs/remotes/<remote-name>`.

When we clone, we copy the objects from the origin repository to our local. But this is a bit tricky.
So if we make a new commit in local:

```
$ git log -1

commit 066deb4f117a22bbe9ec4ddba1669b4fabe831d5  
Author: autorName <autor@email.com> 
Date:   Tue Apr 4 10:42:11 2017 +0200 
<new commit message>

$ git show-ref master

066deb4f117a22bbe9ec4ddba1669b4fabe831d5 refs/heads/master 
e42d61410e311f9b2cbfdad40a7323e936a68d76 refs/remotes/origin/master 
```

We can see that our local branch "master" is pointing to our last commit, but "origin/master" branch is not. `git push` to update remote repo and then: 

```
$ git show-ref master

066deb4f117a22bbe9ec4ddba1669b4fabe831d5 refs/heads/master 
066deb4f117a22bbe9ec4ddba1669b4fabe831d5 refs/remotes/origin/master
```

Both branches points now to our last commit. `git push` sends new objects (new commits, branches,...) to the remote repo, and updates branches to reflect local branches.

What if we have some local commits and remote branch has different history with new commits? If we push, there will be a conflict. There are 2 options to solve this:

1. **not recommended**: `git push -f`: **-f** means force! We force Git to copy our history to the remote, so remote branch will point to our last commit and all other commits that remote had, will be removed by the garbage collector. This may cause issues to our coworkers.
2. **recommended**: solve conflicts in our local repository before `git push`.
    1. `git fetch` will copy in our local repo the remote new history and changes the `origin/master` (our local) pointer to the same as remote, so now our local `master` branch will point to a different commit than `origin/master`.
    2. `git merge origin/master` create new object which our local will point to. It merges the branch `origin/master` (the same as the remote `master` branch) into our local `master` branch.
    3. `git push` copy our history without conflicts to remote repo and sets te remote branch to the same object as our local object created before with `git merge origin/master`.

As this procedure is so common (`git fetch` + `git merge origin/master`), there exist a command that makes both: `git pull`

So now we can do just `git pull` + `git push`

Never rebase shared objects, objects that have been pushed to a repo shared with others.

[autor profile link]: https://app.pluralsight.com/profile/author/paolo-perrotta "PluralSight profile"
[course link]: https://app.pluralsight.com/library/courses/how-git-works/table-of-contents "How git works"
[SHA1]: https://en.wikipedia.org/wiki/SHA-1 "SHA1 algorithm"
