In software development, Git /ɡɪt/ is a distributed revision control and source code management (SCM) system with an emphasis on speed.[4] Git was initially designed and developed by Linus Torvalds for Linux kernel development in 2005.
Every Git working directory is a full-fledged repository with complete history and full version tracking capabilities, not dependent on network access or a central server.
Git is free software distributed under the terms of the GNU General Public License version 2.
Contents  [hide] 
1 History
2 Design
2.1 Characteristics
2.2 Data structures
3 Implementations
4 Adoption
5 See also
6 References
7 Further reading
8 External links
History[edit]

Git development began after many developers of the Linux kernel chose to give up access to BitKeeper, a proprietary SCM system that had previously been used to maintain the project.[5] The copyright holder of BitKeeper, Larry McVoy, had withdrawn free use of the product after he claimed that Andrew Tridgell had reverse-engineered the BitKeeper protocols.
Torvalds wanted a distributed system that he could use like BitKeeper, but none of the available free systems met his needs, particularly in terms of performance. Torvalds took an example of an SCM system requiring thirty seconds to apply a patch and update all associated metadata, and noted that this would not scale to the needs of Linux kernel development, where syncing with fellow maintainers could require 250 such actions at a time. His goal was for patches to take three seconds.[6] Torvalds had several other design criteria:
Take Concurrent Versions System (CVS) as an example of what not to do; if in doubt, make the exact opposite decision.
Support a distributed, BitKeeper-like workflow.
Very strong safeguards against corruption, either accidental or malicious.[7][8]
These three criteria eliminated every then-existing version control system, except for Monotone. Considering performance as well excluded this too.[7] So, immediately after the 2.6.12-rc2 Linux kernel development release,[7] he set out to write his own.[7]
Torvalds has quipped about the name git, which is British English slang roughly equivalent to "unpleasant person". Torvalds said: "I'm an egotistical bastard, and I name all my projects after myself. First 'Linux', now 'git'."[9][10] The man page describes git as "the stupid content tracker".[11]
The development of Git began on 3 April 2005.[12] The project was announced on 6 April,[13] and became self-hosting as of 7 April.[12] The first merge of multiple branches was done on 18 April.[14] Torvalds achieved his performance goals; on 29 April, the nascent Git was benchmarked recording patches to the Linux kernel tree at the rate of 6.7 per second.[15] On 16 June, the kernel 2.6.12 release was managed by Git.[16] Torvalds turned over maintenance on 26 July 2005 to Junio Hamano, a major contributor to the project.[17] Hamano was responsible for the 1.0 release on 21 December 2005, and remains the project's maintainer.[18]
Design[edit]

