# AutoMySQLBackup
===============
 A fork and further development of AutoMySQLBackup from sourceforge. http://sourceforge.net/projects/automysqlbackup/ 

## DISCLAIMER
-------------------------
I take no resposibility for any data loss or corruption when using this script.
This script will not help in the event of a hard drive crash. If a copy of the
backup has not been stored offline or on another PC. You should copy your backups
offline regularly for best protection.

Happy backing up...

## INSTALL
-------------------------
Extract the package into a directory. If you are reading this you have probably done
this already.

To install the Automysqlbackup the easy way.
1. Run the install.sh script.
2. Edit the /etc/automysqlbackup/myserver.conf file to customise your settings.
3. See usage section.

To install it manually (the hard way).
1. Create the /etc/automysqlbackup directory.
2. Copy in the automysqlbackup.conf file.
3. Copy the automysqlbackup file to /usr/local/bin and make executable.
4. cp /etc/automysqlbackup/automysqlbackup.conf /etc/automysqlbackup/myserver.conf
5. Edit the /etc/automysqlbackup/myserver.conf file to customise your settings.
6. See usage section.

# USAGE
-------------------------

Automysqlbackup can be run a number of ways, you can choose which is best for you.

1. Create a script as below called runmysqlbackup using the lines below:

#~~~~ Copy From Below Here ~~~~
#!/bin/sh

/usr/local/bin/automysqlbackup /etc/automysqlbackup/myserver.conf

chown root.root /var/backup/db* -R
find /var/backup/db* -type f -exec chmod 400 {} \;
find /var/backup/db* -type d -exec chmod 700 {} \;

#~~~~~ Copy To Above Here ~~~~

2. Save it to a suitable location or copy it to your /etc/cron.daily folder. 

3. Make it executable, i.e. chmod +x /etc/cron.daily/runmysqlbackup.


The backup can be run from the command line simply by running the following command.

  automysqlbackup /etc/automysqlbackup/myserver.conf

If you don't supply an argument for automysqlbackup, the default configuration
in the program automysqlbackup will be used unless a global file

  CONFIG_configfile="/etc/automysqlbackup/automysqlbackup.conf"

exists.

You can just copy the supplied automysqlbackup.conf as many times as you want
and use for separate configurations, i.e. for example different mysql servers.

## !!! NEW !!!
----------
As of version 3.0 we have added differential backups using the program diff. In an
effort to make the reconstruction of the full archives more user friendly, we
added new functionality to the script. Therefore, while preserving the old syntax,
we created options for the script, so that the new functions can be accessed.

Usage automysqlbackup options -cblh
-c CONFIG_FILE  Specify optional config file.
-b      Use backup method.
-l      List manifest entries.
-h      Show this help.

If you use these options, you have to specify everything according to them and can't
mix the old syntax with the new one. Example:

before (still valid!):

  >> automysqlbackup "myconfig.conf"

now:

  >> automysqlbackup -c "myconfig.conf" -b

which is equivalent to

  >> automysqlbackup -bc "myconfig.conf"

or in English: The order of the options doesn't matter, however those options expecting
arguments, have to be placed right before the argument (as seen above).

The option '-l' (List manifest entries) finds all Manifest files in your configuration
directory (you need to specify your optional config file - otherwise a fallback will be
used: global config file -> program internal default options). It then filters from which
databases these are and presents you with a list (you can select more than one!) of them.
Once you have chosen, you will be given a list of Manifest files, from which you choose
again and after that from which you choose differential files. When you have completed
all your selections, a list of selected differential files will be shown. You may then
choose what you want to be done with/to those files. At the moment the options are:
- create full backup out of differential one
- remove the differential backup and its Manifest entry.


## CONFIGURATION OPTIONS
-------------------------

!! "automysqlbackup" program contains a default configuration that should not be changed:

The global config file which overwrites the default configuration is located here
"/etc/automysqlbackup/automysqlbackup.conf" by default.

Please take a look at the supplied "automysqlbackup.conf" for information about the configuration options.

