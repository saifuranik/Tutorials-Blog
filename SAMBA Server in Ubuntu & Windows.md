# SAMBA Server in Ubuntu & Windows

for making connection of samba at first we have to create an ubuntu machine in VMware and if you wish you can also make a window server in vm or you can use your existing window machine

Just follow the commands but Remember there some commands you need to modify as your info so while copy and paste command please don’t rush

read whole command understand why you going to use it then apply it will be easy for you in next time

SAMBA setup in ubuntu and window
# Step 1 : update Ubuntu machine
```sh
apt update -y
```
# Step 2: install SAMBA package
```sh
apt install samba samba-client samba-common -y
```
# Step 3: Enable samba in Firewall
```sh
 firewall-cmd --permanent --zone=public --add-service=samba
 
 firewall-cmd --reload 
 
 sudo ufw allow samba
```
# Step 4: Creating a Directory for samba that you wanna share ( Linux )
```sh
 mkdir <folder name >
```
# Step 4: Opening samba config file in terminal
```sh
sudo vim /etc/samba/smb.conf
```
# Step 5: Make Configuration permission for created folder for share
add this spinet at the end of the file
```sh
[<shared folder name >]
   path = /home/anik/<shared folder name >
   available = yes
   valid users = anik      #your user name
   read only = no
   browsable = yes
   public = yes
   writable = yes
   create mask = 0775
   directory mask = 0775
   force user = anik       #your user name
   force group = anik       #your Group name


[global]
    workgroup = WORKGROUP
    server string = Samba Server
    security = user
    map to guest = bad user
    dns proxy = no

[public]
    path = /srv/samba/public
    browsable = yes
    writable = yes
    guest ok = yes
    read only = no
```
```
sudo mkdir -p /srv/samba/public
sudo chown -R nobody:nogroup /srv/samba/public
sudo chmod -R 0775 /srv/samba/public
```
# Step 6 : re-Start the service
```
sudo service smbd restart
sudo systemctl restart smbd
sudo systemctl restart nmbd

sudo systemctl status smbd
sudo systemctl status nmbd

sudo systemctl enable smbd
sudo systemctl enable nmbd

sudo ufw allow Samba

```

# Step 7: set permission for create fiels from windows
```
sudo chown -R <username>:<username> /home/anik/<shared file name>
sudo chmod -R 775 /home/<username>/<shared file name>
```
# Step 8: Setting up a user account for Samba
Since Samba doesn’t use the system account password, we need to set up a Samba password for our user account.
The username used must belong to a system account, else it won’t save.
Setting up User Accounts and Connecting to Share
```
sudo smbpasswd -a <username>
```
# Step 9: Find your local IP
```
ip a
```
example :( 192.168.0.108/24 )

# Step 10: Samba status
```
sudo systemctl status smbd
```
* create some file in the folder what you going to share
# Step 11: Go to your window server and type your ip address in window search bar with folder
```
\\<ubuntu ip address>\< shared folder name>
```
# \\192.168.0.108\sambashare

** Congratulations you are done SAMBA server Linux to Window ***
Next Post will be SAMBA server LINUX to LINUX




