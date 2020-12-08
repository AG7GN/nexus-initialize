# Preparing a New Nexus Image

Many of these OS preparation tips are from David Ranch's excellent [HOWTO](http://www.trinityos.com/HAM/CentosDigitalModes/RPi/rpi4-setup.html).

1) Download the "Raspbian Pi OS with desktop" [image](https://www.raspberrypi.org/software/operating-systems/) and burn it to an SD card.

1) Boot the SD card in the Pi.

1) Follow prompts and provide the requested settings.  
	
1) Reboot when prompted.

1) Run __Raspberry > Preferences > Raspberry Pi Configuration__
	- Click __Change Password...__ and set pasword to `changeme`.
	- Change hostname to 'nexus'
	- In __Interfaces__ tab, enable SSH, VNC, SPI, I2C, Serial Port

1) Reboot

1) Change password to `changeme` if prompted

1) Disable NFS clients

		sudo systemctl disable nfs-client.target

1) Bring it up to date

		sudo apt-get clean
		sudo apt-get update
		sudo apt-get autoremove
		sudo apt-get upgrade	
		sudo apt-get dist-upgrade

1) Check eeprom state

		sudo rpi-eeprom-update
		
	If this returns something other than __up-to-date__ for BOOTLOADER and VL805, then run this command followed by a reboot:
	
		sudo rpi-eeprom-update -a
		
1) Get script to remove old kernels

		cd /tmp
		wget http://www.trinityos.com/HAM/CentosDigitalModes/RPi/usr/local/sbin/remove-old-kernels.sh
		sudo mv /tmp/remove-old-kernels.sh /usr/local/sbin/
		sudo chmod 700 /usr/local/sbin/remove-old-kernels.sh
		sudo /usr/local/sbin/remove-old-kernels.sh

1) Install persistent iptables:

		sudo apt-get install iptables-persistent
		
1) Edit `/etc/apt/sources.list` and enable `deb-src`:	

	- Uncomment the line starting with `#deb-src` so it looks like:
	
			deb-src http://raspbian.raspberrypi.org/raspbian/ buster main contrib non-free rpi
	- Save and close the file
			
1) Install the toolchains and packages:

		sudo apt-get install vim tcpdump lsof gpm telnet minicom links exfat-utils \
		yad dosfstools xscreensaver build-essential autoconf automake libtool cmake \
		extra-xdg-menus bc dnsutils libgtk-3-bin jq xdotool moreutils build-essential \
		exfat-utils aptitude
		sudo apt update