Git's design was inspired by BitKeeper and Monotone.[19][20] Git was originally designed as a low-level version control system engine on top of which others could write front ends, such as Cogito or StGIT.[20] The core Git project has since become a complete version control system that is usable directly.[21] While strongly influenced by BitKeeper, Torvalds deliberately attempted to avoid conventional approaches, leading to a unique design.[22]
Characteristics[edit]
Git's design is a synthesis of Torvalds's experience with Linux in maintaining a large distributed development project, along with his intimate knowledge of file system performance gained from the same project and the urgent need to produce a working system in short order. These influences led to the following implementation choices:
Strong support for non-linear development
Git supports rapid branching and merging, and includes specific tools for visualizing and navigating a non-linear development history. A core assumption in Git is that a change will be merged more often than it is written, as it is passed around various reviewers. Branches in git are very lightweight: A branch in git is only a reference to a single commit. With its parental commits, the full branch structure can be constructed.
Distributed development
Like Darcs, BitKeeper, Mercurial, SVK, Bazaar and Monotone, Git gives each developer a local copy of the entire development history, and changes are copied from one such repository to another. These changes are imported as additional development branches, and can be merged in the same way as a locally developed branch.
Compatibility with existing systems/protocols
Repositories can be published via HTTP, FTP, rsync, or a Git protocol over either a plain socket, or ssh. Git also has a CVS server emulation, which enables the use of existing CVS clients and IDE plugins to access Git repositories. Subversion and svk repositories can be used directly with git-svn.
Efficient handling of large projects
Torvalds has described Git as being very fast and scalable,[23] and performance tests done by Mozilla showed it was an order of magnitude faster than some version-control systems, and fetching version history from a locally stored repository can be one hundred times faster than fetching it from the remote server.[24]
Cryptographic authentication of history
The Git history is stored in such a way that the ID of a particular version (a commit in Git terms) depends upon the complete development history leading up to that commit. Once it is published, it is not possible to change the old versions without it being noticed. The structure is similar to a Merkle tree, but with additional data at the nodes as well as the leaves.[25] (Mercurial and Monotone also have this property.)
Toolkit-based design
Git was designed as a set of programs written in C, and a number of shell scripts that provide wrappers around those programs.[26] Although most of those scripts have since been rewritten in C for speed and portability, the design remains, and it is easy to chain the components together.[27]
Pluggable merge strategies
As part of its toolkit design, Git has a well-defined model of an incomplete merge, and it has multiple algorithms for completing it, culminating in telling the user that it is unable to complete the merge automatically and that manual editing is required.
Garbage accumulates unless collected
Aborting operations or backing out changes will leave useless dangling objects in the database. These are generally a small fraction of the continuously growing history of wanted objects. Git will automatically perform garbage collection when enough loose objects have been created in the repository. Garbage collection can be called explicitly using git gc --prune.[28]
Periodic explicit object packing
Git stores each newly created object as a separate file. Although individually compressed, this takes a great deal of space and is inefficient. This is solved by the use of packs that store a large number of objects in a single file (or network byte stream) called packfile, delta-compressed among themselves. Packs are compressed using the heuristic that files with the same name are probably similar, but do not depend on it for correctness. A corresponding index file is created for each packfile, telling the offset of each object in the packfile. Newly created objects (newly added history) are still stored singly, and periodic repacking is required to maintain space efficiency. The process of packing the repository can be very computationally expensive. By allowing objects to exist in the repository in a loose, but quickly generated format, git allows the expensive pack operation to be deferred until later when time does not matter (e.g. the end of the work day). Git does periodic repacking automatically but manual repacking is also possible with the git gc command. For data integrity, both packfile and its index have SHA-1 checksum inside, and also the file name of packfile contains a SHA-1 checksum. To check integrity, run the git fsck command.
Another property of Git is that it snapshots directory trees of files. The earliest systems for tracking versions of source code, SCCS and RCS, worked on individual files and emphasized the space savings to be gained from interleaved deltas (SCCS) or delta encoding (RCS) the (mostly similar) versions. Later revision control systems maintained this notion of a file having an identity across multiple revisions of a project. However, Torvalds rejected this concept.[29] Consequently, Git does not explicitly record file revision relationships at any level below the source code tree.
Implicit revision relationships have some significant consequences:
It is slightly more expensive to examine the change history of a single file than the whole project.[30] To obtain a history of changes affecting a given file, Git must walk the global history and then determine whether each change modified that file. This method of examining history does, however, let Git produce with equal efficiency a single history showing the changes to an arbitrary set of files. For example, a subdirectory of the source tree plus an associated global header file is a very common case.
Renames are handled implicitly rather than explicitly. A common complaint with CVS is that it uses the name of a file to identify its revision history, so moving or renaming a file is not possible without either interrupting its history, or renaming the history and thereby making the history inaccurate. Most post-CVS revision control systems solve this by giving a file a unique long-lived name (a sort of inode number) that survives renaming. Git does not record such an identifier, and this is claimed as an advantage.[31][32] Source code files are sometimes split or merged as well as simply renamed,[33] and recording this as a simple rename would freeze an inaccurate description of what happened in the (immutable) history. Git addresses the issue by detecting renames while browsing the history of snapshots rather than recording it when making the snapshot.[34] (Briefly, given a file in revision N, a file of the same name in revision N−1 is its default ancestor. However, when there is no like-named file in revision N−1, Git searches for a file that existed only in revision N−1 and is very similar to the new file.) However, it does require more CPU-intensive work every time history is reviewed, and a number of options to adjust the heuristics. This mechanism does not always work; sometimes a file that is renamed with changes in the same commit is read as a deletion of the old file and the creation of a new file. Developers can work around this limitation by committing the rename and changes separately.
Git implements several merging strategies; a non-default can be selected at merge time:[35]
resolve: the traditional three-way merge algorithm.
recursive: This is the default when pulling or merging one branch, and is a variant of the three-way merge algorithm.
When there are more than one common ancestors that can be used for three-way merge, it creates a merged tree of the common ancestors and uses that as the reference tree for the three-way merge. This has been reported to result in fewer merge conflicts without causing mis-merges by tests done on actual merge commits taken from Linux 2.6 kernel development history. Additionally this can detect and handle merges involving renames.
—Linus Torvalds[36]
octopus: This is the default when merging more than two heads.
Data structures[edit]
Git's primitives are not inherently a source code management (SCM) system. Torvalds explains,[37]
In many ways you can just see git as a filesystem — it is content-addressable, and it has a notion of versioning, but I really really designed it coming at the problem from the viewpoint of a filesystem person (hey, kernels is what I do), and I actually have absolutely zero interest in creating a traditional SCM system.
From this initial design approach, Git has developed the full set of features expected of a traditional SCM,[21] with features mostly being created as needed, then refined and extended over time.


