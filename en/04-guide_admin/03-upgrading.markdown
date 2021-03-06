
## Upgrades ##

### Introduction ###

The officially supported way of upgrading Cerb5 is by using version control software like Git[^git] or Subversion[^subversion].  These tools will automatically update your helpdesk files to the latest version.  The major advantage of version control is that it will attempt to automatically merge official code changes with any customization you have done. It also gives you the ability to list all your local changes to any project files, and to revert to an official version when desirable.

[^git]: Wikipedia: _Git_  
	<http://en.wikipedia.org/wiki/Git_(software)>
[^subversion]: Wikipedia: _Subversion_  
	<http://en.wikipedia.org/wiki/Subversion_(software)>

We recommend that you use Git whenever possible because the official project is hosted using Git on [GitHub](http://www.github.com).  This environment makes it much easier for people to collaborate and share improvements.  You can even _fork_ your own copy of Cerb5 so you can keep track of your custom changes.

If you aren't able to use Git directly, GitHub provides a Subversion interface to our project repository.

On Unix-based servers you can check if Git is installed by typing:

    $ git --version

If you need to install Git, it's usually available in a package named `git-core`.  The actual package name will depend on your distribution of Linux/BSD.

On Unix-based servers you can check if Subversion is installed by typing:

    $ svn --version

On Windows-based servers, a graphical tool like [TortoiseGit](http://code.google.com/p/tortoisegit/) or [TortoiseSVN](http://tortoisesvn.tigris.org/) is your best option.

If you can't use Git or Subversion, you really should consider having us host your helpdesk on Cerb5 On-Demand rather than managing it yourself.<!--[How do you upgrade Cerb5 without SVN?](http://wiki.cerb5.com/wiki/Upgrade_Cerb5_without_SVN)_-->

### Preparation ###

* **Always make a backup** of your helpdesk database prior to upgrading. MySQL backups are best done with `mysqlhotcopy` or
    `mysqldump`.  See the chapter on [Backups](#backups) for more information.

* Copy your `framework.config.php` file to `backup.config.php`

* Change directory to your `cerb5` installation.

* The choice of using Git or Subversion to update will depend on how you originally installed the software.

#### Updating using Git on Linux ####

Ensure that you're using Git:

	$ git status
	
You can also verify that a `.git` directory exists.  If the above command returns an error, or the `.git` directory doesn't exist, then you probably installed using Subversion.  Skip to the next section.

Update the list of branches from the remote repository.

	$ git fetch origin
	
If this gives an error, you can check your list of remotes with the `git remote` command.  Use another remote in place of `remote`.
	
Display the list of available branches:
	
	$ git branch
	
Switch to the new version, using the `-m` switch to merge your local changes (e.g. `framework.config.php`):
	
	$ git checkout -m 5.4.1
	
#### Updating using Subversion on Linux ####

You can ensure that you're using Subversion with the following command:

	$ svn info
	
You can also verify that a `.svn` directory exists.  If this returns an error, or the `.svn` directory doesn't exist, then you may have installed the software using Git instead.  Refer to the previous section.
	
In most cases you can update to the latest stable version using:

	$ svn update
	
#### Updating with TortoiseSVN on Windows ####

**Using TortoiseSVN to update Cerberus Helpdesk**

![](images/maintenance/maintenance_updating_svn_win.png)

TortoiseSVN integrates with your Windows GUI. To update Cerberus Helpdesk:

* Open the folder containing your Cerb5 files.
* Right-click in the empty whitespace of the folder (as if you were going to create a new file).
* Choose "SVN Update" from the pop-up menu.

You should be shown a list of updated files and the new build versions for the project. Press the "OK" button when finished reading.

#### Dealing with conflicts ####

If you encounter conflicts while updating, you can attempt to resolve them manually, or you can revert your changes and restore your `framework.config.php` settings by hand.  Don't simply copy over the new file with your old file, because it may have changed in the recent version.

Ensure that you have no remaining conflicts before continuing with the upgrade.

**Git:**

	$ git status
	
**Subversion:**

	$ svn status

### Finishing the Upgrade ###

#### Permissions ####

You should set file ownership and permissions again after updating your files using Subversion.

**Unix-based servers:**

From your cerb5/ directory at the console (replace **www-data** with
your appropriate Apache user and group):

    chown -R www-data:www-data .
    chmod -R 0774 storage/

Permissions, especially on php files are a common upgrade issue. See **Troubleshooting** (below) for more details.

**Windows-based servers:**

Use Windows Explorer to set the appropriate write permissions on the `/cerb5/storage` directory for your IIS user.

#### Database schema updates ####

Some Cerb5 updates contain database changes which require a helpdesk administrator to finalize. This will prohibit all helpdesk activity (e.g., logins, scheduled tasks, mail parsing) to prevent any database corruption while you're between versions.

After your files are updated, attempt to log into your Cerb5 helpdesk instance as you normally would. If a database update is required the software will automatically prompt you. Upon finalizing you should be able to log in and continue working.

#### Community Tools ####

Very rarely, the `index.php` file which drives Community Tools like the Support Center may change during an upgrade.

How to tell if you need to update your Community Tool file:

* Log into your Cerb5 helpdesk.
* Click _setup_ from the top right.
* Click the _Community Portals_ menu.
* Click the _Configure_ menu item.
* Select any portal to edit it.
* Click the _Installation_ tab.
* Compare the following line from the `index.php` output with your deployed `index.php`:
	
	`define('SCRIPT_LAST_MODIFY', 2009070901); // last change`

* If the number is different you should replace the `index.php` file for your Community Tool with the new version from the helpdesk.

We're working on a way to make this check happen automatically.

#### Install directory ####

Git will usually leave your `cerb5/install` directory deleted, but Subversion will always restore it.  Delete the `install` directory if it exists.

<!--
#### Troubleshooting ####

If you opted for a "safe" upgrade by making a backup and moving your install to a different location, you may see a blank page or cache-related error in your browser when loading the Helpdesk. Try [clearing the cache](http://wiki.cerb5.com/wiki/Clearing_the_cache) to fix this.

Occasionally you may want to force your plugins to reload after a simple update that doesn't patch the database and clear out the cache. Click 'Helpdesk Setup' and select the plugins tab to automatically reload them.
-->

<!--
##### Linux errors caused by PHP file permissions #####

When upgrading, Subversion can change file permissions. When this happens you may see several symptoms:

* Cerb5 fails to start and the browser displays an *Internal Server Error* page.
* Cerberus displays in the browser, but nothing happens when you click some of the buttons or links.
* If you have access to your server logs, look for errors similar to *SoftException in Application* or other errors indicating a specific PHP file failed.

If you are receiving the *Internal Server Error* page, check the permissions on `index.php` in your main Cerb5 directory.

If you are receiving the *Internal Server Error* page, or Cerb5 fails to run when links or buttons are clicked, check the file permissions on the PHP files.
-->

<!--
**PHP permissions**

File permissions are dependent on server setup and vary widely. Before proceeding, if you have any questions about correct PHP file permissions, check the permissions on other browser accessible PHP files that are currently working, or check with a server administrator.

If PHP permissions are causing these errors, the permissions may be set to allow group or public writing to these files. Disallow writing to *public* and *group*. Then test your installation.

You can use `CHMOD` or an FTP client to set the permissions. With the exceptions noted in the upgrade procedure, all files within the Cerberus directory and sub-directories usually have the same permissions.
-->