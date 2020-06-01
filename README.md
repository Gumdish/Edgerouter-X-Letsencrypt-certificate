# Edgerouter X - Letsencrypt certificate creation and installation

Here are my notes on how to get a Letsencrypt certificate installed on an Edgerouter X devices.

## References/Sources:
[Link to Ubiquity Community](https://community.ui.com/questions/How-to-install-a-Trusted-CA-Certificate-to-the-Edgerouter-and-access-the-Edgerouter-with-an-IPad/299f4059-1d0e-4967-b5d1-1f39158d5583)


**Important**
This setup is manual process and will not automatically renew the certificate.

I had to do this as a manual process because my domain registrar does not support DNS API

You will need:
1. A registered domain - I use hover.com
2. SSH access to your Edgerouter
3. acme.sh - A Linux bash script that handles all of the [Letsencrypt](https://github.com/acmesh-official/acme.sh) requests.

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
###### For example, mydomain.com or myrouter.mydomain.com. 
###### I am going to assume a subdomain is being used because it will clarify common questions. If you are not using a subdomain, you can easily modify the following steps.
1. 
## Step 3 -
