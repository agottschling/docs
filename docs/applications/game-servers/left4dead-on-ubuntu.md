---
author:
    name: Linode Community
    email: docs@linode.com
description: 'Install and configure a Left 4 Dead 2 multiplayer server on Ubuntu 14.04.'
keywords: 'left 4 dead,l4d2,game servers,games,ubuntu, ubuntu 14.04,steam'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
published: 'TBA'
modified: Wednesday, December 9th, 2015
modified_by:
    name: Linode
title: 'Install and Configure a Left 4 Dead 2 Multiplayer Server on Ubuntu 14.04'
contributor:
    name: Andrew Gottschling
    link: https://github.com/agottschling
---
Left 4 Dead 2 is a single player game developed and published by Valve Inc. Aside from it's fantastic singleplayer mode, Left 4 Dead 2 also offers a great multiplayer mode so you can blast zombies with your friends. This guide will explain how to prepare your VPS, install SteamCMD, and install, then configure, Left 4 Dead 2.

## Prerequisites

Have the following items before you begin:

- A [Steam](http://store.steampowered.com) account.
- A copy of [Left 4 Dead 2](http://store.steampowered.com/app/550/) that you have purchased on Steam.
- A Linode with at least 2GB of RAM and 15GB of free disk space.
- An up-to-date Linode running Ubuntu 14.04. We suggest you follow our [Getting Started](/docs/getting-started) guide for help configuring your Linode.

{: .note }
>This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you’re not familiar with the sudo command, reference the [Users and Groups](/docs/tools-reference/linux-users-and-groups) guide.

## Preparing your Linode

Left 4 Dead 2 is sold on Steam. Therefore, we will use SteamCMD to download and maintain servers for games sold on steam.

Because current generation Linodes run a 64-bit operating system, we need to download a few extra libraries in order to run SteamCMD.

1.  Configure the package manager to include packages for i386 architecture:

        sudo dpkg --add-architecture i386

2.  Update the system:

        sudo apt-get update && sudo apt-get upgrade

3.  Install the 32-bit libraries required:

        sudo apt-get install lib32gcc1 lib32stdc++6 libc6-i386 libcurl4-gnutls-dev:i386 screen

    {: .note }
    > If you're running a legacy Linode on a 32 bit kernel, install these packages instead:
    >
    >     sudo apt-get install libcurl4-gnutls-dev:i386 libc6-i386 libgcc1 screen

If you have a firewall running on your Linode, add exceptions for SteamCMD:

    sudo iptables -A INPUT -p udp- m udp --sport 4380 --dport 1025:65355 -j ACCEPT
    sudo iptables -A INPUT -p udp -m udp --sport 10999 --dport 1025:65355 -j ACCEPT
    sudo iptables -A INPUT -p udp -m udp --sport 7777 --dport 1025:65355 -j ACCEPT
	sudo iptables -A INPUT -p udp -m udp --sport 27015 --dport 1025:65355 -j ACCEPT

{: .note }
> If you've configured your firewall according to our [Securing Your Server](/docs/security/securing-your-server) guide, be sure to add these port ranges to your `/etc/iptables.firewall.rules` file.

## Install SteamCMD and Left 4 Dead 2

1.  From your user's home folder, download SteamCMD into its own directory:

        mkdir steamcmd
        cd steamcmd
        wget http://media.steampowered.com/installer/steamcmd_linux.tar.gz

3.  Extract the package and remove the archive file:

        tar -xvzf steamcmd_linux.tar.gz
        rm steamcmd_linux.tar.gz

4.  Run the SteamCMD Installer.

        ./steamcmd.sh

    This command will display output similar to this:

        Redirecting stderr to '/home/linode/Steam/logs/stderr.txt'
        [  0%] Checking for available updates...
        [----] Downloading update (0 of 7,013 KB)...
        [  0%] Downloading update (1,300 of 7,013 KB)...
        [ 18%] Downloading update (3,412 of 7,013 KB)...
        [ 48%] Downloading update (5,131 of 7,013 KB)...
        [ 73%] Downloading update (6,397 of 7,013 KB)...
        [ 91%] Downloading update (7,013 of 7,013 KB)...
        [100%] Download complete.
        [----] Installing update...
        [----] Extracting package...
        [----] Extracting package...
        [----] Extracting package...
        [----] Installing update...
        [----] Installing update...
        [----] Installing update...
        [----] Cleaning up...
        [----] Update complete, launching Steam...
        Redirecting stderr to '/home/linode/Steam/logs/stderr.txt'
        [  0%] Checking for available updates...
        [----] Verifying installation...
        Steam Console Client (c) Valve Corporation
        -- type 'quit' to exit --
        Loading Steam API...OK.

        Steam>

    The `Steam>` prompt is similar to the linux command prompt, with the exception of not being able to execute normal linux commands. 

4.  Install Left 4 Dead 2 from the SteamCMD prompt:

        login anonymous
        force_install_dir ../L4D2-server
        app_update 222860 validate

    This can take some time. If the download looks like it has frozen, be patient; it may take a long time. Once the download is complete, you should see this output:

        Success! App '222860' fully installed.

        Steam>

5.  Finally, exit SteamCMD.

        quit

##Configuring Left 4 Dead 2

1.  Before you configure the server, you should download a example config file:

        cd ~/L4D2-server/left4dead2/cfg
		
	Choose one of the following example files:
	    * wget https://www.gottnt.com/l4d2/basic-server.cfg
	    * wget https://www.gottnt.com/l4d2/detailed-server.cfg
		
    Make sure to rename the file to `server.cfg` before launching the server:
	
2.  Open the configuration file with `nano` to edit the configuration. Most server options are explained in the configuration file. Simply follow the instructions:

        nano server.cfg

3.  When you are finished, exit `nano` and save your changes.

4.  Next, it is a good idea to write a custom startup script that will execute your custom config files. 
    {: .file }
    ~/L4D2-server/start_L4D2.sh
    :   ~~~
        screen srcds -console -game left4dead2 +port 27020 +maxplayers 8 +exec server.cfg +map c2m1_highway
        ~~~

	This script, when run, will execute the Don't Starve Together server in a [Screen](/docs/networking/ssh/using-gnu-screen-to-manage-persistent-terminal-sessions) session.
	
5.  Make the script executable:

        chmod +x ~/dstserver/bin/start_dst.sh

##Using the Server

1.  To start the server, simply run the executable. 
        ./start-L4D2
        
2.  To detach from the screen session running the server console, press these two key combinations in succession:

    **CONTROL + A**<br>
    **CONTROL + D**

3.  To bring the console back, type the following command:

        screen -r

4.  To stop the server, bring back the console and type `exit`.

## Entering The Server
In order to connect to the server, a lobby needs to be created:
1.  Choose a game mode you would like to play. Click on the icon that coresponds to the game type you want. Select the **Play With Friends** option.

2.  On the next window, click **Create Versus Lobby**. In the next screen, select a campaign you want. For the **Server Type** select **Best Available Dedicated**, then click **Create Lobby**. After you click that the lobby will be created. 

At this point you will need to enter one of the commands to make your lobby connect to your dedicated server.

Make sure you have the developer console enabled - Under **Options--Keyboard/Mouse** make sure that **Allow Developer Console** is checked.

3. Press the `~` key to open the developer console and enter the following commands, making sure to replace 123.456.789.012 with your Linode's external or public IP address:
        
		mm_dedicated_force_servers 123.456.789.012
	
4. Finally, invite friends to the game using the Steam Overlay (`SHIFT + TAB`) and then begin the game. 