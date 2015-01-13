******************************************************
mysqlfuse - Access a MySQL database like a file system
******************************************************

This is an attempt to use FUSE to provide a view of a MySQL database as a
filesystem.  See the __doc__ string in the mysqlfuse.py file for an
explanation.

Currently it MOSTLY works.  But not everything really works quite right.
This is v.0.00001alpha, mind you.


Concept
=======
The root directory contains subdirs named for each table in the db.

Within each table, we look up the PRIMARY indexes (those have to be unique,
right?).  Ummm... for the first cut, we'll take them in order given by
Sequence (ideally we should be able to take them in all orders, see below).
This will be better with an example::

    CREATE TABLE `testtab` (
        `key1` varchar(10) default NULL,
        `key2` varchar(7) default NULL,
        `data1` text default NULL,
        `data2` varchar(18) default NULL,
        PRIMARY KEY (`key1`, `key2`)
    )

This branch of mysqlfuse is going to have the directories alternating
between keyname and keyvalue.  This should allow browsing by any ordering
of keys.

(N.B. Handling NULL keys and different types of keys (int, varchar) is
going to get tricky--actually it hasn't.) The above should lead to this::


    .                      testtab
                          /       \
                         /         \
                     key1           key2     (all the key fields)
               _____/ |  \         / |  \
              /       |   \       /  |   \
             foo     ...  bar   baz ...  quux  (all the values for each field)
              |            |     |         |
              |            |     |         |
             key2         key2  key1      key1  (the key fields not above each)
           /  |  \       / | \ / |  \    / |  \
          /   |   \     |  | | | |   |  |  |   \
        baz  ...  quux baz ..  foo..bar foo...  bar  (values for those)
       /   \
     data1  data2  ...


At the bottom level, the filenames are ``data1`` and ``data2``, i.e. all the
*names* of the non-key fields, and each such file contains the *value* for
that field in that row (note that each directory just above the bottom
level contains a single row, assuming the keys have to be unique,
i.e. primary).  Higher-level entries are all directories, not files.  They
alternate between name-of-a-key-field and values-of-a-key-field.  For
example, ``/books/book/My_Friend_Flicka/page/71/pagecontents``.

This way, the data can be accessed by any possible permutation of keys,
since both ``/books/book/My_Friend_Flicka/page/71/`` and also
``/books/page/71/book/My_Friend_Flicka/`` are there.  This gets more fun when
there are more elements in the key.



Usage
=====
::

  $ mysqlfuse.py -o host=localhost,user=sqluser,passwd=password,db=databasename <mountpoint>

Debugging information can be found in the ``DBG`` file.


Dependencies
============
* fuse-python__
* MySQL-Python__

__ https://pypi.python.org/pypi/fuse-python
__ http://mysql-python.sourceforge.net/

On Debian, simply install ``python-fuse`` and ``python-mysql``::

  $ sudo apt-get install python-fuse python-mysql
