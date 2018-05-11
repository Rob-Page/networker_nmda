# Oracle Networker Installation Guide on Centos
## PRE REQ
1. Install vscode
## Install Centos
1. Download Centos http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Everything-1708.iso

2. Open vmware workstatoion and select "create a new Virtual Machine"

    1. Add the location of the downladed iso to the disk drive
    2. Follow the prompts to install centos

Once the setup is complete login to your user account.

## Download packages (it is possible these version or links have been changed so choose what best fits your siutuation)
We are going to be installing Oracles Express(free version) Databse, as well as SQL developer, and Networker. At this point we can start downloading a few packages we will need to install those products:
### ALL of these should be downloaded onto your newly created Centos machine into the Downloads directory

1. Download Java 8 jre from Oracle (Linux x64 rpm package)
http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html

3. Download Java 8 jdk from Oracle Java SE Development Kit 8u171 (Linux x64 rpm package)
http://www.oracle.com/technetwork/pt/java/javase/downloads/jdk8-downloads-2133151.html?printOnly=1

2. Download Oracle llg xe (Oracle Database Express Edition 11g Release 2 for Linux x64)http://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html

3. Download Oracle sql developer (Linux RPM)
http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html

## Install JAVA jre and jdk
1. Install  java jre 
    1. cd into Downloads `cd ~/Downloads`
    2. rpm -ivh jrepackagename.rpm
2. Verify installation
    1. java -version
    2. rpm -qa | grep jre
3. install java JDK
    1. cd into Downloads `cd ~/Downloads`
    2. rpm -ivh jdkpackagename.rpm
4. Verify installation
    1. rpm -qa | grep jdk
5. Make sure you are using the right version of java for the java alias, check the version on the output:
`sudo update-alternatives --config java`
    1. select the newest installed version
## Install Oracle
1. Install Oracle
    1. cd into Downloads
    2. rpm -ivh oracle.rpm
2. Run configuration file that it tells you to run at the end of the install
3. Check to see if you can login to the Oracle server `sqlplus sys AS sysdba`

## Setup .bashrc with all the system varibles we setup
1. type `cd ~`
1. type `vi .bashrc`
2. type `i`
3. copy and paste the following text into the file:
TMPDIR=$TMP; export TMPDIR   
ORACLE_BASE=/u01/app/oracle; export ORACLE_BASE
ORACLE_HOME=$ORACLE_BASE/product/11.2.0/xe; export ORACLE_HOME
ORACLE_SID=XE; export ORACLE_SID
PATH=$ORACLE_HOME/bin:$PATH; export PATH
LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib:/usr/lib64; export LD_LIBRARY_PATH
CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH
export JRE_HOME=/usr/java/jdk1.8.0_171-amd64/jre
export PATH=$PATH:$JRE_HOME/bin
export JAVA_HOME=/usr/java/jdk1.8.0_171-amd64
export JAVA_PATH=$JAVA_HOME
export PATH=$PATH:$JAVA_HOME/bin
4.  hit the esc key, type `:wq` and hit the enter key
5.  type `source ~/.bashrc`
6.  type `vi /etc/.bashrc`
7.  type `i` to insert
8.  add the line `source ~/.bashrc`
9.  hit the esc key, type `:wq` and hit the enter key
10. Check oracle home path: `echo $ORACLE_HOME` this should print out the oracle home path. Somthing like /u01/app/product/11.2/XE/

## Install SQL Developer
1. Install sql Developer 
    1. cd into Downloads `cd ~/Downloads`
    2. rpm -ivh sqldeveloper.rpm
2. Verify installation
    1. rpm -qa | grep sql
3. open sql developer from the CLI `/opt/sqldeveloper/sqldeveloper.sh`

