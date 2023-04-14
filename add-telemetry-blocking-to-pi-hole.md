Go to http://raspberry-pi.local/admin 

Then click on Adlists, and under Address add
```
https://raw.githubusercontent.com/MoralCode/pihole-antitelemetry/main/telemetry-domains.txt
```

Optional: Add a comment explaining what this list does.

Then go to Tools > Update Gravity and click on Update.

OR

SSH onto the raspberry-pi and run the below to update the adlists.
```sh
pihole -g
```