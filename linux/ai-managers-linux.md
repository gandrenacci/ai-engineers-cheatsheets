# Linux Cheatsheet for AI Infrastructure Managers

**Compact Linux cheatsheet tailored** for  **AI / Solutions / Infrastructure-oriented manager with strong hands-on activity** on:

* Linux servers
* Docker / Podman
* logs and troubleshooting
* permissions
* networking
* AI model serving
* config and process checks

## 1) Navigation and file system

Commands you use constantly to move around servers and inspect folders.

```bash
pwd
```

Show current directory.

```bash
ls
ls -l
ls -la
ls -lh
```

List files.
`-l` detailed view, `-a` hidden files, `-h` human sizes.

```bash
cd /path
cd ..
cd ~
cd -
```

Move between directories.
`cd -` jumps to previous path.

```bash
tree
tree -L 2
```

Show folder structure. Very useful for projects and repos.

```bash
mkdir mydir
mkdir -p a/b/c
```

Create folders.
`-p` creates full path if missing.

---

## 2) File operations

For copying configs, moving models, renaming folders, cleaning files.

```bash
cp file1 file2
cp -r src_dir dest_dir
```

Copy file or directory.

```bash
mv old new
```

Move or rename.

```bash
rm file
rm -r folder
rm -rf folder
```

Delete.
`-rf` is dangerous: recursive + force.

```bash
touch file.txt
```

Create empty file or update timestamp.

```bash
stat file.txt
```

Show detailed metadata.

```bash
du -sh *
du -sh /var/home/root/llm-models
```

See folder sizes.

```bash
df -h
```

Disk free space.

---

## 3) Read files fast

Perfect for configs, logs, scripts, YAML, service files.

```bash
cat file
less file
head file
tail file
tail -f logfile.log
```

`tail -f` follows logs live. Very useful for containers and services. 

```bash
nl -ba file
```

Show file with line numbers.

```bash
sed -n '1,120p' file
```

Print specific line range.

```bash
grep "text" file
grep -i "error" file
grep -r "vllm" .
```

Search text.
`-i` ignore case, `-r` recursive. 

```bash
find . -name "*.log"
find . -type f -name "*.yaml"
```

Find files by name/type. 

---

## 4) Permissions and ownership

Critical when working with scripts, model folders, containers, mounted volumes.

```bash
ls -l
```

Check permissions and owner.

```bash
chmod 755 script.sh
chmod 644 config.yaml
chmod +x script.sh
```

Change permissions.
`755` executable script, `644` normal config file. 

```bash
chown user:user file
chown -R user:user folder
```

Change owner.

```bash
id
whoami
groups
```

See current user and group membership.

---

## 5) Editing files

Fast editing on servers.

```bash
vim file
```

Essential Vim:

```bash
i
Esc
:w
:q
:wq
dd
yy
p
```

Insert, save, quit, delete line, copy line, paste. 

If available:

```bash
nano file
```

Simpler editor.

---

## 6) Piping and redirection

This is where Linux becomes powerful for diagnostics and automation.

```bash
command > out.txt
command >> out.txt
command 2> err.txt
command > all.txt 2>&1
```

Redirect output / errors. 

```bash
ls -l | grep ".yaml"
```

Pipe one command into another.

```bash
ps aux | grep vllm
```

Find process.

```bash
find . -name "*.log" | xargs grep -i "error"
```

Search errors across many logs.

---

## 7) Processes and system monitoring

Useful for debugging model serving, containers, API processes, scripts.

```bash
ps
ps -ef
ps aux
```

List processes. 

```bash
top
htop
```

Live CPU/RAM view.
`htop` is nicer if installed.

```bash
kill PID
kill -9 PID
pkill -f vllm
```

Stop process.
`-9` forces termination.

```bash
pgrep -af python
pgrep -af podman
```

Find process by name with full command.

```bash
free -h
uptime
```

Memory and load.

```bash
uname -a
hostnamectl
```

System/kernel info. 

---

## 8) Journals and services (very important)

For modern Linux and RHEL-style systems.

```bash
systemctl status service-name
systemctl start service-name
systemctl stop service-name
systemctl restart service-name
systemctl enable service-name
systemctl disable service-name
```

Manage services.

```bash
journalctl -u service-name
journalctl -u service-name -f
journalctl -xe
```

Read service logs.
`-f` follows live logs.

Examples:

```bash
systemctl status rhaiis
journalctl -u rhaiis -f
```

---

## 9) Networking

For testing endpoints, servers, ports, APIs, model services.

```bash
ping 8.8.8.8
ping -c 4 google.com
```

Connectivity test. 

```bash
curl http://localhost:8000
curl -I https://example.com
curl -X GET http://host:port/v1/models
```

HTTP test.
Excellent for OpenAI-compatible endpoints. 

```bash
ss -tulpn
netstat -tuln
```

See listening ports.
`ss` is preferred on newer systems. 

```bash
ip a
ip route
hostname -I
```

Check IP addresses and routing.

```bash
nslookup host
dig host
```

DNS checks.

```bash
nc -zv host 8000
```

Test whether port is open.

---

## 10) SSH and remote work

Essential for servers, clusters, remote AI environments.

```bash
ssh user@server
```

Connect to server. 

```bash
ssh-keygen
ssh-copy-id user@server
```

Create and install SSH key. 

```bash
scp file user@server:/path
scp -r folder user@server:/path
```

Copy files remotely. 

```bash
rsync -avz folder/ user@server:/path/
```

Better sync than `scp`, especially for large folders.

---

## 11) Package management

Depends on distro, but these are the common ones.

### Debian/Ubuntu