Some data flows and storage levels in the Git revision control system.
Git has two data structures: a mutable index (also called stage or cache) that caches information about the working directory and the next revision to be committed; and an immutable, append-only object database.
The object database contains four types of objects:
A blob (binary large object) is the content of a file. Blobs have no file name, time stamps, or other metadata.
A tree object is the equivalent of a directory. It contains a list of file names, each with some type bits and the name of a blob or tree object that is that file, symbolic link, or directory's contents. This object describes a snapshot of the source tree.
A commit object links tree objects together into a history. It contains the name of a tree object (of the top-level source directory), a time stamp, a log message, and the names of zero or more parent commit objects.
A tag object is a container that contains reference to another object and can hold additional meta-data related to another object. Most commonly, it is used to store a digital signature of a commit object corresponding to a particular release of the data being tracked by Git.
The index serves as connection point between the object database and the working tree.
Each object is identified by a SHA-1 hash of its contents. Git computes the hash, and uses this value for the object's name. The object is put into a directory matching the first two characters of its hash. The rest of the hash is used as the file name for that object.
Git stores each revision of a file as a unique blob. The relationships between the blobs can be found through examining the tree and commit objects. Newly added objects are stored in their entirety using zlib compression. This can consume a large amount of disk space quickly, so objects can be combined into packs, which use delta compression to save space, storing blobs as their changes relative to other blobs.
Git servers typically listen on TCP port 9418.[38]
Implementations[edit]



gitg is a graphical front-end using GTK+
Git is primarily developed on Linux, although it also supports most major operating systems including BSD, Solaris, OS X, and Microsoft Windows.[39]
The JGit implementation of Git is a pure Java software library, designed to be embedded in any Java application. JGit is used in the Gerrit code review tool and in EGit, a Git client for the Eclipse IDE.[40]
The Dulwich implementation of Git is a pure Python software component for Python 2.[41]
The libgit2 implementation of Git is an ANSI C software library with no other dependencies, which can be built on multiple platforms including Microsoft Windows, Linux, Mac OS X, and BSD.[42] It has bindings for many programming languages, including Ruby, Python and Haskell.[43][44][45] It is used as the underlying Git implementation in Microsoft's Team Foundation Service and Visual Studio.[46]
The Plastic SCM versioning system contains its own implementation of the Git protocol, to allow Plastic SCM clients to interoperate with remote Git repositories.[47]
The Perforce version management system also supports Git clients. Perforce can make its versioned content available as Git repositories using the git+ssh and smart HTTP transports, and these repositories can be used as Git remotes from any Git client.[48]
JS-Git is a JavaScript implementation of a subset of Git.[49]
Adoption[edit]

The Eclipse Foundation reported in its annual community survey that as of May 2013, more than 36% of professional software developers use Git as their primary source control system,[50] compared with 27.6% in 2012 and 12.8% in 2011.[51] Open source directory Ohloh reports a similar uptake among open source projects.[52]
The UK IT jobs website itjobswatch.co.uk reports that as of February 2014, approximately 16.6% of UK permanent software development job openings list Git as a requirement,[53] compared to 16.9% for Subversion,[54] 11.4% for Microsoft Team Foundation Server,[55] 1.0% for Visual SourceSafe,[56] and 1.95% for Mercurial.[57]
See also[edit]

Portal icon	Free software portal
Comparison of revision control software
Comparison of open source software hosting facilities
List of revision control software
Repo (script)
References[edit]

