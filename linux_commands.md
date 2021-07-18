# Linux commands
1. process:
- `ps aux | grep <command>`: show information of process with name <command>
- `htop`: show state of processes in system
2. network:
- `ss`: port and address a process is using
	- `-n`: list process using numeric address (ip instead of domain)
	- `-t`: list tcp connections
	- `-u`: list udp connections
	- `-a`: list all connections
	- `-p`: show the process using the socket
	- `-e`: show some extended information, like uid

3. tools
- `strace`: trace system calls