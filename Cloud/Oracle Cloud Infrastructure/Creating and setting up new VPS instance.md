The entire process is described here:
https://bafista.ru/oracle-cloud-besplatnyj-virtualnyj-server-vps-i-probros-portov/

Free Oracle Cloud VPS setup playlist:
https://www.youtube.com/playlist?list=PLKxZjsTaY5Xu2Zfkgt0TV8eMEmJjentxx

After having provisioned a new VPS:
- Write down a server's public IP address
- Download keys to a local machine and change private key extention to .ppk
- Change access rights to the file: leave only current user as owner, remove all the others, grant current user full access, remove inheritance
- Try to connect to VPS from PowerShell console:
	`ssh -i "path/to/key.ppk" ubuntu@<vps ip address>`

### Installing basic set of useful packages

`sudo apt update`
`sudo apt upgrade`
`sudo apt install nano vim wget git tar unzip tmux ncdu ranger htop -y`

### Setting up swap file

Make sure a swap file is not set up (the command should produce no output):
	`sudo swapon -s`

Create the file /root/swapfile with a size of 2048 MB (2 GB).  The specified amount of disk space will be allocated without writing data to the file.
	`sudo fallocate -l 2048M /root/swapfile`

Make sure the swap file is successfully created:
	`sudo ls -lh /root/swapfile`

Set appropriate rights to the swap file (only root user is entitled to have rights to it):
	`sudo chmod 600 /root/swapfile`

Format the file as swap device and prepare it for use as a swap area:
	`sudo mkswap /root/swapfile`

	Output example:
```
		Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
		no label, UUID=12345678-1234-1234-1234-123456789abc
```

Activate the file as a swap device:
	`sudo swapon /root/swapfile`

Make sure a swap file is set up and active:
	`sudo swapon -s`
You can also use this command for this purpose:
	`free -h`

	Output example:
```
		              total        used        free      shared  buff/cache   available
Mem:           7.7G        2.1G        3.4G        200M        2.2G        5.1G
Swap:          2.0G          0B        2.0G
```


Make sure the swap function is automatically enabled on system restart:
	Open /etc/fstab file for edit:
		`sudo nano /etc/fstab`
	Insert this line at the end of the file:
		`/root/swapfile none swap sw 0 0`
	Reboot the system:
		`sudo reboot`
	Make sure a swap file is set up and active:
		`sudo swapon -s`

