# Making a container to run mdbook

1. install mdbook using ```apt install mdbook```

2. Follow the mdbook documentation [here](https://rust-lang.github.io/mdBook/index.html) to create an mdbook
	- My mdbook is on github at https://github.com/Eztorr/HomeLabDocs/tree/mdbook
3. Create a script to serve the mdbook:
```
#!/bin/bash
#Script to serve mdbook over http, used for system service
set -e

#serves the mdbook over port 3000 unless specified otherwise with -p
sudo mdbook serve  /path/to/mdbook -n localipaddr

```
4. create a user called mdbook using
```
useradd mdbook
```

5. Create a service by using the command 
```
nano /etc/systemd/system/mdbook.service
```

- Create your service in this file
	
	```
	[Unit]
    Description=mdbook web service

    [Service]
    ExecStart=/path/to/script
    User=mdbook
    Restart=always 
    RestartSec=3 

    [Install]
    WantedBy=multi-user.target

	```
6. use ```sudo visudo``` and add the line ```mdbook ALL=(root) NOPASSWD: /bin/mdbook``` to allow the mdbook command to be used by the user mdbook

7. reload the systemd daemon and start the service by using the commands
```
systemctl reload-daemon
systemctl start mdbook.service
```

8. mdbook should be up on http://localipaddr:3000

# Using Github

Make sure to install git using ```apt install git```

Use ```git clone``` to clone desired repo

Set up a cronjob by using ```crontab -e``` to pull change from a repo
	
- I used a script call pull.sh
	
	```
	#!/bin/bash

	set -e
	
	#changes working directory to the one housing the git repo
	cd HomeLabDocs/

	#pulls from remote
	git pull origin mdbook
	```
	
- The cronjob I used pulled from my repo every day at 12am
	
	``` 0 0 * * * /home/mdbook/mdbook/pull.sh ```

