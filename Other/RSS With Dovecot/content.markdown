RSS With Dovecot
================


0. About this
-------------

This tutorial is licensed under [Creative Commnons Attribution-ShareAlike][CC-BY-SA].


1. What are we going to do tomorrow night?
------------------------------------------

Tonight I will describe the approach I used to to have may RSS feeds being
delivered into a Dovecot mailbox. Someday I realized that I'd like to start
to subscribe to various RSS feeds, but the solution would need to fulfill
the following requirements:

 1. Needs to be either centralized or synchronized over multiple machines
 2. Easy to add new feeds on any machine
 3. If synchronized, it needs to do that automatically
 4. Ideally in an application that runs in the background and notifies me
    of new entries

These requirements ruled out the following possibilities:

 * Any online solution (I'd need to keep an extra tab or window open)
 * Any "normal" RSS application (no easy synchronization)
 * Delivery of RSS feeds to an e-mail box

But the thought of having an RSS feed delivered to my mailbox sounded quite
good for me. There also is [feed2imap][feed2imap] which allows to deliver RSS
feeds to an IMAP mailbox, given that I'm running my own Dovecot server, that
seemed like a very good solution. But that would require that I edit
a configuration file on the server for every feed that I would like to add.
That is a dealbreaker, I'm not going to log into a server, switch users and
then edit a file.

So instead I derived a system that works the following:

 * feed2imap delivers feeds to a Dovecot
 * The mailbox the feeds are delivered to is completely new/separate
 * The configuration file of feed2imap is created on every run from an e-mail
   in the drafts directory of the mailbox


2. Prerequisites
----------------

You'll need a server with Dovecot and a new mailbox. I'd advice against using
an already existing one, because that might lead to problems when it comes
to processing the e-mail in the drafts directory. You'll also need [feed2imap][feed2imap]
and bash. Why bash? Because I did not test it in another shell.


3. Building the feed2imap configuration
---------------------------------------

feed2imap does need a configuration file that looks roughly like this:

	feeds:
	  - name: feed name
	  - url: https://news.feeds.org/rss/
	  - target: imaps://feeds:PASSWORD@server/INBOX.feed%20name
	
	  - name: Another feed
	  - url: https://blog.somewhere.com/user/rss/
	  - target: imaps://feeds:PASSWORDS@server/INBOX.Another%20Feed

That is a rather easy format, we can easily create this from bash script.
So let us cut directly to the interesting part, the script to generate
the configuration:

	#!/usr/bin/env bash
	
	# The configuration file of feed2imap.
	config="./config"
	
	# The directory that contains the e-mails from which we create the config.
	source="$HOME/.mail/local/.Drafts/cur"
	
	# The e-mail directory that contains the feeds.
	maildir="$HOME/.mail/local/.INBOX."
	
	# The target IMAP server.
	target="imap://feeds:PASSWORD@server/INBOX."
	
	
	# Clear the configuration file and add the first line.
	echo "feeds:" > $config
	
	# With every edit the e-mail in the drafts directory will change its name,
	# and because we won't use the mailbox for anything else but feeds, we can
	# simplye use the wildcard here.
	# tac is used so that we go backwards through the e-mail, breaking at the
	# first empty line, which should be separator between the e-mail text and
	# the headers.
	tac $source/* | while read line; do
		if [ -z "$line" ]; then
			# The first empty line should be the separator between the text and
			# the headers, so we can stop processing now.
			exit 0
		fi
		
		# Get the name of the feed.
		name=$(echo "$line" | sed "s/ <.*//g")
		
		# Turn it into a directory name that can be used in the URL.
		directory=$(echo "$name" | sed "s/ /%20/g")
		
		# Get the URL of the feed.
		url=$(echo "$line" | sed --regexp-extended --silent "s/.*<(.*?)>.*/\1/p")
		
		# Create the target directory in the mailbox if it does not exist.
		if [ ! -d "$maildir$name" ]; then
			mkdir "$maildir$name"
		fi
		
		# Add the feed to the configuration file.
		echo "  - name: $name" >> $config
		echo "    url: $url" >> $config
		echo "    target: $target$directory" >> $config
		echo "" >> $config
	done

All you now need is an e-mail in the drafts directory which looks like this:

	feed name https://news.feeds.org/rss/
	Another feed https://blog.somewhere.com/user/rss/
	

Important here is the linefeed at the end of the file, otherwise it can't be
correctly read by the bash script.


4. Run it!
----------

Running it is easy, add a cron job that runs the script to build
the configuration and then run feed2imap itself.


5. Usage
--------

Add the IMAP account to you favorite e-mail account. Whenever you want to add
a RSS feed, you edit the e-mail in the drafts directory and add the feed.
Then you only need to wait until the script has run and rescan your IMAP
account for the new directory. Done.


6. Debugging
------------

If something goes, debugging is as easy as running the script and checking how
the generated configuration looks.


 [CC-BY-SA]: http://creativecommons.org/licenses/by-sa/4.0/
 [feed2imap]: http://home.gna.org/feed2imap/