## Create a basic data base 
1. Inside sql developer there will be an option to open an exsisting linke XE
2. Click the XE hyper link and you will see a new database connection appear on the left side of the screen 
3. A text editor will apear on the main portion of the screen, copy and paste the following text:
    (DROP TABLE contact;
    CREATE TABLE contact
    (
          contact_id            INT          NOT NULL PRIMARY KEY 
        , contact_first_name    VARCHAR(240) NOT NULL
        , contact_last_name     VARCHAR(240) NOT NULL
        , contact_phone         INT     
        , contact_email         VARCHAR(240) NOT NULL
        , contact_chat          VARCHAR(240) 
    );
    
    create sequence contact_id_seq;
    )
    
    INSERT INTO 
    contact (contact_id, contact_first_name, contact_last_name, contact_phone, contact_email, contact_chat) 
    VALUES(contact_id_seq.nextval, 'Robert', 'Page', 3423, 'Robert.Page@dell.com', 'Robert.Page@emc.com');
    
    INSERT INTO 
    contact (contact_id, contact_first_name, contact_last_name, contact_phone, contact_email, contact_chat) 
    VALUES(contact_id_seq.nextval, 'Omar', 'Salazar', 3423, 'jesus.com', 'jesus.com');
    
    INSERT INTO 
    contact (contact_id, contact_first_name, contact_last_name, contact_phone, contact_email, contact_chat) 
    VALUES(contact_id_seq.nextval, 'pablo', 'cruz', 3423, 'tom.cruz@dell.com', 'mi3@emc.com');
    
    INSERT INTO 
    contact (contact_id, contact_first_name, contact_last_name, contact_phone, contact_email, contact_chat) 
    VALUES(contact_id_seq.nextval, 'Perry', 'POW', 3423, 'lostatsea@dell.com', 'Prisner.of.war@emc.com');
     
     select * from contact;
4. click the play button on the top left of the text editor;
5. You should a table with results pop up bellow.

## Prepare our database for backup
1. Log into sqlplus on the xe database: `sqlplus sys@xe`
2. shutdown the database: `shutdown` 
3. startup and mount the database: `startup mount`
4. Modify the databse mode to archivelog: `alter database archivelog;`
5. Modify the database to an open state: `alter database open;`
6. Log out of sqlplus `quit`

## Create a Backup to local disk using RMAN
1. Create a new directory to hold our backups: `sudo mkdir -p /backup/rman`
2. Set permissions on the directory(DO NOT DO THIS IN A SENSITIVE ENVIRONMENT DEFINE PERMISSIONS FOR YOUR USERS AS NEEED INSTEAD OF A BLANKET ALL READ WRITE): `chmod 777 /backup/rman`
3. login to RMAN with the sys user on the xe database: `rman TARGET SYS@xe`
4. Show the xe databse rman configuration: `SHOW ALL;`
5. Configure a default channel for this databse: `CONFIGURE CHANNEL DEVICE TYPE DISK FORMAT '/backup/rman/%d_full_%u_%s_%p';`
    1. This channel defined as a disk location will output all our backups to /backup/rman in the format  we defined "%d_full_%u_%s_%p" 
    2. %u generates a unique name
    3. %s represents the backup set number
    4. %p represents the backup piece number
    5. %d represents the database name
6. Configure RMAN to retain backups for 7 days:`CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;`
7. Run a backup: `BACKUP AS BACKUPSET DATABASE PLUS ARCHIVELOG;`
8. Quit rman: `quit`
9. See the backup files we just made: `ls -ltr /backup/rman`


## Install Net worker

###Install dependencies
1. `yum install ksh`
2. `yum install glibc.i686`
3. `yum install nss-softokn-freebl.i686`
### Setup file system
1. Make a base directory: `mkdir /nsr`
OR
1. (optional) go into another disk and mkdir nsr
    `mkdir -p /disk2/nsr`
2. create symbolic link to the other location: 
    `ln -s /disk2/nsr /nsr`
3. Verify that it is pointing right:
    `ls -ltr`
    should see: lrwxrwxrwx.   1 root root    10 May  3 13:17 nsr -> /disk2/nsr
#### EX of why we do symbolic links
We could use a symbolic link to a mounted disk to aviod overloading our root directory during a -D9 log gathering or other high space using functions

### Setup Networking for client
To avoid issues with localhost references we need to name our host and define a path to it.
#### Change sysconfig networking file
1. `vi /etc/sysconfig/network`
2. type `i` to insert new characters
3. Add this line to the bottom of the file: `NETWORKING=yes`
4. Add this line to the bottom of the file: `HOSTNAME=nwserver01`
5. type esc, then `:wq`
#### Change hosts file
1. `vi /etc/hosts`
2. type `i` to insert new characters
2. Add this line to the bottom of the file: `x.x.x.x nwserver01`
3. type esc, then `:wq`
#### Change machine's hostname
1. `hostnamectl set-hostname nwserver01`
2. reboot machine to apply changes `reboot`

