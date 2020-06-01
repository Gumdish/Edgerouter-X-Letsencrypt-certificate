# Edgerouter X - Let’s Encrypt certificate creation and installation.
# Let’s Encrypt setup to eliminate the certificate warnings on your Edgerouter

Here are my notes on how to get a Let’s Encrypt certificate installed on an Edgerouter X devices.

## References/Sources:
[Link to Ubiquity Community](https://community.ui.com/questions/How-to-install-a-Trusted-CA-Certificate-to-the-Edgerouter-and-access-the-Edgerouter-with-an-IPad/299f4059-1d0e-4967-b5d1-1f39158d5583)


**Important**
This setup is manual process and will not automatically renew the certificate.

I had to do this as a manual process because my domain registrar does not support DNS API. You will need to create a DNS TXT record for your domain. If you cannot create a TXT record, then this guide will not work for you.

You will need:
1. A registered domain - I use [hover.com](https://hover.evyy.net/c/2340677/660812/2799). Affiliate link. Please support the time and work I did to create this tutorial.
2. SSH access to your Edgerouter
3. acme.sh - A Linux bash script that handles all of the [Let’s Encrypt](https://github.com/acmesh-official/acme.sh) requests.

## Part A - Install acme.sh
1. SSH to your router and navigate to `/config/auth`
   - This directory is important because it will be preserved during any upgrades to the router software
2. Download and install acme.sh
   - `curl https://get.acme.sh | sh`
     - This script will do the following
       - Create `/config/auth/.acme.sh/acme.sh`
       - Download acme.sh 
       - Edit .profile
       - Attempt to create a cron entry
    - Edit .bashrc and add the .acme.sh directory to PATH
      - `PATH=$PATH:/config/auth/.acme.sh`
3. Confirm that acme is installed
   - Logout and login 
   - Type `acme.sh -v` to see version
     - You should see something like 
     ```
     https://github.com/acmesh-official/acme.sh
     v2.8.6
     ```
    
## Part B - Request Certificate
### Before beginning this section, you need to decide if you will use your root domain or a subdomain.
###### For example, mydomain.com or myrouter.mydomain.com. I am going to assume a subdomain is being used because it will clarify common questions. If you are not using a subdomain, you can easily modify the following steps. Remember to substitute your subdomain and domain in place of myrouter.mydomain.com.
1. Verify you are still in /config/auth/.acme.sh
2. Run `acme.sh --issue --dns -d myrouter.mydomain.com`
   - You will be shown a link to [ACME DNS manual mode](https://github.com/acmesh-official/acme.sh/wiki/dns-manual-mode). Read this if you need additional information about DNS manual mode.
3. Next run `acme.sh --issue --dns -d myrouter.mydomain.com --yes-I-know-dns-manual-mode-enough-go-ahead-please`
   - You will see instructions on how to edit your DNS with a TXT record.
   - If you are not familiar with how to create a DNS TXT record, please contact your domain registrar. If you cannot create a TXT record, then this guide will not work for you.
   - Create a TXT record named `_acme-challenge.myrouter` for mydomain.com. The value of the TXT record will come from the output from command you used in Part B, step 3.
4. Now run `acme.sh --issue --dns -d myrouter.mydomain.com --yes-I-know-dns-manual-mode-enough-go-ahead-please --renew`
   - Notice the `--renew` at the end of the command.
5. The acme.sh script will create multiple files in a new directory.
   - The directory will be named myrouter.mydomain.com
   - The files we need are:
     - fullchain.cer
     - myrouter.mydomain.com.cer
     - myrouter.mydomain.com.key
   
## Part C - Install Certificate
1. Navigate to `/config/auth/.acme.sh/myrouter.mydomain.com`
2. Combine the three files into a new file named `server.pem`. The file must be named `server.pem`.
   - Run `cat myrouter.mydomain.com.key myrouter.mydomain.com.cer fullchain.cer > server.pem
3. Backup your existing server.pem
   - `cp /etc/lighttpd/server.pem /etc/lighttpd/server.pem.old`
4. Copy new server.pem
   - `cp /config/auth/.acme.sh/myrouter.mydomain.com/server.pem /etc/lighttpd/server.pem`
5. Restart lighttpd
   - `sudo kill -SIGTERM $(cat /var/run/lighttpd.pid)`
   - `sudo /usr/sbin/lighttpd -f /etc/lighttpd/lighttpd.conf`
   - If you see an error after restarting lighttpd, you can copy server.pem.old back to server.pem and restart lighttpd. Something may be wrong in your certificates that caused the server to fail to start.
   
## Part D - Renew Certificate (Every 89 days)
1. Navigate to `/config/auth/.acme.sh`
2. Run `acme.sh --issue --dns -d myrouter.mydomain.com --yes-I-know-dns-manual-mode-enough-go-ahead-please`
3. Edit your TXT record and save the new string provided from the previous step
4. Run `acme.sh --issue --dns -d myrouter.mydomain.com --yes-I-know-dns-manual-mode-enough-go-ahead-please --renew`
5. Repeat steps 2, 4 and 5 from Part C


