Mercurial HG HOWTO guide
************************

by Kailas Patil


In this tutorial I will cover the basic commands you will need to use mercurial.
hg help is your first friend and Mercurial Wiki is your second.

Help for Command
================

``$ hg help <command>``

or

``$ hg <command> - -help``


Commands to Create, Clone Repository
====================================

To make a new repository:

``$ hg init <path>``

To copy a repository from an existing repository:

``$ hg clone <sourcePath>  [<DestinationPath>]``

To clone specific branch of the repository:

``$ hg clone -r <barnchName> <sourcePath> [<destinationPath>]``

To copy existing repository to a new locaiton:

``$ hg clone . <newPath>``

To get changes from server repository and update working set:

``$ hg pull -u``

To get changes for specific branch from server repository:

``$ hg pull -r <branchName>``

To see what changes will come in on a PULL command:

``$ hg incoming``

To publish changes to specific branch on server repository:

``$ hg push -r <branchName>``

To see what changes will go out on a PUSH command:

``$ hg outgoing``


Commands for Add, Remove, Rename, Copy Operation
================================================

Add Specific file to repository:

``$ hg add <filename1, filename2, ...>``

To remove file from repository but don't delete from file system:

``$hg remove <filename1, filename2...>``

To remove file from repository and delete from file system as well"

``$ hg remove -f <filename1, filename2,...>``

To add all new files and remove all deleted files from repository:

``$ hg addremove``

To move or rename files in the repository:

``$ hg move <oldfilename> <newfilename>``

To copy files in the repository:

``$ hg copy <oldfilename> <newfilename>``


Commands for Commit, Revert Changes
===================================

To commit Changes to server repository:

``$ hg commit``

``$ hg push``

To commit as a particular user:

``$ hg commit -u <username>``

To revert all changes in local repository:

``$ hg revert -a``

To revert specific changes in local repositroy

``$ hg revert <filename1, filename2, ..>``

Commands to View Changes
To view changes between working set on your local repository and repository tip:

``$ hg diff``

To view changes between working set on your local repository and specific revision:

``$ hg diff -r <revisionNumber>``

To view changes between two revisions:

``$ hg diff -r <revisionNumber> -r <revisionNumber>``

To check what are changes in working set:

``$ hg status``

To list all changesets:

``$ hg log``

Commands to Update Working Set
==============================

To change working set to tip:

``$ hg pull``

``$ hg up``

To change working set with discarding any current work:

``$ hg update -C``

To change working set to specific revision:

``$ hg update -r <revisionNumber>``

To change working set to specific branch:

``$ hg update -r <branchName>``

To see the list of branches available for merging:

``$ hg heads``


Commands for Handling tags and Branches
=======================================

To delete a tag:

``$ hg tag -r <tagtext>``

To tag a revision:

``$ hg tag [-r <revisionNumber] <tagtext>``

To list tags:

``$ hg tags``

To create new branch:

``$ hg branch <branchName>``
``$ hg commit -m "New Branch created <branchName>"``


To delete a branch:

``$ hg commit - - close-branch <branchName>``

To see the list of branches available:

``$ hg branches``

For HG Diff command setting in .hgrc file in /home/username folder::

    [diff]
    git=1
    showfunc=1
    unified=8


Commands related to Patch
=========================

Generating a patch:

``$ hg diff  >  patchfilename``

Discarding all local changes:

``$ hg revert -a``


How to get rid of older mercurial heads?
========================================

Occasionally you want to merge two heads, but you want to throw away all changes from one of the heads, a so-called dummy merge. The internal:local and internal:other merge tools look like they do that, but only work if both branches have changed the content of the file. If the 'other' branch changes the file and 'local' does not, a merge using the internal:local tool will include that change, and vice versa. File renames, attribute changes and files added also suffer from this problem.

So to safely merge X into the current revision without letting any of the changes from X come through, do::

    hg -y merge --tool=internal:fail XXXX:XXXXX
    hg revert --all --rev .
    hg resolve -a -m
    hg commit -m "Merge (and kill) old XXXX:XXXXX"
    
This will ensure that only changes from the current working copy parent revision are committed when you commit the merge.

Using internal:fail will fail the merge - this is useful if you want to prevent Mercurial from starting a merge tool after a merge with conflicts. The -y option causes any questions that may come up to be answered in the affirmative, which is harmless since any changes will be reverted in the next step.


How to remove changesets from a Mercurial repository?
=====================================================

To remove accidental commit but to keep modified files you need to "strip" the wrong revision. For example if last revisions are::

    $ hg log | less

    changeset:   381:114be9299ed6
    tag:         tip
    user:        Andrei Levin <andrei.levin@didotech.com>
    date:        Thu Mar 17 18:30:57 2016 +0100
    files:       broker/models/broker.py broker/views/broker_view.xml transfer_service/__init__.py transfer_service/__openerp__.py transfer_service/report/invoice_detail_report.xml
    description:
    [broker]: Added control preventing double submission of the same order,
    Colored purchase order lines


    changeset:   380:31a7240f351f
    user:        Andrei Levin <andrei.levin@didotech.com>
    date:        Thu Mar 17 16:09:21 2016 +0100
    files:       broker/__openerp__.py broker/models/broker.py broker/views/broker_view.xml broker/wizard/distribution_list.py
    description:
    [broker]: Added possibility to download Purchase Order

to remove last commit you should type::

    $ hg strip --keep 381

