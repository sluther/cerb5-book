
## Backups ##

### Introduction ###

To fully protect your Cerb5 helpdesk data you need to backup both the MySQL database and the `/cerb5/storage/` filesystem.  While there are countless good approaches for performing backups, this document will focus on the best practices we've discovered over the past several years of hosting hundreds of helpdesk instances on our On-Demand network.  The examples will be Unix-based since that's what we're most comfortable with.

### Requirements ###

* A Unix-based server with shell access
* A Cerb5 installation

### Setting up the environment ###

#### Creating a backups user ####

For convenience and permissions, it's a good idea to make a `backups` user on the local system.  If at all possible, you should put the `backups` user home directory on a different hard disk than your live databases to provide for fault tolerance and better write performance.  The examples below will refer to this location as `~backups`.  A separate location is important -- while a [RAID](http://en.wikipedia.org/wiki/RAID) configuration will protect you from the failure of individual storage hardware devices, it won't protect you from filesystem corruption or non-hardware-related data loss (e.g. bugs, errant queries, maliciousness).

* You can usually accomplish this with something like:

		# adduser --home /backups --disabled-password --disabled-login backups

#### Creating a backups database user with a shadow password ####

It's a really smart idea to make a separate backup user that is read-only, especially when you start automating backups.  

* In MySQL you can do this with the following query (_make up your own password!_):

		mysql> GRANT SELECT, RELOAD, LOCK TABLES 
		ON *.* 
		TO backups@localhost 
		IDENTIFIED BY 's3cret';