Default configuration
CONFIG_configfile="/etc/automysqlbackup/automysqlbackup.conf"
CONFIG_backup_dir='/var/backup/db'
CONFIG_do_monthly="01"
CONFIG_do_weekly="5"
CONFIG_rotation_daily=6
CONFIG_rotation_weekly=35
CONFIG_rotation_monthly=150
CONFIG_mysql_dump_usessl='yes'
CONFIG_mysql_dump_username='root'
CONFIG_mysql_dump_password=''
CONFIG_mysql_dump_host='localhost'
CONFIG_mysql_dump_socket=''
CONFIG_mysql_dump_create_database='no'
CONFIG_mysql_dump_use_separate_dirs='yes'
CONFIG_mysql_dump_compression='gzip'
CONFIG_mysql_dump_commcomp='no'
CONFIG_mysql_dump_latest='no'
CONFIG_mysql_dump_max_allowed_packet=''
CONFIG_db_names=()
CONFIG_db_month_names=()
CONFIG_db_exclude=( 'information_schema' )
CONFIG_mailcontent='log'
CONFIG_mail_maxattsize=4000
CONFIG_mail_address='root'
CONFIG_encrypt='no'
CONFIG_encrypt_password='password0123'

!! automysqlbackup (the shell program) accepts one parameter, the filename of a configuration file. The entries in there will supersede all others.

Please take a look at the supplied "automysqlbackup.conf" for information about the configuration options.



## ENCRYPTION
-------------------------

To decrypt run (replace bz2 with gz if using gzip):

openssl enc -aes-256-cbc -d -in encrypted_file_name(ex: *.enc.bz2) -out outputfilename.bz2 -pass pass:PASSWORD-USED-TO-ENCRYPT



## BACKUP ROTATION
-------------------------

Daily Backups are rotated weekly.
Weekly Backups are run on fridays, unless otherwise specified via CONFIG_do_weekly.
Weekly Backups are rotated on a 5 week cycle, unless otherwise specified via CONFIG_rotation_weekly.
Monthly Backups are run on the 1st of the month, unless otherwise specified via CONFIG_do_monthly.
Monthly Backups are rotated on a 5 month cycle, unless otherwise specified via CONFIG_rotation_monthly.

Suggestion: It may be a good idea to copy monthly backups offline or to another server.



## RESTORING
-------------------------

Firstly you will need to uncompress the backup file and decrypt it if encryption was used (see encryption section).

eg.
gunzip file.gz (or bunzip2 file.bz2)

Next you will need to use the mysql client to restore the DB from the sql file.

eg.
  mysql --user=username --pass=password --host=dbserver database < /path/file.sql
or
  mysql --user=username --pass=password --host=dbserver -e "source /path/file.sql" database

NOTE: Make sure you use "<" and not ">" in the above command because you are piping the file.sql to mysql and not the other way around.



Information
-----------

Creating backups means copy the data in a way where it can be restored again. To backup a mysql database server in most cases it is needed to create a dump from the database and each table. If the data, mysql stores in its data directory is simply copied, restoring the data in the mysql database will not be possible. This command line tool enables you to create and maintian mysql backups. You can backups of innodb and myisam tables.

Automysqldumper uses mysqldump for creating the sql backup. By default databases are backed up in separate gzipped files. To restore a database you can use:

```
zcat daily_andi_wiki_2016-12-24_03h59m_Saturday.sql.gz | mysql -u root -p
```

Change the name of the file to your needs. After this simple step you get back the data into your database.

To setup this script have a look at the automysqlbackup.conf file. In there you have several options to configure the mysql backup script to your needs.

Adjustments
-----------

You can find some original files from the sourceforge package. Some files are adjusted to my needs:
- support for MySQL 5.6 and MySQL 5.7
- support for login path (since MySQL 5.6.x a secure way to save your mysql credentials was implemented)
- adjusted the use of --ssl because it became depreated in MySQL 5.7. The parameter --ssl-mode=REQUIRED is used instead.

Add login path
--------------
Per default this script uses the login path automysqldump.

```
mysql_config_editor set --login-path=automysqldump --host=localhost --user=root --password
```

After that command give your mysql root password and you're done. If you want to do your backup with another user, simply change the username.


Backup your mysql server with ease by using this adjusted script. If you encounter any errors feel free to [drop an issue](https://github.com/sixhop/AutoMySQLBackup/issues/new). :) (English or German)
