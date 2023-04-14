Add this to the blocklists
```sh
https://raw.githubusercontent.com/kboghdady/youTube_ads_4_pi-hole/master/youtubelist.txt
```

Add this as well
```sh
https://raw.githubusercontent.com/kboghdady/youTube_ads_4_pi-hole/master/crowed_list.txt
```

Clone the repo
```sh
git clone https://github.com/kboghdady/youTube_ads_4_pi-hole.git ~/pi-hole/youtube && cd $_
```

Update the repo dir in the script
```sh
vim youtube.sh
repoDir='/home/tom.ferguson/pi-hole/youtube'
```

Make the script executable
```sh
sudo chmod a+x youtube.sh
```

Create a Cronjob to run the script
```sh
sudo crontab -e
0 */1 * * * sudo /home/tom.ferguson/pi-hole/youtube.sh >/dev/null 
```

Install sqlite3
```sh
sudo apt install sqlite3
```

Run the script manually (will take a while)
```sh
sudo ./youtube.sh
```

To delete blacklists from your database run
```sh
sudo sqlite3 /etc/pihole/gravity.db "delete from domainlist where domain like '%googlevideo.com%' "
```