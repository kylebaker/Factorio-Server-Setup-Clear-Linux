**Why I'm doing this?**
My friends and I play factorio and having a dedicated server makes that easier for us to play together. As well, I like having excuses to try new things. Originally I was doing this with Ubuntu, but once I heard that Clear Linux had a new installer (via [Linux Unplugged](https://linuxunplugged.com/303)), and after playing with it as my main OS on my laptop, I decided to also try and migrate over my server from Ubuntu to Clear Linux. I'm also fairly average at linux so if there are any recommendations on how to make this easier please let me know!

**Hardware used**
Some leftover hardware from other projects:
-Intel G4560
-8gb of ram
-120gb ssd
-Some random compatible mini ITX board
-A cheap powersupply I got off of amazon
-An old case from a build from years ago

**Game Plan**
-Install Clear Linux Server
-Create two accounts, one with admin rights and another without that will be used to run factorio (separate accounts help with security)
-Install the necessary bundles
-Install factorio
-Install a community made factorio updater
-Make two bash scripts. One to start the factorio server and another to update factorio when needed

**Install Clear Linux**
Follow the guide [HERE](https://clearlinux.org/documentation/clear-linux/get-started/bare-metal-install-server) to install Clear Linux. If you want to set a static IP you can do it here. The only thing I'd recommend is to create your user accounts here as well. For this project, I have two. One named `kyle` with admin rights and another called `gamemaster` that does not have admin rights.

**Login as your admin account**
Once the system reboots I usually do everything via SSH from here on out. I logged in as my admin account (kyle).

**Install Factorio Headless**

First install `wget` by doing the following command: `sudo swupd bundle-add wget`

Next we are going to get the tar package from factorio's site [HERE](https://www.factorio.com/download-headless). I like to use the experimental version. Since we will be running this game under the gamemaster account, first get into gamemaster's home directory by typing: `cd /home/gamemaster`

Once there, run the following command to download the factorio software: `sudo wget -O factorio_headless.tar.gz https://www.factorio.com/get-download/0.17.45/headless/linux64`

I'm installing the latest experiemental version of factorio (as of the time of writing this guide). If you want to use any other version, make sure you follow the same syntax above and replace the link with the version you will be downloading.

Once downloaded, run this command to unzip the package: `sudo tar xf factorio_headless.tar.gz`. This will unpack everything into a newly made folder called factorio. Jump into that directory by typing: `cd factorio`

**Setup Factorio**
We are going to need to adjust some settings in the data folder so cd into that by typing: `cd data`. If you `ls` you can see we have a file called `server-settigns.example.json`, go ahead and make a new copy of it by typing: `sudo cp server-settings.example.json server-settings.json`. 

Next we are going to edit that newly copied file. I use vim as my text editor, so at this point I installed it by typing: `sudo swupd bundle-add vim` then I edited the file by typing: sudo vim server-settigns.json`

In this file, I only changed a few things. This is what I changed:
-*name*: The name of the server instance that your friends will search when looking for the game
-*description*: The description that will show up next to the name
-*username*: the username of your factorio account
-*token*: I use this instead of a plaintext password since it is more secure. You can find yours by logging into your factorio account and generating it from the profile page on their site. 
-*game_password*: the password used to enter the game
-*autosave_interval*: I set this to 60. Once you hit end game, saving takes a while so it's nice to not have your game paused for too long. 

Save this file and exit the text editor. 

Go ahead and go up a directory by typing: `cd ..`

Here is where I made two new directories. One called saves and the other called mods with the following command: `sudo mkdir -p saves mods`

The mods folder is where you'd put the .zip files of mods you can find online. Once the factorio server starts, it'll check that folder and if there are any mods inside it, add them automatically. I add mods here via sftp. In Clear Linux that is disabled by default, so follow the guide [HERE](https://clearlinux.org/documentation/clear-linux/reference/bundles/openssh-server#enable-sftp) if you want to enable sftp.

Before you can start a game, you need to create a save file. You can do this by typing: `sudo ./bin/x64/factorio --create saves/my_save.zip`

At this point you can test your server. Go ahead and find your IP by the following command: `ip a`. For this guide, my IP is 192.168.85.135 which is needed once the game is running. To run the server, type: `sudo ./bin/x64/factorio --start-server //home/gamemaster/factorio/saves/my_saves.zip` From here I can launch the Factorio game from my desktop, select Play, Connect to server, and type the IP of my server. If you set a password, here is where you'll also enter that. You should see that you are able to play the game now. Go ahead and exit.

**Setup Factorio Community Updater**
Since I'm using the experiemental version of factorio, the game updates frequently. Doing it manually is annoying. Luckily, there is a community made guide [HERE](https://github.com/narc0tiq/factorio-updater). This updater uses a non-standard library called `Requests`. If you install `sudo swupd bundle-add python-extras`, it contains Requests within the bundle by default (thanks @puneetse for this one).
 
We are going to use git to grab the project above. This needs to be installed by typing: `sudo swupd bundle-add git`. Once installed, go ahead and clone the project by typing: `sudo git clone https://github.com/narc0tiq/factorio-updater.git`. This will make a new folder in your factorio directory called `factorio-updater` that contains the python code that will update factorio for you. 

**Give gamemaster permissions to factorio**
At this point we have been using the admin account to set everything up and one of the reasons why we needed to sudo a lot in the previous steps. However, once we are finished we want the user gamemaster to run these commands. We also want this game to run even if we are not ssh'd into the machine. You can do that with tmux (`sudo swupd bundle-add tmux`). 

To give permission to gamemaster, type: `sudo chmown gamemaster:gamemaster /home/gamemaster -R`

Now lets swtich to the gamemaster account by typing: `su gamemaster` and typing the password you created when you installed the system. 

**Make bash scripts**
I always forget how to run the scripts to start the server and update the software, so I made two files called `start_factorio.sh` and `update_factorio.sh`

Inside start_factorio, I have the following line: `./factorio/bin/x64/factorio --start-server-load-latest  --server-settings ~/factorio/data/server-settings.json`

Inside update_factorio.sh, I have the following line: `python3 ~/factorio/factorio-updater/update_factorio.py -xDa ~/factorio/bin/x64/factorio` 

NOTE: If you do not want the experimental version of factorio, only use the flags `-Da` in that last command.

Once made, you need to set them to be executable by typing: `chmod +x start_factorio.sh update_factorio.sh`

**Open Factorio port on your router**

In your router settings, you need to open port 34197. I specifically made it so only the IP of my server has that port open, and not any other device on my network. As well, it JUST needs to be UDP 34197 that is open. 

**Run Factorio**
You can now type `./start_factorio.sh` and it will run. If you used ssh like me, you can use tmux to have it be persistent once you log out. If it ever needs to be updated, you can stop the game server, call your updater script and start up the game again by calling your start script. If you have mods, sometimes they'll need to be updated manually to have the game server start properly. Update them and try the start script again.

**That's it**
Once it is running, you should be able to search for it in the public games list by the name you gave your sever in the server-settings.json. If you don't see it, you either 1) need to update your factorio server since your client doesn't match versions or 2) need to verify your factorio server settings are properly set.

There are cleaner ways to do this, but this is the way I have my server setup. Please let me know if you have any questions or suggestions. Thanks!