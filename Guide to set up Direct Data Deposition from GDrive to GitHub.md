# Guide to set up Direct Data Deposition from Google Drive to GitHub #
This guide will help you to set up your own remote server which runs on Linux and automatically synchronizes data from Google Drive to GitHub repository. If you won't encounter any errors, this procedures should take about 2-3 hours. 

The guide is written up in a way that no prior expertise is needed with Linux or using remote servers.

## Authors
Zoltan Kekecs - ELTE and Lund University

Pietro Rizzo - Lund University

Endre Csikós - ELTE

## Correspondance
Contact Zoltan Kekecs via kekecs.zoltan@gmail.com

### I. Setting up a remote server with Ubuntu Linux ##
This guide is written presuming that you will use a DigitalOcen Droplet with Ubuntu Linux running on the server. Most of this guide should work on other types of servers as well, as long as the server runs Ubuntu Linux, but we only tested the guide so far with DigitalOcen Droplet. You can get a free 60 day trial by clicking on the following referral code. Disclosure: if you spend 25 USD after setting up your server using this link, Zoltan Kekecs (one of the authors of the guide) would get a one time 25 USD credit on his DigitalOcean account. https://m.do.co/c/9f9d58f02cb0 (Or, if you don’t want to use our referral link for some reason, you can use the following link for the same free trial: https://try.digitalocean.com/performance/)

Confirm email address provided through link received via email

Add billing info. Caution: this one is a bit tricky. Prepaid cards are not accepted. Other cards may also be declined.
But the really tricky part is that when your card IS accepted, the system doesn't notify you. You input your billing info, you click "Save card",
a loading sign appears in place of the button, and then the "Save card" button reappears as if nothing happened.
What you have to do if this happens, is to go to your DigitalOcean page and check that your card has actually been accepted by looking into the Account>Billing section.

Now it is time to create a new Droplet.

While you are on your Digital Ocean page, click the green "Create" button in the top right and choose "Droplets".

Make the following choices in the menu that follows:
"Choose an Image" -> Ubuntu 16.04.6 x64. (The version number will probably be higher by the time you read this guide) Updates happen.

"Plan option" -> **Standard**. $5/month, $0.007/hour, 1 GB/1 CPU, 25 GB SSD disk, 1000 GB transfer

Bakcups are not necessary.

Block storage is not necessary.

"Choose a datacenter region": Whatever works best for you. Consider distance (affecting the data transfer speed) and ethical/legal issues related to sensible data storage (servers located in certain countries might be okay or not depending on your country/university/journal policy on the matter).

"Select additional options": Nothing esle is required for the guide to work.

"Add your SSH keys": You can ignore this step.

"How many droplets"? One is enough.

"Choose a hostname". Choose a host name that you will remember later.

Et voilá. You have your very own remote server.

Write down some basic information for your own future reference. You will need the following information later:
```
	Droplet Name: 
	IP Address: 
	Username: 
	Password: 
```
Your droplet's password will also be mailed to you.

Now go on your Droplet page > Access > Console Access > Launch Console. A console appears where you can interact with your remote server.

In Droplet:
```
User:	root@[Droplet_Name]:~# sudo apt update
Console:*lots of stuff*
User:	root@[Droplet_Name]:~# apt install unzip
Console:*lots of stuff*
User:	root@[Droplet_Name]:~# curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
Console:*lots of stuff*
User:	root@[Droplet_Name]:~# unzip rclone-current-linux-amd64.zip
Console:*lots of stuff*
User:	root@[Droplet_Name]:~# cd rclone-*-linux-amd64
User:	root@[Droplet_Name]:~/rclone-v1.47.0-linux-amd64# sudo cp rclone /usr/bin/
User:	root@[Droplet_Name]:~/rclone-v1.47.0-linux-amd64# sudo chown root:root /usr/bin/rclone
User:	root@[Droplet_Name]:~/rclone-v1.47.0-linux-amd64# sudo chmod 755 /usr/bin/rclone
User:	root@[Droplet_Name]:~/rclone-v1.47.0-linux-amd64# sudo mkdir -p /usr/local/share/man/man1
User:	root@[Droplet_Name]:~/rclone-v1.47.0-linux-amd64# sudo cp rclone.1 /usr/local/share/man/man1/
User:	root@[Droplet_Name]:~/rclone-v1.47.0-linux-amd64# sudo mandb
Console:[lots of stuff]
```
Close the Droplet console.