### Download Networker
1. download Networker 9.2 from : ftp://ftp•••••••••.legato.com/pub/NetWorker/ 
2. untar and unzip the folder: `tar -xzf file_name.tar.gz`
### CHOOSE ONE INSTALATION
#### For a a CLIENT
`rpm -ivh lgtoclnt-nw*.rpm`
#### For a STORAGE NODE
`rpm -ivh lgtoclnt-nw*.rpm lgtoxtdclnt*.rpm lgtonode*.rpm`
#### For a NETWOKER SERVER 
`rpm -ivh lgtoclnt*.rpm lgtoxtdclnt*.rpm lgtonode*.rpm lgtoserv*.rpm lgtoauthc*.rpm`
##### Configure authentication server
`/opt/nsr/authc-server/scripts/authc_configure.sh`


### POST INSTALLATION
1. Confirm which packages are installed: `rpm -qa | grep lgto`
2. Start Services(deamons) for networker: `/etc/init.d/networker start`
3. Confirm Services(deamons) are running:`/etc/init.d/networker status`

### INSTALL NMC
1. Confirm that Services(deamons) are running :`/etc/init.d/networker status`
2. Instal NMC package: `rpm -ivh lgtonmc*.rpm`
3. Run NMC configuration script: `/opt/lgtonmc/bin/nmc_config`
4. Confirm nw and gst services are up: 
`/etc/init.d/networker status`
`/etc/init.d/gst status`
5. Navigate to localhost:9000 in a web browser 
6. Download the gconsole.jnlp by clicking the higlighted here link in "click here to start NetWorker Management Console."
6. Open a new terminal  (NOT AS ROOT) an navigate to downloads `cd ~/Downloads` run the java nmc program by typing `javaws gconsole.jnlp`
7. Fill out user as `Administrator` and password as the password you defined while filling out the /opt/lgtonmc/bin/nmc_config
8. Fillout the wizard as prompted

### INSTALL NMDA
1. Confirm that Services(deamons) are running :`/etc/init.d/networker status`
2. Instal NMC package: `rpm -ivh lgtonmda*.rpm`
4. Confirm nw and gst services are up: 
`/etc/init.d/networker status`

### LINK LIBRARY

1. cd $ORACLE_HOME/lib
2. ln -s /usr/lib/libnsrora.so libobk.so
3. confirm that the link exsists by typing `ls -ltr|grep libobk` output should see something like:
lrwxrwxrwx.  1 rob rob    21 May 10 14:29 libobk.so -> /usr/lib/libnsrora.so

### ADD ORACLE CLIENT
1.  Open NMC
2.  Add a new client through the wizard
3.  When asked for the TNS directory enter: /u01/app/oracle/product/11.2.0/xe/network/admin
4.  Select OS user and enter in username: oracle, and select XE as the database SID
5.  Proceed through the rest of the wizard as needed.
6.  Create a new AFD device called oracle backup.
7.  Add that AFD device to the default pool
8.  Add your new client to a group
9.  Create a new workflow and that coraspodes to the group and add it to a policy.
10. Run the policy backup.

### Restore test case.
1. type `sqlplus sys AS sysdba` into a commandline
2. Verify that the table exsists: `DESC contact`
3. Delete data in the table: `DELETE contact;`
4. Drop the table in the database: `DROP TABLE contact;`
5. Verify that the table no longer exsists: `DESC contact`

### Restore the database using the NMC
1.  Go to the protection tab
2.  Select "clients" from the left hand side
3.  Right click on the client nwserver01 and click recover
4.  Go through the wizard, select Oracle when asked what kind.
5.  When asked if it is an original or duplicate Select Restore to original database.
6.  When asked for the TNS directory enter: /u01/app/oracle/product/11.2.0/xe/network/admin
7.  Select OS user and enter in username: oracle, and select XE as the database SID
8.  Select Restore and recover the entire database or specific database objects.
9.  Select the whole database.
10. When asked to if you want it shutdown and mount the database select yes.
11. Use default datafile location
12. Edit RMAN script to Allocate and release only one channel(This is only for our situation because of our free database version restrictions. Normally default is going to be okay)
13. Run recovery