* You should then create a [Shadow file](http://en.wikipedia.org/wiki/Shadow_password) which will "securely" store your password for automation:

	* Put the password text inside a hidden file.  It's better to use something like `vi` instead of `echo`, since you don't want to leave your password in your command history.  We'll use `echo` here for simplicity:
	
			# echo -n "s3cret" > ~backups/.db.shadow;
	
	* Make the backups user the owner:
	
			# chown backups:backups ~backups/.db.shadow;
	
	* Make the file read-only by the owner and invisible to world:
	
			# chmod 400 ~backups/.db.shadow;

In the examples below we'll use this shadow file in place of literally typing the password on the command line.  In addition to enabling automation, this also helps prevent sensitive information from being visible to other users in the global process list.

### Backing up the database ###

The database stores the majority of your helpdesk information. In the majority of cases, it stores anything that isn't an attachment.

#### Using mysqlhotcopy (recommended for MyISAM) ####

One of the quickest ways to backup (and restore) a MyISAM-based MySQL database is to use the `mysqlhotcopy` [^mysqlhotcopy] tool, which copies the raw .frm, .MYD, and .MYI files to a new location.  This utility will flush any pending row changes from memory to the disk and then lock the tables while copying them to a new location.  Unless your database is huge (relative to your hardware), or your disk or network I/O is very slow, you should be able to hotcopy a live database with minimal interruption.  If you're using replication you can make hotcopies of a slave without any service interruption.

[^mysqlhotcopy]: MySQL Documentation: _mysqlhotcopy_  
	<http://dev.mysql.com/doc/refman/5.0/en/mysqlhotcopy.html>

**Pros:**

* It's as fast as your disk or network I/O.
* Restoring a hotcopy on similar hardware is nearly instantaneous.  It won't need to rebuild the indexes.
* It's very flexible, allowing for table inclusion/exclusion by regexp patterns, truncating indexes, redirecting output over [SCP](http://en.wikipedia.org/wiki/Secure_copy), etc.

**Cons:**

* It's Unix only.
* It's only compatible with MyISAM tables (not InnoDB).
* Binary files are less corruption-tolerant than SQL dump text files.
* You may have issues restoring a hotcopy on a new machine with a significantly different architecture (e.g. 32-bit to 64-bit and [http://en.wikipedia.org/wiki/Endianness endianness]).

It's **important** to reiterate that `mysqlhotcopy` will not properly back up InnoDB tables, but it won't give you an error if you try.  If you're using InnoDB tables you must use `mysqldump` or an Inno-DB specific hotcopy solution.

**Usage: (to local filesystem)**

	$ mysqlhotcopy -u backups -p`cat ~backups/.db.shadow` --addtodest --noindices \
	c5_database ~backups/dbs/

**Usage: (to SCP)**

	$ mysqlhotcopy -u backups -p`cat ~backups/.db.shadow` --addtodest --noindices \
 	--method='scp -c arcfour -C -2' c5_database backups@remotehost:~backups/dbs/

#### Using mysqldump (recommended for InnoDB) ####

If you're using InnoDB tables, or you don't have access to the `mysqlhotcopy` tool or you aren't comfortable using it, then `mysqldump` is the standard tool for performing database backups.

**Pros:**

* It works for most database storage engine types (e.g. MyISAM, InnoDB).
* It's very corruption-tolerant since it writes text files that you can edit with any text editor.  In the event of file corruption it's easier to salvage your data.
* It's available in almost every environment.
* SQL dumps are widely compatible with different versions of MySQL on different architectures.

**Cons:**

* When you reimport a SQL dump file the database will have to recreate your key indexes which may take a significant amount of time.  You can use the `--disable-keys` flag to improve this.
* Writing a dump file is usually slower than using a tool like `mysqlhotcopy` to copy the binary data files.
* To ensure a consistent snapshot you may need to lock your tables with `--lock-tables`.  This prevents data from being written in a file you've already backed up while you're still backing up other files.  Without locking, some tables will have references to data in other tables that doesn't exist.  However, if you're using InnoDB you can use the `--single-transaction` flag to make a consistent snapshot without locks (but you should ensure the database schema itself doesn't change during the process; which usually only happens during a software update).

**Usage: (MyISAM)**

	$ mysqldump -Q --disable-keys --e -u backups -p`cat ~backups/.db.shadow` \
	--lock-tables c5_database > c5_database.sql

### Backing up the storage filesystem ###

The `/cerb5/storage` filesystem stores the pending mail parser queue, import queue, and all the file attachments from mail.  It's the only filesystem hierarchy you need to backup for a full recovery (the rest of the files are temporary caches, or can be downloaded from the project website). Since the bulk of the `storage` directory is comprised of tons of small file attachments that will never be modified (only deleted), it's the ideal candidate for incremental backups.

#### Using rsync (recommended) ####

[rsync](http://en.wikipedia.org/wiki/Rsync) is one of the simplest ways to copy only changed files to a new location.  In a nutshell, its purpose is to keep two copies of the same directory in-sync.

**Pros:**

* It's available on most Unix distributions.
* It's fast and flexible.
* You can use key-based authentication to automate remote copies.

**Cons:**

* It's Unix-only, though there are several clones and ports for Windows.

**Usage: (to local filesystem)**

	rsync -a --verbose --delete /path/to/cerb5/storage ~backups/storage
	
**Usage: (to SSH)**

	rsync -aze ssh --verbose --delete /path/to/cerb5/storage \
	backups@remotehost:~backups/storage

**Tips:**

* When dealing with directories in rsync, including a trailing slash (`storage/`) means to copy the directory's contents and not the directory.  Excluding the trailing slash (`storage`) will copy a directory *and* its contents.
* The `--delete` option will remove files from the destination directory that are no longer in the source directory.  Since this can be dangerous if you mistype a directory, you may omit this option until you're confident things work.

### Keeping off-site backups ###

It's crucial to assume that [anything that can go wrong will go wrong](http://en.wikipedia.org/wiki/Murphy%27s_law) (at some point).  You can't trust your local RAID, your server, or your datacenter, to store the only copy of data that your business is doomed without.

At the simplest, off-site backups may involve downloading a copy of your backups to your office and burning an extra copy to DVD.  Keep in mind, it does you no good to have 250GB of backups on your office network with a 256Kbps upstream to your datacenter.  If you could drive across the country to hand deliver the backups faster than you could upload them then you'll want to place your backups somewhere where you can retrieve them quickly.  However, there's something to be said for the secure feeling of having tangible, offline copy of your critical data; and even WebGroup Media's 9 years old Cerb5 database could fit on a DVD and be re-uploaded from a residential Internet connection.

If you have the resources, you may also choose to have a standby server in a different location than your production server.  This would allow you to make server-to-server backups, which would require two hardware configurations, or two datacenters, to fail at the same time before you follow them into the blackness of failure.

Our favorite choice, cloud and utility computing, also provides a great opportunity for off-site backups, since you can store massive amounts of data, highly redundantly, for a few dollars per month; and you'll likely be able to move data to and from a cloud computing network MUCH faster than using your office DSL.

#### Using Amazon EC2/S3 (recommended) ####

Amazon S3 is a storage service.  At the time of this writing, Amazon S3 costs 10 cents (USD $) per month per gigabyte stored.  For that price, your data is protected by being replicated to multiple locations.  You also get reasonably fast network access to it (generally 10-20 MB/sec for us at WGM in Southern California).  That's $1/mo per redundant 10GB!  Be sure to read the terms on their site, as there are also similar rates for bandwidth (though most of the time you'll just be uploading).

**Pros:**

* It's highly redundant and secure.
* It's inexpensive.
* Amazon is a very well known and respected online technology company.

**Cons:**

* Cloud and utility computing are in their infancy, and some growing pains are inevitable.
* It costs money (but it's worth every penny!)

**Usage:**

* [Sign up](http://www.amazonaws.com/) for Amazon Web Services (click 'Sign Up' on the right).
 
![](images/maintenance/backups_aws_signup.png)

* Once your registration is complete, click on 'AWS Access Identifiers' from the account menu.
 
![](images/maintenance/backups_aws_accessid.png)

* Make a note of your Access Key ID and Secret Access Key.  You'll need to plug these values into the various tools you use to interact with their services.

![](images/maintenance/backups_aws_access_secret.png)

##### Using Jets3t at the server command line #####

The [Jets3t](http://jets3t.s3.amazonaws.com/downloads.html) project provides a Synchronize tool that works much like rsync, replicating changes from a local directory structure to a remote S3 bucket.  It requires a Java Runtime Environment (JRE) of version 1.5 or later to be available.

**Usage:**

	~backups/jets3t/bin/synchronize.sh -k UP yourbucket/backups/server1/dbs/20110531 *.gz


**Tips:**

* The `-k` option above prevents files from being deleted from S3 if they no longer exist in the local filesystem.
* If you don't require SSL (i.e. your content is public) you can modify `jets3t/config/jets3t.properties` and set `s3service.https-only` to `false`.  This should give you a moderate speed boost on uploading.

##### Using S3Fox in Firefox3 #####

Our favorite tool for browsing S3 buckets is the [S3Fox](https://addons.mozilla.org/en-US/firefox/addon/3247) extension for [Firefox](http://www.mozilla.com/en-US/firefox/).  Once installed, you can simply drag files back and forth between your S3 account and your local machine.  It will also allow you to modify the access level (ACL) of any file, which gives you the option of creating a publicly sharable URL.  This is great, since S3 is useful for far more than just off-site backups -- you can host downloads or high-resolution screencasts without bogging down your servers (and paying a bandwidth ransom at the datacenter) if your content becomes popular.

![The S3Fox extension for Firefox](images/maintenance/backups_aws_s3fox.png)
