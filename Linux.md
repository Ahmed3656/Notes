- first
- Directories (file system)
	/home     -> contains home directories for all **non-root** users
	/root        -> contains the root user home directory
	/bin          -> contains executables for all essential **user** binaries (commands)
	/sbin        -> contains executables for all essential **system** binaries (commands). Only users with **superuser** privilege are allowed to  run/execute these programs
	/lib           -> contains essential shared libraries which are executed from /bin and /sbin
	/usr          -> contains user home directories
	/usr/local -> contains programs that you install on the computer (third party applications)
	/opt         -> contains optional third party applications
	/boot       -> contains files required for booting
	/etc          -> contains configurations for system-wide applications
	/dev         -> contains devices files (hardware)
	/var          -> variable data that changes frequently during system operation like logs, temp, cache data, runtime data, databases, etc.
	/tmp        -> contains temporary files
- Commands
	user@(server/machine/host name):(directory where "~" is home)(privilege where "$" is regular user and "#" is the root user)
	
	-uname -a: displays the operating system and the kernel information
	-pwd: stands for "print working directory" where it prints where you are currently standing
	-ls: list folders and files found in the current directory
	-ls -l: lists everything in the current directory as a list with extra information
	-ls -la: lists everything in the current directory as a list with extra information and includes hidden files/directories
	-lscpu: displays the CPU's information
	-lsmem: displays the memory's information
	-cat: displays file contents
	-clear: clears the current terminal
	-history: displays all of the commands used previously. It also accepts a number argument if you want a specific amount of commands to only appear
	-cd: stands for "change directory" and transfers you to the destined directory
	-mkdir: stands for "make directory" where it creates a directory
	-touch: creates a file in the current directory
	-rm: deletes the selected file
	-rm -r: deletes the selected directory even if it isn't empty where "-r" stands for recursively remove
	-rm name*: deletes all the files that start with "name"
	-rm -r name*: deletes all the directories recursively that start with "name"
	-mv: renames the selected file/directory
	-cp: copies the selected file to a new file u create/select
	-cp -r: copies the selected directory recursively
	-adduser: adds a new user
	-apt: is the package manager command which allows you to manage packages
		install
		remove
		update
		list
		list --upgradable
	-snap: is another package manager which allows you to manage packages
	-sudo: grants superuser privilege to run commands like adduser, apt, and snap
	-su - user: login as the specified user
- Package Manager
	In ubuntu the package manager included is APT(Advanced Package Tool) which is the traditional package manager.
	APT installs .deb packages from official repositories.
	It relies on system libraries and shared dependencies.
	APT requires manual updates.
	APT is faster as it integrates packages directly with the system.
	APT stores packages in /usr/bin/, /usr/lib/, etc.
	
	Snap(Snappy) is another package manager which is a universal package manager.
	Snap installs self-contained packages from the Snap Store.
	It bundles all required dependencies within the package.
	Snap updates packages automatically in the background.
	Snap is slower as it runs applications in a sandbox.
	Snap stores packages in /var/lib/snapd/ and /snap/.

	Summary:
		APT is best for **lightweight, system-integrated applications**.
		Snap is useful for **cross-distro compatibility and the latest software versions**.
- Vim
	
- last