```bash
apt update
apt install pkg
apt remove pkg
apt upgrade
```

### RHEL / Rocky / Alma / Fedora

```bash
dnf install pkg
dnf remove pkg
dnf update
rpm -qa | grep pkg
```

The uploaded cheatsheet includes the `apt` family; I’m extending this with `dnf` because it fits your RHEL-oriented work better. 

---

## 12) Root and privilege escalation

```bash
sudo command
sudo -i
sudo su
passwd
```

Run admin commands / switch to root / change password. 

Useful checks:

```bash
sudo -l
```

See what sudo permissions you have.

---

## 13) Archives and transfers

Helpful for logs, backups, model packages, deployments.

```bash
tar -cvf archive.tar folder/
tar -xvf archive.tar
tar -czvf archive.tar.gz folder/
tar -xzvf archive.tar.gz
```

```bash
zip -r archive.zip folder/
unzip archive.zip
```

---

## 14) Environment variables and shell

Very useful for APIs, tokens, paths, deployments.

```bash
echo $HOME
echo $PATH
env
printenv
```

Show environment variables.

```bash
export VAR=value
```

Set variable for current session.

```bash
source ~/.bashrc
source ~/.zshrc
```

Reload shell config.

```bash
which python
which podman
whereis python
```

Find executable location.

---

## 15) Command history and speed

For productivity.

```bash
history
```

Show previous commands. 

```bash
!!
```

Repeat last command. 

```bash
Ctrl + R
```

Search command history interactively. 

```bash
clear
```

Clean terminal screen.

---

# Tailored section for your real activity

## 16) Logs and troubleshooting

Very relevant for AI services, APIs, containers, infra issues.

```bash
tail -f /var/log/messages
journalctl -f
journalctl -u service-name -f
```

Follow logs live.

```bash
grep -i error logfile.log
grep -Ri "cuda" .
grep -Ri "permission denied" .
```

Search errors quickly.

```bash
find /var/log -name "*.log" | xargs grep -i "failed"
```

Scan many logs.

```bash
dmesg | tail -50
```

Kernel messages, useful for driver/hardware issues.

---

## 17) Podman / Docker essentials

You are working a lot with containers and AI serving, so this matters a lot.

### Podman

```bash
podman ps
podman ps -a
podman images
podman logs container-name
podman logs -f container-name
podman exec -it container-name bash
podman stop container-name
podman start container-name
podman restart container-name
podman rm container-name
podman rmi image-name
```

### Docker

```bash
docker ps
docker ps -a
docker images
docker logs container-name
docker logs -f container-name
docker exec -it container-name bash
docker stop container-name
docker start container-name
docker restart container-name
docker rm container-name
docker rmi image-name
```

Inspect config:

```bash
podman inspect container-name
docker inspect container-name
```

Check mounted volumes:

```bash
podman inspect container-name | grep -i mount -A 20
```

---

## 18) AI / model serving checks

For vLLM, inference endpoints, GPUs, model folders.

```bash
curl http://localhost:8000/health
curl http://localhost:8000/v1/models
```

Check endpoint health / model exposure.

```bash
ls -lh /path/to/models
du -sh /path/to/models/*
```

Inspect model folders and sizes.

```bash
nvidia-smi
watch -n 1 nvidia-smi
```

GPU usage live.

```bash
env | grep -i cuda
env | grep -i hf
```

Check CUDA / Hugging Face environment variables.

```bash
ps aux | grep vllm
ss -tulpn | grep 8000
```

Confirm process and exposed port.

---

## 19) Useful one-liners for your day-to-day

### Find big files

```bash
find /var/home/root -type f -size +1G -exec ls -lh {} \;
```

### Find recent files

```bash
find . -type f -mtime -2
```

Files modified in last 2 days.

### Search config values

```bash
grep -R "model" .
grep -R "port" .
grep -R "gpu-memory-utilization" .
```

### See top memory consumers

```bash
ps aux --sort=-%mem | head
```

### See top CPU consumers

```bash
ps aux --sort=-%cpu | head
```

### Check open ports

```bash
ss -tulpn
```

### Check whether a path exists before acting

```bash
[ -d /path ] && echo "exists"
[ -f file.txt ] && echo "exists"
```

### Follow only error lines

```bash
tail -f app.log | grep --line-buffered -i error
```

---

## 20) Safer habits

Compact reminders that save you from trouble.

```bash
rm -rf *
```

Never run without checking where you are first.

```bash
pwd && ls -la
```

Good habit before delete/move.

```bash
cp file file.bak
```

Create quick backup before editing.

```bash
history | tail -20
```

Useful to reconstruct what you just did.

```bash
sudo !!
```

Repeat last command with sudo.

---

# Mini survival set

If you had to keep only the most useful commands for your job:

```bash
pwd
ls -lah
cd -
find . -name "*.log"
grep -Ri "error" .
tail -f logfile.log
ps aux | grep name
top
free -h
df -h
ss -tulpn
curl http://host:port
ssh user@host
scp file user@host:/path
chmod +x script.sh
chown -R user:user folder
systemctl status service
journalctl -u service -f
podman ps
podman logs -f container
nvidia-smi
```

---

# Very short mental model

Use this logic:

* **Where am I?** → `pwd`, `ls -lah`
* **Where is the file?** → `find`, `grep -R`
* **What is happening?** → `ps`, `top`, `journalctl`, `tail -f`
* **Is the service alive?** → `systemctl status`, `curl`, `ss -tulpn`
* **Why is it failing?** → logs + permissions + port + process
* **Is the container okay?** → `podman ps`, `podman logs -f`, `inspect`
* **Is the GPU okay?** → `nvidia-smi`