Now it's time for you to discover PuTTY, a terminal emulator. In PuTTY you can do the same things you would do in the Droplet console. 
Why bother moving to PuTTY, then? Because on PuTTY you can select and copy lines of text, which is a great thing when you have to copy and paste several strings of long text.
Install PuTTY from https://www.putty.org

Open Putty:

1. Find Droplet ipv4 in DigitalOcean Droplet page and copy it
2. Open PuTTY > Session and paste the ipv4 in the "Host Name (or IP adress)" line
3. Write a session name in the "Saved Sessions" line (doesn't really matter what name you put here, but it helps if its a name you will recognize later as your direct data deposition server)
4. Save 
5. Select saved session and Open it
6. Reply "yes" to Security Alert.

Now PuTTY will open a console to your remote server. Login (username: "root", password: [Droplet_Password]). 

### General Putty advice ###

ctrl+c and ctrl+v does not work in the console. However, when using putty, you can copy and paste things using other hotkeys:

You can **copy text** from the console by selecting it with your mouse, move your cursor to the very end of selection (not before, not after! Exactly where the highlighted portion of text ends) and press the middle mouse button. This will copy the selected text onto your computer clipboard when using Windows. 

In order to **paste text** from your clipboard into the PuTTY console, press the right mouse button of your mouse.

## II. Setting up the Google Drive Sync ##

### Linking a Google Drive folder to a folder on your droplet ###

First You should create a folder in your Google Drive root section. I suggest naming the folder **testfolder**. You should also put a few files in this folder. It is OK if they are just text filed or documents, just so that you can test whether everything works as intended.


Go back to PuTTY:
```
User:	root@[Droplet_Name]:~# cd ..
User:	root@[Droplet_Name]:/#
User:	root@[Droplet_Name]:/# cd srv
User:	root@[Droplet_Name]:/srv# mkdir test_git_folder
User:	root@[Droplet_Name]:/srv# cd test_git_folder
User:	root@[Droplet_Name]:/srv/test_git_folder#
User:	root@[Droplet_Name]:/srv/test_git_folder# mkdir gdrive
User:	root@[Droplet_Name]:/srv/test_git_folder# cd gdrive
```
**gdrive** is the "local" folder on your remote server which we are going to sync with the Google Drive folder.
```
User:	root@[Droplet_Name]:/srv/test_git_folder/gdrive# rclone config
Console:
	[yyyy/mm/dd hh:mm:ss] NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults
	No remotes found - make a new one
	n) New remote
	s) Set configuration password
	q) Quit config
User:	n/s/q> n
User:	name> [I recommend "Remote[ProjectName]"]
Console:Type of storage to configure.
	Choose a number from below, or type in your own value
	[long list of options]
User:	Storage> drive 
Console:[...]
User:	client_id> [Enter]
Console:[...]
User:	client_secret> [Enter]
Console:[...]
User:	scope> drive
Console:[...]
User:	root_folder_id> [Enter] 
User:	service_account_file> [Enter]
Console:Edit advanced config? (y/n)
	y) Yes
	n) No
User:	y/n> n
Console:Remote config
	Use auto config?
	 * Say Y if not sure
	 * Say N if you are working on a remote or headless machine or Y didn't work
	y) Yes
	n) No
User:	y/n> n
Console:If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/oauth2/auth?access_type=offline&client_ [...etc.]	
	Log in and authorize rclone for access
	Enter verification code> 
```
1. Copy the link starting with https://accounts.google.com..., and paste it in your preferred internet browser.
2. Authorize
3. Copy verification code from the browser and paste it into the console when prompted to Enter verification code

Back to PuTTY:
```
User:	Enter verification code> \[paste code here]
Console:
	--------------------
	[[Remote_Name]]
	type = drive
	scope = drive
	token = {"access_token":"ya29.GlsJBzUG29UGKl94ItLp30N-d_0k1zUkIIWmEetN3GR8bYaAmCCM_Yp36dsBWC1uXpKYxYzJkf6gh40lFA6sYwphkPHHx8Z-ajGYynG-YpX4yUfnu3IjHUssO9ah","token_type":"Bearer","refresh_token":"1/hyDJZAgL9WucRbdgaWtSTMnsU7YD2mxsHgv62T-HIIM","expiry":"2019-05-14T16:22:28.152944127Z"}
	--------------------
	y) Yes this is OK
	e) Edit this remote
	d) Delete this remote
User:	y/e/d> y
Console:
	Current remotes:

	Name                 Type
	====                 ====
	[Remote_Name]                  drive

	e) Edit existing remote
	n) New remote
	d) Delete remote
	r) Rename remote
	c) Copy remote
	s) Set configuration password
	q) Quit config
	e/n/d/r/c/s/q>
User: e/n/d/r/c/s/q> q
```

You have now linked the google drive folder with the local folder. To test if this works, you can do the following: 


```
User:	root@[Droplet_Name]:/srv/test_git_folder/gdrive# rclone ls [Remote_Name]:[Folder_name_within_drive]
```
Remote_Name is the name of the New remote you entered earlier when you created it.
Folder_name_within_drive is the name of the Google Drive folder you would like to sync with the local folder (e.g. testfolder).
If everything works, then the console returns the correct list of the files present in the testfolder within your gogle drive.

### Synchronizing the Drive folder with your local folder: ###

Now you need to set up automatic synchronization of the Drive folder with our local folder. The basic idea is to create a simple computer program (a script) that will periodically run and synch the two folders. We use a program in Linux called cron for this purpose. (This is also sometimes called "setting up a cron job".)


```
User:	root@\[Droplet_Name]:/srv/test_git_folder/gdrive# rclone sync \[Name-of-Drive]:testfolder /srv/test_git_folder/gdrive/
User:	root@[Droplet_Name]:/srv/test_git_folder/gdrive# cd ..
User:	root@[Droplet_Name]:/srv/test_git_folder# cd ..
User:	root@[Droplet_Name]:/srv# 
User:	root@[Droplet_Name]:/srv# mkdir cron_script
User:	root@[Droplet_Name]:/srv# cd cron_script
```

The folder **cron_script** is going to be the folder containing the script file that cron will have to execute periodically. Now we write the actual script that will synchronize the folders. This script will be stored in a simple text file called rclone-cron.sh, which we can create and edit using the nano text editor in Linux.


```
User:	root@[Droplet_Name]:/srv/cron_script# nano /srv/cron_script/rclone-cron.sh
Console:
	[nano text editor appears]
User:	[input the following in nano text editor]	
	#!/bin/bash
	if pidof -o %PPID -x “rclone-cron.sh”; then 
	exit 1
	fi
	rclone sync [Remote_Name]:[Folder_name_within_drive] /srv/test_git_folder/gdrive/
	exit
	
	[exit nano text editor by pressing Ctrl+x, then y, then Enter]
```


Now you need to set up the cron job to periodically execute this script. This can be done by editing the 'crontab':


```	
User:	root@[Droplet_Name]:/srv/cron_script# cd ..
User:	root@[Droplet_Name]:/srv# cd ..
User:	root@[Droplet_Name]:/# chmod a+x /srv/cron_script/rclone-cron.sh
User:	root@[Droplet_Name]:/# crontab -e
Console:
	no crontab for root - using an empty one

	Select an editor.  To change later, run 'select-editor'.
	  1. /bin/ed
	  2. /bin/nano        <---- easiest
	  3. /usr/bin/vim.basic
	  4. /usr/bin/vim.tiny

	Choose 1-4 [2]:
User:	Choose 1-4 [2]: 2 
```

Make sure that you select **/bin/nano** as it doesn't always appear as the second option.

```
Console:
	[text editor opens]
User:	[scroll to the bottom of the text and paste the following:]
	*  *  *  *  *  /srv/cron_script/rclone-cron.sh > /tmp/cron_job_rclone.log 2>&1
	
	[Exit (Ctrl+x) and proceed to save it without altering the default name.]
	
Console:
	crontab: installing new crontab
```

The five stars (*) in this script indicate that the script will be executed (synchronization will happen) every 60 seconds. If you want to set up less frequent synching, you can use https://crontab.guru/ to find the proper code for your purposes. 

You should test whether the auto-sync between your Drive and your local folder works by putting a new text file in your google drive folder. Wait 2-3 minutes and list the contents of your local folder like this:


```
User:	root@[Droplet_Name]:/# cd /srv/test_git_folder/gdrive
User:	root@[Droplet_Name]:/srv/test_git_folder/gdrive# ls
Console:
  [file names will be listed here]
```

Ideally, the list of files in the local folder should match that in your Google Drive folder. 

IMPORTANT NOTE: rclone does not directly sync gdoc files. These google documents are converted into another file format. By default this is word .docx format for google text files and excel .xlsx format for google spreadsheets. Unfortunately rclone does not sync google forms, so google forms will nto appear in your local folder (and they will not be pushed to GitHub either). 

If you have verified that all the above works, proceed to the next step. 

(If you encountered some error or the drive folder contents do not seem to match the local folder contents, you need to first investigate the source of the problem and make sure that you can set up auto-sync with drive before you move forward. There are many things that could go wrong, so we do not provide troubleshooting tips here.)

## III. Setting up sync between GitHub and Drive ##

Visit [GitHub](https://github.com) in your internet browser. [Create a GitHub account](https://m.wikihow.com/Create-an-Account-on-GitHub) if you don't have one already.

Create new repository,

1. Set the repository name \[I recommend Repository\[Project Name]]
2. Make the repository Public
3. Do not "Initialize this repository with a README"
4. Click 'Create'

### Linking a local folder to a GitHub repository ###

Now go back to PuTTY. Here you will install git, the program that will sync your local folder's content with your GitHub repository that you just created. After installation you will link the local folder and the repository.

```
User:	root@[Droplet_Name]:/srv/test_git_folder/gdrive# apt install git
Console:[...]
User:	root@[Droplet_Name]:/srv/test_git_folder/gdrive# cd /srv/test_git_folder
User:	root@[Droplet_Name]:/srv/test_git_folder#
User:	root@[Droplet_Name]:/srv/test_git_folder# git init
Console:Initialized empty Git repository in /srv/test_git_folder/.git/
User:	root@[Droplet_Name]:/srv/test_git_folder# sudo git remote add origin https://github.com/[Github_Username]/[Repository_Name]
```

You can test whether the link between the local folder and the repository works by modifying the README.md file, and pushing the contents of the local folder to the repository.

```
User:	root@[Droplet_Name]:/srv/test_git_folder# sudo bash -c 'echo "hello" > README.md'
User:	root@[Droplet_Name]:/srv/test_git_folder# sudo git add *
User:	root@[Droplet_Name]:/srv/test_git_folder# git config --global user.name "[Github_Username]"
User:	root@[Droplet_Name]:/srv/test_git_folder# git config --global user.email "[Github_Email]"
User:	root@[Droplet_Name]:/srv/test_git_folder# sudo git commit -m "test commit message"
User:	root@[Droplet_Name]:/srv/test_git_folder# sudo git push -u origin master
Console:
	On branch master
	nothing to commit, working directory clean
User:	Username for 'https://github.com': [Github_Username]
User:	Password for 'https://[Github_Username]@github.com':
Console:
	Counting objects: 3, done.
	Writing objects: 100% (3/3), 228 bytes | 0 bytes/s, done.
	Total 3 (delta 0), reused 0 (delta 0)
	To https://github.com/[Github_Username]/[Repository_Name]
	 * [new branch]      master -> master
	Branch master set up to track remote branch master from origin.
```
Wait a few minutes and check your GitHub repository in your internet browse. The README file should now have the text "hello" in it.

### Configure SSH-key authentication ###

Notice that you had to enter your credentials for GitHub when you pushed the local folder content to the repository. This is fine when you amnually push, but if you want to set up auto-sync the same would not work. To overcome this, you will have to set up using SSH-key authentication with GitHub. You will first need to enable this in your Linux server and create a public SSH-key.

In PuTTY:
```
User:	root@[Droplet_Name]:/srv/test_git_folder# cd ..
User:	root@[Droplet_Name]:/srv# cd ..
User:	root@[Droplet_Name]:/# nano /etc/ssh/sshd_config
Console:
	[nano text editor appears]
User:	[perform the following in nano text editor]	
		set "PermitRootLogin" to "PermitRootLogin yes"
		set "PermitEmptyPasswords" to "PermitEmptyPasswords yes"
		set "PasswordAuthentication" to "PasswordAuthentication yes"
		
	[exit nano text editor by pressing Ctrl+x, then y, then Enter]
	
User:	root@[Droplet_Name]:/# ls -al /root/.ssh
Console:
	total 8
	drwx------ 2 root root 4096 May 14 14:13 .
	drwx------ 8 root root 4096 May 14 16:35 ..
	-rw------- 1 root root    0 May 14 14:13 authorized_keys
```

Since none of the listed keys end in .pub, that means no public keys are present. Let's create one:


```
User:	ssh-keygen -t rsa -b 4096 -C "[Github_Email]"
Console:Enter file in which to save the key (/root/.ssh/id_rsa):
User:	[Enter]
Console:Enter passphrase (empty for no passphrase):
User:	[Enter]
Console:Enter same passphrase again:
User:	[Enter]
Console:
	Your identification has been saved in /root/.ssh/id_rsa.
	Your public key has been saved in /root/.ssh/id_rsa.pub.
	The key fingerprint is:
	SHA256:qqw8oQrHfQ7FYgwSMAJErNUp3YMkoEfjCmp6e411w4U [Github_Email]
	The key's randomart image is:
	+---[RSA 4096]----+
	|                 |
	|o*oo= o          |
	|=.+.   . .       |
	|                 |
	|o.  + o.S.       |
	|o..o o..+        |
	|o.+.o+o. .       |
	|o+.oo=.          |
	|o ++o .          |
	+----[SHA256]-----+
User:	root@[Droplet_Name]:/# eval $(ssh-agent -s)
Console:Agent pid 12740
User:	root@[Droplet_Name]:/# ssh-add /root/.ssh/id_rsa
Console:Identity added: root/.ssh/id_rsa (root/.ssh/id_rsa)
User:	root@[Droplet_Name]:/# cat /root/.ssh/id_rsa.pub
Console:
	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC7inJT0m5VwBWGAdMe0/I[...long string of characters...]XAm1cRw== [Github_Email]
```

This long string of text is the SHH key.

1. Go to GitHub in your web browser and [create a new SHH key](https://github.com/settings/ssh/new) 
2. Pick a name for the key (I recommend "SSHkey\[Project name]")
3. Copy-paste the SHH key into the 'Key' box. (you need to copy the whole string starting with ssh-rsa and ending with your GitHub emial.)
4. Enter your GitHub password when prompted


An SHH key has now been added to your GitHub account and can be found in https://github.com/settings/keys.
Now let's switch from https to SHH authentication in git on your remote server as well.

First, you can check your current git remote in PuTTY

```
User:	root@[Droplet_Name]:/# cd /srv/test_git_folder
User:	root@[Droplet_Name]:/srv/test_git_folder# git remote -v
Console:origin  https://github.com/[Github_Username]/[Repository_Name] (fetch)
Console:origin  https://github.com/[Github_Username]/[Repository_Name] (push)
```

The fact that the git remome is https://github.com/... means that you are using https authentication.

You need to switch the remote URL to git@github.com:[Github_Username]/[Repository_Name].git for the SSH authentication to take effect. You can do this by:

```
User:	root@[Droplet_Name]:/srv/test_git_folder# git remote set-url origin git@github.com:[Github_Username]/[Repository_Name].git
User:	root@[Droplet_Name]:/srv/test_git_folder# git remote -v
Console:origin  git@github.com:[Github_Username]/[Repository_Name].git (fetch)
Console:origin  git@github.com:[Github_Username]/[Repository_Name].git (push)
User:	root@[Droplet_Name]:/srv/test_git_folder# ssh -vT git@github.com
```
*texts pouring in* ...in the end you should see something like this.
(If you see permission denied instead, please use [this guide](https://help.github.com/en/enterprise/2.16/user/articles/error-permission-denied-publickey) to resolve the issue (it's probably a typo).)

```
Console:Hi [Github_Username]! You've successfully authenticated, but GitHub does not provide shell access.
debug1: client_input_channel_req: channel 0 rtype exit-status reply 0
debug1: channel 0: free: client-session, nchannels 1
Transferred: sent 3412, received 2484 bytes, in 0.2 seconds
Bytes per second: sent 18018.5, received 13117.8
debug1: Exit status 1
```

Now let's test whether switching to SSH authentication worked. In PuTTY create a new text file named testfile2.txt and see if you can push it to your GitHub repository.


```
User:	root@[Droplet_Name]:/srv/test_git_folder# nano testfile2.txt
  
  [enter some text in the nano text editor, anything will do, like "hello" or "test"]
  
  [exit nano text editor by pressing Ctrl+x, then y, then Enter]
  
User:	root@[Droplet_Name]:/srv/test_git_folder# git add testfile2.txt
User:	root@[Droplet_Name]:/srv/test_git_folder# git commit -m "test push"
Console:On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
User:	root@[Droplet_Name]:/srv/test_git_folder# git push -u origin master
Console:Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 316 bytes | 316.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:[Github_Username]/[Repository_Name].git
   288f134..84be1d6  master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

After a few seconds you should see the testfile2.txt file when you check your GitHub repository with your web browser.

(If you don't see this, you need to investigate the source of the issue and fix it before moving forward, because if a single push does not work, auto-push will not work either.)


### Set up periodic auto-push from the local folder to the GitHub repository  ###

The last step is to set up automatic push from the local folder to GitHub. Similarly to the auto-sync with Google Drive, the idea is to set up a cron job, where cron would periodically execute a script that will do the git push.

Let's create a script as a text file named sync.sh and save it to the folder /srv/cron_script.

```
User:	root@[Droplet_Name]:/srv/test_git_folder# cd ..
User:	root@[Droplet_Name]:/srv# cd cron_script
User:	root@[Droplet_Name]:/srv/cron_script# sudo touch sync.sh
User:	root@[Droplet_Name]:/srv/cron_script# sudo nano sync.sh
```

Paste the following into the nano text editor:

```
#!/bin/bash
cd /srv/test_git_folder
git add *
git commit -am "Scheduled update `date`"
git push -u origin master
  [exit nano text editor by pressing Ctrl+x, then y, then Enter]
  
User:	root@[Droplet_Name]:/srv/cron_script# sudo chmod +x sync.sh
User:	root@[Droplet_Name]:/srv/cron_script# sudo chmod 765 sync.sh
User:	root@[Droplet_Name]:/srv/cron_script# sudo -s
User:	root@[Droplet_Name]:/srv/cron_script# crontab -e
```
Insert the following line as the last line in the crontab:
```
*  *  *  *  *  /srv/cron_script/sync.sh > /tmp/job.log 2>&1
	[Exit (Ctrl+x) and proceed to save it without altering the default name.]
	
Console:crontab: installing new crontab
User:	root@[Droplet_Name]:/srv/cron_script# sudo service cron restart
```


Thank your for following through. Hope all worked out just fine and now your auto-sync is up and running.

**This is the end of our guide.** 