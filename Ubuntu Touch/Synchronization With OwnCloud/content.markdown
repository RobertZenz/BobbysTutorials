Ubuntu Touch - Synchronization With OwnCloud
============================================


0. About this
-------------

This tutorial is licensed under [Creative Commons Attribution-ShareAlike][CC-BY-SA].

This tutorial was tested on a BQ Aquaris 4.5. SSH access or a terminal will be
needed on the Ubuntu Touch device.


1. syncevolution
----------------

We will be using `syncevolution` for syncing the address books and calendars from
the [OwnCloud][owncloud] server to the Ubuntu Touch address books and calendars.


2. Create the connection
------------------------

First, we need to create a connection to our OwnCloud server.

    syncevolution --configure \
        --template WebDAV \
        username=USERNAME \
        password=PASSWORD \
        syncURL=https://yourowncloud/cloud/remote.php/ \
        keyring=no \
        target-config@owncloud

Note that the last parameter, `target-config@owncloud`, the `owncloud` part can
be replaced with whatever name you want to give the connection. Also `USERNAME`,
`PASSWORD` and the address obviously need to be replaced with your arguments.

Now we will setup the main template.

    syncevolution --configure \
        --template SyncEvolution_Client \
        sync=none \
        syncURL=local://@owncloud \
        username= \
        password= \
        owncloud

Note that here is now information that you need to replace.


3. Default address book
---------------------

Now we will add a remote address book to synchronize with the default address book
of the device. First we tell `syncevolution` where to find the remote
address book.

    syncevolution --configure \
        database=https://yourowncloud/cloud/remote.php/carddav/addressbooks/USERNAME/ADDRESSBOOK_NAME/ \
        backend=carddav \
        target-config@owncloud \
        REMOTE_ADDRESSBOOK_NAME

The URL must be replaced again, note the `USERNAME` and `address book_NAME` parts
in the url that also need to be replaced. The last parameter is the local name
of the address book, which is used by `syncevolution`, you can choose any name.

Now we will link it up against the default address book.

    syncevolution --configure \
        sync=two-way \
        backend=addressbook \
        database= \
        owncloud \
        REMOTE_ADDRESSBOOK_NAME

The last parameter is the name you have given it above. Also note the empty
`database` parameter, empty means the default databook.

The first time we need to do a `slow` synchronization.

    syncevolution --sync slow owncloud REMOTE_ADDRESSBOOK_NAME

After the first synchronization we cann do a "normal" synchronization.

    syncevolution owncloud REMOTE_ADDRESSBOOK_NAME


4. Multiple address books
------------------------

You can of course add multiple address books to synchronize into the default
address book, but that yields multiple problems (one of them being that all your
contacts will end in one address book).

If you want to have multiple address books, and also synchronize them separate,
you'll have to create multiple address books on the device. Currently there is
no GUI for doing that, but we can use `syncevolution` for it.

    syncevolution --create-database \
        backend=evolution-contacts \
        database=LOCAL_ADDRESSBOOK_NAME

This creates a new addressbook on your device.

Now the steps to wire up multiple address books are the same as in the part
above, except that in the second command, the database part must be filled in.

    syncevolution --configure \
        sync=two-way \
        backend=addressbook \
        database=LOCAL_ADDRESSBOOK_NAME \
        owncloud \
        REMOTE_ADDRESSBOOK_NAME


5. Calendars
------------

Now we can also synchronize calendars. The commands are basically the same, with
only the parameters set for calendars.

    syncevolution --configure \
        database=https://yourowncloud/cloud/remote.php/carddav/calendars/USERNAME/CALENDAR_NAME/ \
        backend=caldav \
        target-config@owncloud \
        REMOTE_CALENDAR_NAME

    syncevolution --configure \
        sync=two-way \
        backend=calendar \
        database= \
        owncloud \
        REMOTE_CALENDAR_NAME

The first sync is the same as with the contacts.

If you want to have multiple calendars, you again need to create a local one.

    syncevolution --create-database \
        backend=evolution-calendar \
        database=LOCAL_CALENDAR_NAME

And again, the steps for multiple calendars are the same as those for contacts.



 [owncloud]: https://owncloud.org/
 
