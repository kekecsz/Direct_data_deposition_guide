# Guide to set up Direct Data Deposition from Google Drive to GitHub #
This guide will help you to make your own server which runs on Linux and automatically synchronizes data from Google Drive to GitHub repository. If you won't encounter any errors, this procedures should take about 2-3 hours. 

The guide is written up in a way that no prior expertise is needed with Linux or using remote servers.

## Authors
Pietro Rizzo - Lund University

Endre Csikós - ELTE

Zoltan Kekecs - ELTE and Lund University

## Correspondance
Contact Zoltan Kekecs via kekecs.zoltan@gmail.com

## I. Setting up a server with Ubuntu Linux ##

This guide is written up presuming that you will use a DigitalOcen Droplet with Ubuntu Linux running on the server. Most of this guide should work on other types of servers as well, as long as the server runs Ubuntu Linux, but we only tested the guide so far with DigitalOcen Droplet.
You can get a free 60 day trial by clicking on the following referral code. Disclosure: if you spend 25 USD after setting up your server using this link, I would get a one time 25 USD credit on my DigitalOcean account. 
https://m.do.co/c/9f9d58f02cb0
(Or, if you don’t want to use our referral link for some reason, you can use the following link for the same free trial: https://try.digitalocean.com/performance/)

Confirm email address provided through link received via email

Add billing info. Caution: this one is a bit tricky. Prepaid cards are not accepted. Other cards may also be declined.
But the really tricky part is that when your card IS accepted, the system doesn't notify you. You input your billing info, you click "Save card",
a loading sign appears in place of the button, and then the "Save card" button reappears as if nothing happened.
What you have to do, if this happens, is to go to your DigitalOcean page and check that your card has actually been accepted by looking into the Account>Billing section.

Now it is time to create a new Droplet.

While you are on your Digital Ocean page, click the green "Create" button in the top right and choose "Droplets".

Make the following choices in the menu that follows:

"Choose an Image" -> Ubuntu 16.04.6 x64. Or whatever is the closest thing to it. Updates happen.

"Plan option" -> *Standard*. Not that you have a choice.

$5/month, $0.007/hour, 1 GB/1 CPU, 25 GB SSD disk, 1000 GB transfer

You don't need Backups. Probably.

You don't need to add block storage.

"Choose a datacenter region": Whatever works best for you. Consider distance (affecting the data transfer speed) and ethical/legal issues related to sensible data storage (servers located in certain contries might be okay or not depending on your country/university/journal policy on the matter).

"Select additional options": Do not select any.

"Add your SSH keys": Ignore the step

"How many droplets"? 1.

"Choose a hostname". You'll read this name later, and you probably won't remember what it referres to. I would recommend something like "Droplet(ProjectName)".

Et voilá. Droplet done.

Write down some basic information for your own future reference:
```
	Droplet Name: [ProjectName] DirectDataDeposition
	IP Address: [ProjectName] 188.166.32.157
	Username: root (it's always "root")
	Password: [ProjectName]
```
Your droplet's password will also be mailed to you.

Now go on your Droplet page > Access > Console Access > Launch Console. The black window appears. You're inside the Droplet. Scary place huh? Don't worry, you won't have to stay here for too long. Be brave and proceed.

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
Why bother moving to PuTTY, then? Because on PuTTY you can select and copy lines of text, which is a great thing when you have to copypaste several strings of gibberish.
Install PuTTY from https://www.putty.org

Open Putty:
1. Find Droplet ipv4 in DigitalOcean Droplet page and copy it
2. Open PuTTY > Session and paste the ipv4 in the "Host Name (or IP adress)" line
3. Write a session name in the "Saved Sessions" line
4. Save 
5. Select saved session and Open it
6. Just reply "yes" in response to PuTTY's Security Alert.
PuTTY is now an interface on the Droplet. Login (username: "root", password: [Droplet_Password]).

Now you're looking inside the Droplet again, but this time you're doing it through the friendly glass of PuTTY. Believe it or not, PuTTY might seem a bit weird at first, but it is actually a good friend.

Now, how does one treat PuTTY? To copy text, select it with your mouse, move your cursor to the very end of selected area 
(not before, not after! Exactly where the highlighted portion of text ends) and press the wheel in the middle of your mouse. You do have one, right? I hope you do.
That will copy the selected text. In order to paste text in PuTTY, instead, press the right button of your mouse.

## II. Setting up the Google Drive Sync ##

First You should create a folder in your Drive root section, if you haven't already. I suggest naming the folder *testfolder*.
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
*gdrive* is the local folder which we are going to sync with the Google Drive folder.
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
1. Leave PuTTY
2. Copy, paste and follow link
3. Authorize
4. Copy verification code

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
User:	root@[Droplet_Name]:/srv/test_git_folder/gdrive# rclone ls [Remote_Name]:testfolder
//tesfolder is the name of the Google Drive folder you would like to sync with the local folder
Console:
       -1 doc2.docx
       -1 doc1.docx
```
This last passage is a test: input rclone ls \[Name-of-Drive]:\[Folder Name]. If everything works, then the console returns the correct list of the files present in the testfolder.
Now we should synchronize the Drive folder with our local folder:
```
User:	root@\[Droplet_Name]:/srv/test_git_folder/gdrive# rclone sync \[Name-of-Drive]:testfolder /srv/test_git_folder/gdrive/
User:	root@[Droplet_Name]:/srv/test_git_folder/gdrive# cd ..
User:	root@[Droplet_Name]:/srv/test_git_folder# cd ..
User:	root@[Droplet_Name]:/srv# 
User:	root@[Droplet_Name]:/srv# mkdir cron_script
User:	root@[Droplet_Name]:/srv# cd cron_script
```
The folder *cron_script* is going to be the folder for our script files.
```
User:	root@[Droplet_Name]:/srv/cron_script# nano /srv/cron_script/rclone-cron.sh
Console:
	[nano text editor appears]
User:	[input the following in nano text editor]	
	#!/bin/bash
	if pidof -o %PPID -x “rclone-cron.sh”; then 
	exit 1
	fi
	rclone sync [Remote_Name]:[drive_folder_name] /srv/test_git_folder/gdrive/
	exit
	[exit nano text editor by pressing Ctrl+x, then y, then Enter]
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
Make sure that you select */bin/nano* as it doesn't always appear as the second option.
```
Console:
	[another text editor opens]
User:	[scroll to the bottom of the text and paste the following:]
	*  *  *  *  *  /srv/cron_script/rclone-cron.sh > /tmp/cron_job_rclone.log 2>&1
```
Exit (Ctrl+x) and proceed to save it without altering the default name.
```
Console:
	crontab: installing new crontab
```

## III. Setting up sync between GitHub and Drive ##

Leave PuTTY and go to [GitHub](https://github.com). Create an account if you don't have one already.

Create new repository,
1. Set the repository name \[I recommend Repository\[Project Name]]
2. Make the repository Public
3. Do not "Initialize this repository with a README"
4. Click 'Create'

Back to PuTTY:
```
User:	root@[Droplet_Name]:/# apt install git
Console:[...]
User:	root@[Droplet_Name]:/# cd /srv/test_git_folder
User:	root@[Droplet_Name]:/srv/test_git_folder#
User:	root@[Droplet_Name]:/srv/test_git_folder# git init
Console:Initialized empty Git repository in /srv/test_git_folder/.git/
User:	root@[Droplet_Name]:/srv/test_git_folder# sudo git remote add origin https://github.com/[Github_Username]/[Repository_Name]
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
Leave PuTTY and check the Github repository: the README file should have appeared.

Back to PuTTY:
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
Since none of the listed keys end in .pub, that means no public keys are present. We'll need to create one.
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
	|&+++.+           |
	|o*oo= o          |
	|=.+.   . .       |
	|++ o .  E .      |
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
	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC7inJT0m5VwBWGAdMe0/Il+i/aWwUPZ45h92IJoSgyVICZwLcGznRfdcwWeOHnLiR7Zk3L/zlIEMpZHxMNmrQ5cgqaWilAcfOYF//BuG2SkeuuILI9xx3TWbpfuHlxiQ+zNhNLiqBPkRI5Mv/Yl9iO2SR7Cm3VyUDz1ulHLBE6+BMgkA37YELjwHJlxqASI6h9+wj/KzYK7r6fB6uorQaYwViBY/6HTwcheKXwK4j7bvattG0r4A+ucgOFsxYiHpMP3Vx6Jmk6RdQTUDu1IQ7lDuXMVYczVa04rgvhbymZhRuKR7EPTate4GZulf97U70M1KTqPKmQ33y0u0BuTluLIhgn+fO6cznLSmTQvWsipmxz/jF4H8/AUSMl+dMMOtdRL5K7UnLuL9j9uxymb9/dTTsgeBXZB4uk+l6qAlckajwUKSnWUpJMQC0ogaMLmB+eVIlRJv5eAxy0PSo5M/NB7LsO7PffnJbUPXrTeWV23cPGHCEtxiz5vTbse1m2G4lNeTC0vOJ/DE93dzceRb13tbjk1qqPiyfAjLmURRPdKVoQ1Eg8dVmoEKOWaytwjmY4+ZZMEcdBcD1xBNIhFHH80uYhYUWJA79R3+9LefK0pVSnb86gPyOtZ03d0EtCpO81dyHrvgYzcXb3G2PpD2yrOSQYrIDaGNYa0qNXAm1cRw== [Github_Email]
```
This long string of text is the SHH key.
1. Leave PuTTY
2. Go to GitHub and create a new SHH key (https://github.com/settings/ssh/new)
3. Pick a name for the key (I recommend "SSHkey\[Project name]")
4. Copy-paste the SHH key in the 'Key' box.
5. Enter your GitHub password if prompted to
6. SHH key has been created and can be found in https://github.com/settings/keys.

Back to PuTTY. Now it's time to test if this all works on the server:
```
User:	root@[Droplet_Name]:/# cd /srv/test_git_folder
User:	root@[Droplet_Name]:/srv/test_git_folder# git remote -v
Console:origin  https://github.com/[Github_Username]/[Repository_Name] (fetch)
Console:origin  https://github.com/[Github_Username]/[Repository_Name] (push)
User:	root@[Droplet_Name]:/srv/test_git_folder# git remote set-url origin git@github.com:[Github_Username]/[Repository_Name].git
User:	root@[Droplet_Name]:/srv/test_git_folder# git remote -v
Console:origin  git@github.com:[Github_Username]/[Repository_Name].git (fetch)
Console:origin  git@github.com:[Github_Username]/[Repository_Name].git (push)
User:	root@[Droplet_Name]:/srv/test_git_folder# ssh -vT git@github.com
```
*texts pouring in* ...in the end you should see something like this:
```
Console:Hi [Github_Username]! You've successfully authenticated, but GitHub does not provide shell access.
debug1: client_input_channel_req: channel 0 rtype exit-status reply 0
debug1: channel 0: free: client-session, nchannels 1
Transferred: sent 3412, received 2484 bytes, in 0.2 seconds
Bytes per second: sent 18018.5, received 13117.8
debug1: Exit status 1
```
If you see permission denied instead, please use [this guide](https://help.github.com/en/enterprise/2.16/user/articles/error-permission-denied-publickey) to resolve the issue (it's probably a typeo).
```
User:	root@[Droplet_Name]:/srv/test_git_folder# nano testfile2.txt
```
Enter some text in this testfile. Then,
```
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
User:	root@[Droplet_Name]:/srv/test_git_folder# cd ..
User:	root@[Droplet_Name]:/srv# cd cron_script
User:	root@[Droplet_Name]:/srv/cron_script# sudo touch sync.sh
User:	root@[Droplet_Name]:/srv/cron_script# sudo nano sync.sh
```
Paste the following:
```
#!/bin/bash
cd /srv/test_git_folder
git add *
git commit -am "Scheduled update `date`"
git push -u origin master
```
Exit via Ctrl+X,Y,\[enter]
```
User:	root@[Droplet_Name]:/srv/cron_script# sudo chmod +x sync.sh
User:	root@[Droplet_Name]:/srv/cron_script# sudo chmod 765 sync.sh
User:	root@[Droplet_Name]:/srv/cron_script# sudo -s
User:	root@[Droplet_Name]:/srv/cron_script# crontab -e
```
Insert the following line to the end:
```
*  *  *  *  *  /srv/cron_script/sync.sh > /tmp/job.log 2>&1
```
Save end exit via Control+X,Y,\[enter]
```
Console:crontab: installing new crontab
User:	root@[Droplet_Name]:/srv/cron_script# sudo service cron restart
```
**This is the end of our guide.** Thank your for following through. Hope all worked out just fine and now your auto-sync is up and running.



