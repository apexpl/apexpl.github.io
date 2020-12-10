
# Managing Multiple Client Servers with LINUX

If you're like me and have a good number of client servers you need to constantly access at different times and are running LINUX on 
your local PC, this article contains good time saving and organizational tips that have helped keep my work day flowing 
smoothly, allowing me to take care of my clients with speed and agility.  We'll go through a couple quick tips on how to efficiently both, login to remote servers via SSH, and 
mount them to our local filesystem with ease.

I personally enjoy using [Linux Mint](https://linuxmint.com/) as my choice of LINUX distro, but this article should work for all flavors of LINUX.  However, you 
should be using the /bin/bash shell, and can find out what shell you're currently using by opening terminal and running the command:

`echo $SHELL`

If this prints `/bin/bash`, then we're in good shape.


### ssh_config File

One huge time saver is the SSH config file located at `~/.ssh/config`.  First, let's start by creating a directory to store all of our SSH keys that we use 
to login to servers.  Open terminal, and run the following command:

`mkdir -m 0600 $HOME/.ssh_keys`

Now copy all of your SSH key files into this directory (eg. clienta.pem, clientb.pem, etc.).  Next, open up the `~/.ssh/config` file in a text 
editor by running the command:

`nano $HOME/.ssh/config`

Here's an example entry within the file using a SSH key:

~~~
host clienta
    hostname 124.58.2276.80
    user ubuntu
    IdentityFile /home/myuser/.ssh_keys/clienta.pem
~~~

Add sections of lines such as above to the `~/.ssh/config` file, one for each server you desire.  Then save and close the file by 
pressing Ctrl+X followed by the "Y" and Enter keys.  Once saved, you can now login to any server via SSH from any 
directory within terminal by simply typing:

`ssh clienta`

This will instantly log you into the server with the information under the `clienta` host you specified within the `~/.ssh/config` file.


### Local /bin/ Directory.

Let's expand on this by allowing easy mounting to remote servers by creating a /bin/ directory that's local to our user account.  Open terminal on your computer, and create a 
/bin/ directory by running the command:

`mkdir -m 0755 $HOME/bin`

Next, open the `~/.profile` file in a text editor by running the command:

`nano $HOME/.profile`

Scroll down to the very bottom of the file, and add the following lines by copying them to your clipboard, then within terminal by pressing Ctrl+Shift+V:

~~~
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
~~~

Then save and close the file by pressing Ctrl+X, followed by the "Y" and Enter keys.  This will save the new .profile file, which will check the newly created 
local /bin/ directory for any commands we try to run.


### Adding mount Bash Commands

First and foremost, we need to ensure the `sshfs` program is installed on our computer.  You may do this by running the following command:

`sudo apt-get -y install sshfs`

Now let's create a /mnt/ directory that will contain all the mounted directories to our remote servers.  Within terminal 
run the commands such as:

~~~
mkdir -m 0755 $HOME/mnt
mkdir -m 0755 $HOME/mnt/clienta
mkdir -m 0755 $HOME/mnt/clientb
~~~

Continue creating one sub-directory for each remote server you may potentially mount to.  Next, let's create the bash commands that we will 
run, and for example, for the `clienta` server open a file by running the following command in terminal:

`nano $HOME/bin/mount_clienta`

Modify the below line as necessary with the proper server information, then copy and paste it into the blank text editor within terminal by pressing Ctrl+Shift+V.

~~~
#!/bin/bash
sshfs -o IdentityFile=/home/myuser/.ssh_keys/clienta.pem ubuntu@124.58.2276.80:/var/www /home/myuser/mnt/clienta
~~~

Save and close the file by pressing Ctrl+X, followed by the "Y" and Enter keys.  Last, change permissions of the file so it's executable by running the command:

`chmod 0755 $HOME/bin/mount_client`

Now any time you need to mount to client A's remote server to transfer files to / from it, from any directory within terminal you can simply type `mount_clienta`, and the remote server mounted 
at the /var/www/ directory will be available to you via the `$HOME/mnt/clienta` directory.  You can then begin using that directory just as 
if it was a local directory on your computer.


### Conclusion

I hope this has given you a couple extra quick tips you can implement helping make things a little 
smoother and more efficient while taking care of your client's, and putting a smile on their face.  Take care, stay safe and KoVid free.