1) Create `~\.vimrc` as follows:

		cd ~
		cat > .vimrc <<EOF
		" An example for a vimrc file.
		"
		" Maintainer:   Bram Moolenaar <Bram@vim.org>
		" Last change:  2008 Jul 02
		"
		" To use it, copy it to
		"     for Unix and OS/2:  ~/.vimrc
		"             for Amiga:  s:.vimrc
		"  for MS-DOS and Win32:  $VIM\_vimrc
		"           for OpenVMS:  sys$login:.vimrc

		" When started as "evim", evim.vim will already have done these settings.
		if v:progname =~? "evim"
		  finish
		endif

		" Use Vim settings, rather then Vi settings (much better!).
		" This must be first, because it changes other options as a side effect.
		set nocompatible
		set noedcompatible " turn off wierd :s///g behaviour (g always means g)
		set background=dark " use light colors for a dark background (def: light)

		set tabstop=3

		" allow backspacing over everything in insert mode
		set backspace=indent,eol,start

		if has("vms")
		  set nobackup          " do not keep a backup file, use versions instead
		else
		  set backup            " keep a backup file
		endif
		set history=50          " keep 50 lines of command line history
		set ruler               " show the cursor position all the time
		set showcmd             " display incomplete commands
		set incsearch           " do incremental searching

		set nomodeline
      set mouse=r
      set paste

		" For Win32 GUI: remove 't' flag from 'guioptions': no tearoff menu entries
		" let &guioptions = substitute(&guioptions, "t", "", "g")

		" Don't use Ex mode, use Q for formatting
		map Q gq

		" CTRL-U in insert mode deletes a lot.  Use CTRL-G u to first break undo,
		" so that you can undo CTRL-U after inserting a line break.
		inoremap <C-U> <C-G>u<C-U>

		" In many terminal emulators the mouse works just fine, thus enable it.
		"if has('mouse')
		"  set mouse=a
		"endif

		" Switch syntax highlighting on, when the terminal has colors
		" Also switch on highlighting the last used search pattern.
		if &t_Co > 2 || has("gui_running")
		  syntax on
		  set hlsearch
		endif

		" Only do this part when compiled with support for autocommands.
		if has("autocmd")

		  " Enable file type detection.
		  " Use the default filetype settings, so that mail gets 'tw' set to 72,
		  " 'cindent' is on in C files, etc.
		  " Also load indent files, to automatically do language-dependent indenting.
		  filetype plugin indent on

		  " Put these in an autocmd group, so that we can delete them easily.
		  augroup vimrcEx
		  au!

		  " For all text files set 'textwidth' to 78 characters.
		  autocmd FileType text setlocal textwidth=78

		  " When editing a file, always jump to the last known cursor position.
		  " Don't do it when the position is invalid or when inside an event handler
		  " (happens when dropping a file on gvim).
		  " Also don't do it when the mark is in the first line, that is the default
		  " position when opening a file.
		  autocmd BufReadPost *
			 \ if line("'\"") > 1 && line("'\"") <= line("$") |
			 \   exe "normal! g`\"" |
			 \ endif

		  augroup END

		else

		  set autoindent                " always set autoindenting on

		endif " has("autocmd")

		" Convenient command to see the difference between the current buffer and the
		" file it was loaded from, thus the changes you made.
		" Only define it when not defined already.
		if !exists(":DiffOrig")
		  command DiffOrig vert new | set bt=nofile | r # | 0d_ | diffthis
								\ | wincmd p | diffthis
		endif

		" use :Swrite instead of :w to save changes to a file that only root can
		" modify.
		if has("unix")
				  command -nargs=? Swrite :w !sudo tee %
		endif
		:color desert
		EOF
	
1) Set `vim` or `nano` as system default text editor:

		sudo update-alternatives --set editor /usr/bin/vim.basic
		
	or, run `select-editor` to change it to nano.
		
1) Add these 2 lines to `/etc/fstab`:

		tmpfs           /tmp            tmpfs   defaults,noatime,mode=1777,size=50m   0   0
		tmpfs           /var/log        tmpfs   defaults,noatime,mode=0755,size=50m  0   0
		
	Save and reboot.
	
1) Install USB drive power management

		sudo apt-get install hdparm
	
1) Disable the special key keyboard mapping tool

		sudo update-rc.d -f triggerhappy remove
		sudo systemctl disable triggerhappy.service
		sudo systemctl disable triggerhappy.socket
		
1) `/etc/rsyslog.conf` changes:

	- Delete this stanza:

			*.=debug;\
				auth,authpriv.none;\
				news.none;mail.none     -/var/log/debug
			
	- Find this line:
	
			*.*;auth,authpriv.none        -/var/log/syslog
		and change it to:
			
			*.*;auth,authpriv,mail.none        -/var/log/syslog
	- Delete these lines:

            mail.info                     -/var/log/mail.info
            mail.warn                     -/var/log/mail.warn
            
   - Change the line:
   
   			mail.err                      /var/log/mail.err
		to:
   			
   			mail.warn                     /var/log/mail.err
   			
   - Find the line:
   
   			kern.*                        -/var/log/kern.log
   			
		and insert a new line with this text right after:	

			kern.debug                    stop
			
	- Save these changes then restart rsyslog:
	
			sudo systemctl restart rsyslog
			
	- Remove the old debug logs:
	
			sudo rm -f /var/log/debug*
			
1) Add ULOG for iptables logging to separate file

	- Install ulogd2:
	
			sudo apt-get install ulogd2
			
	- Edit `/etc/ulogd.conf` and modify the following stanzas to look like this:
	
			[log1]
			group=0

			[log2]
     		group=1

     		[emu1]
     		file="/var/log/ulogd_traffic-emu1.log"
     		sync=1

     		[emu2]
     		file="/var/log/ulogd_traffic-emu2.log"
     		sync=1

	- Restart ulogd:
	
			sudo systemctl restart ulogd

