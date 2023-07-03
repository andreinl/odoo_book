Using Git
=========

Working with branches
---------------------

Create new branch::

    $ git checkout -b feature/order_data
    $ git branch
    * feature/order_data
      master


Pull changes from remote development branch to master::

    $ git checkout master
    Switched to branch 'master'
    $ git pull
    $ git pull origin feature/order_data
    $ git push


Merge master with local development branch::

    $ git checkout master
    $ git merge feature/order_data
    $ git push


Update development branch::

    $ git checkout feature/order_data
    $ git pull origin master

This is almost the same as::

    $ git checkout feature/order_data      # gets you "on branch feature/order_data"
    $ git fetch origin        # gets you up to date with origin
    $ git merge origin/master


Push development branch to remote server::

    $ git push -u origin feature/order_data