Jump up ^ Hamano, Junio (2014-02-14). "[ANNOUNCE] Git v1.9.0". git mailing list. Retrieved 2014-02-15.
Jump up ^ Hamano, Junio (2013-11-20). "[ANNOUNCE] Git v1.8.5-rc3". git mailing list. Retrieved 2013-12-02.
Jump up ^ "git/git.git/tree". git.kernel.org. Retrieved 2009-06-15.
Jump up ^ Torvalds, Linus (2005-04-07). "Re: Kernel SCM saga..". linux-kernel mailing list. "So I'm writing some scripts to try to track things a whole lot faster."
Jump up ^ Feature: No More Free BitKeeper | KernelTrap.org[dead link]
Jump up ^ Torvalds, Linus (2005-04-07). "Re: Kernel SCM saga..". linux-kernel mailing list.
^ Jump up to: a b c d Linus Torvalds (2007-05-03). Google tech talk: Linus Torvalds on git. Event occurs at 02:30. Retrieved 2007-05-16.
Jump up ^ Torvalds, Linus (2007-06-10). "Re: fatal: serious inflate inconsistency". git mailing list. A brief description of Git's data integrity design goals.
Jump up ^ "GitFaq: Why the 'git' name?". Git.or.cz. Retrieved 2012-07-14.
Jump up ^ "After controversy, Torvalds begins work on 'git'". PC World. 2012-07-14. "Torvalds seemed aware that his decision to drop BitKeeper would also be controversial. When asked why he called the new software, "git", British slang meaning "a rotten person", he said. "I'm an egotistical bastard, so I name all my projects after myself. First Linux, now git""
Jump up ^ "git(1) Manual Page". Retrieved 2012-07-21.
^ Jump up to: a b Torvalds, Linus (2007-02-27). "Re: Trivia: When did git self-host?". git mailing list.
Jump up ^ Torvalds, Linus (2005-04-06). "Kernel SCM saga..". linux-kernel mailing list.
Jump up ^ Torvalds, Linus (2005-04-17). "First ever real kernel git merge!". git mailing list.
Jump up ^ Mackall, Matt (2005-04-29). "Mercurial 0.4b vs git patchbomb benchmark". git mailing list.
Jump up ^ Torvalds, Linus (2005-06-17). "Linux 2.6.12". git-commits-head mailing list.
Jump up ^ Torvalds, Linus (2005-07-27). "Meet the new maintainer...". git mailing list.
Jump up ^ Hamano, Junio C. (2005-12-21). "Announce: Git 1.0.0". git mailing list.
Jump up ^ Torvalds, Linus (2006-05-05). "Re: [ANNOUNCE] Git wiki". linux-kernel mailing list. "Some historical background" on git's predecessors
^ Jump up to: a b Torvalds, Linus (2005-04-08). "Re: Kernel SCM saga". linux-kernel mailing list. Retrieved 2008-02-20.
^ Jump up to: a b Torvalds, Linus (2006-03-23). "Re: Errors GITtifying GCC and Binutils". git mailing list.
Jump up ^ Torvalds, Linus (2006-10-20). "Re: VCS comparison table". git mailing list. A discussion of Git vs. BitKeeper
Jump up ^ Torvalds, Linus (2006-10-19). "Re: VCS comparison table". git mailing list.
Jump up ^ Dreier, Roland (2006-11-13). "Oh what a relief it is"., observing that "git log" is 100x faster than "svn log" because the latter has to contact a remote server.
Jump up ^ "Trust". Git Concepts. Git User's Manual. 2006-10-18.
Jump up ^ Torvalds, Linus. "Re: VCS comparison table". git mailing list. Retrieved 2009-04-10., describing Git's script-oriented design
Jump up ^ iabervon (2005-12-22). "Git rocks!"., praising Git's scriptability
Jump up ^ "Git User's Manual". 2007-08-05.
Jump up ^ Torvalds, Linus (2005-04-10). "Re: more git updates..". linux-kernel mailing list.
Jump up ^ Haible, Bruno (2007-02-11). "how to speed up "git log"?". git mailing list.
Jump up ^ Torvalds, Linus (2006-03-01). "Re: impure renames / history tracking". git mailing list.
Jump up ^ Hamano, Junio C. (2006-03-24). "Re: Errors GITtifying GCC and Binutils". git mailing list.
Jump up ^ Hamano, Junio C. (2006-03-23). "Re: Errors GITtifying GCC and Binutils". git mailing list.
Jump up ^ Torvalds, Linus (2006-11-28). "Re: git and bzr". git mailing list., on using git-blame to show code moved between source files
Jump up ^ Torvalds, Linus (2007-07-18). "git-merge(1)".
Jump up ^ Torvalds, Linus (2007-07-18). "CrissCrossMerge".
Jump up