1) Modify log management

	- Open `/etc/logrotate.conf` and change:
		
			#compress
		to:
		
			compress

	- Add these lines immediately after `compress`:
	
			# use bzip2 with higher compression than gzip
			compresscmd /bin/bzip2
			uncompresscmd /bin/bunzip2
			compressoptions -9
			compressext .bz2
	- Save and close the file.
	
	- Edit `/etc/logrotate.d/rsyslog` and comment out all `delaycompress` lines.
	
	- Add these lines to the TOP of the file, not part of any stanza:
	
			rotate 3
			daily
			missingok
			notifempty
			compress
			compresscmd /bin/bzip2
			uncompresscmd /bin/bunzip2
			compressoptions -9
			compressext .bz2
	- Save and close the file.
	
	- Edit `/etc/logrotate.d/ulogd2` and add the following stanza:
	
			/var/log/ulogd_traffic-emu1.log
			{
         		rotate 4
            	weekly
            	missingok
            	notifempty
            	size 10M
            	compress
	    			compresscmd /bin/bzip2
	    			uncompresscmd /bin/bunzip2
	    			compressoptions -9
	    			compressext .bz2
            	sharedscripts
            	create 640 ulog adm
            	postrotate
               	invoke-rc.d ulogd2 reload > /dev/null
            	endscript
			}

1) Enable Hardware Watchdog

	- Install `watchdog`:
	
			sudo apt-get install watchdog
		
		
	- Edit `/etc/watchdog.conf` and uncomment these lines:
	
			max-load-1             = 24
			max-load-5             = 18
			watchdog-device        = /dev/watchdog

	- Enable and start `watchdog`:
	
			sudo systemctl enable watchdog
			sudo systemctl start watchdog.service

1) Changes to `/boot/config.txt`

	- Uncomment this line:
	
			hdmi_force_hotplug=1
	
	- Add these lines to enable Fe-Pi and RTC:
	
			# Enable Fe Pi audio card
			dtoverlay=fe-pi-audio
			# Enable ds3231 Real Time Clock (RTC)
			dtoverlay=i2c-rtc,ds3231

1) Set resolution

	- Run `sudo raspi-config`
	- Select __1 Display Options__, then __D1 Resolution__, then select __DMT Mode 82__ from the list.
	- Tab to OK, then exit `raspi-config`.
	

1) Reboot

1) Install `nexus-initialize` files and scripts and run initializer

		sudo mkdir -p /usr/local/src/nexus
		sudo chown $USER:$USER /usr/local/src/nexus
		cd /usr/local/src/nexus
		git clone https://github.com/AG7GN/nexus-initialize
		nexus-initialize/nexus-install
	
	The `nexus-install` command will delete `$HOME/DO_NOT_DELETE_THIS_FILE_`, which will cause initialize-pi.sh to run at next boot. `initialize-pi.sh` will clear out user's home folder and other personal data from some apps.
	
1) Click __Raspberry > Preferences > Main Menu editor__ and arrange menu to suit.

1) Try connecting to the Pi from a VNC client

	- Should be able to go to `nexus.local`
	
	- Once connected, run __Raspberry > Preferences > Screensaver__, then select __Disable Screen Saver__ in the dropdown.
	- __File > Quit__ to exit __Screensaver__
	
1) Set resolution from VNC

	- Run __Raspberry > Preferences > Screen Configuration__. 
	- Select __Configure > Screens > HDMI-1 > Resolution > 1920 x 1080.
	- Click OK to accept setting.
	- __File > Quit__ to exit __Screen Configuration__
	
1) Reboot and test VNC again.

1) Set version (update date as needed)

		echo "NEXUS_VERSION=20201209" | sudo tee /boot/nexus.txt 1>/dev/null
		
1) Remove `$HOME/DO_NOT_DELETE_THIS_FILE_` and shutdown (don't reboot).

Image is now ready for distribution
