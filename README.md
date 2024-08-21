# Upgrade ERPNext to Version 14 on Ubuntu 20+ or 22+

**Note:** You do **not** need to upgrade Ubuntu from 20+ to 22+ for upgrading ERPNext to Version 14. ERPNext and Frappe 14 work on both Ubuntu 20+ and 22+.

## Steps to Upgrade

1. **Take a Backup**  
   (Do I really need to explain this? 😅)

2. **Ensure All Customizations are Committed**

   ```bash
   cd /opt/bench/frappe-bench/apps/erpnext
   git status
Output should be:

vbnet
Copy code
Refresh index: 100% (6953/6953), done.
On branch version-13
Your branch is up to date with 'upstream/version-13'.
nothing to commit, working tree clean
bash
Copy code
cd /opt/bench/frappe-bench/apps/frappe
git status
Output should be:

vbnet
Copy code
On branch version-13
Your branch is up to date with 'upstream/version-13'.
nothing to commit, working tree clean
Check Python Version

bash
Copy code
python3 -V
Output should be Python 3.8.10. You need to upgrade to Python 3.10.

Check Node Version

bash
Copy code
node --version
Output should be v12.22.12. You need to upgrade to Node v16.x.

Check PIP Version

bash
Copy code
pip3 --version
Output should be 20.2. You need to upgrade to PIP 22.x.

Upgrade Python

bash
Copy code
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.10 python3.10-dev python3.10-distutils
Confirm Python version upgrade by running:

bash
Copy code
python3.10 --version
Output should be Python 3.10.6.

Make Python 3.10 the default:

bash
Copy code
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1
sudo apt install python-is-python3
Verify:

bash
Copy code
python -V
python3 -V
Both should output Python 3.10.6.

Upgrade PIP

bash
Copy code
curl -sS https://bootstrap.pypa.io/get-pip.py | python3.10
pip install html5lib
pip3 install --upgrade pip
sudo apt-get remove python3-apt -y
sudo apt-get install python3-apt -y
Verify:

bash
Copy code
pip --version
pip3 --version
Both should output pip 22.2.2.

Upgrade Node

bash
Copy code
curl -sL https://deb.nodesource.com/setup_16.x | bash -
apt-get install nodejs redis-server -y
Verify:

bash
Copy code
node --version
Output should be v16.17.0.

Upgrade NPM

bash
Copy code
npm upgrade
sudo npm install 16
sudo npm install -g npm@8.19.1
Verify:

bash
Copy code
npm --version
Output should be 8.19.1.

Move Your Old Python Environment

bash
Copy code
cd /opt/bench/frappe-bench/
mv env env-archive
Create a New Virtual Environment for Python 3.10

bash
Copy code
pip install virtualenv
virtualenv --python python3.10 env
env/bin/pip install -U pip
Switch Git Upstream from V13 to V14

bash
Copy code
env/bin/pip install -e apps/frappe -e apps/erpnext
pip3 install frappe-bench
bench switch-to-branch frappe erpnext version-14
Note: If the above command doesn't work, use this one:

bash
Copy code
bench switch-to-branch version-14 frappe erpnext --upgrade
Verify the branch:

bash
Copy code
cd /opt/bench/frappe-bench/apps/erpnext
git status
Output should be Your branch is up to date with 'upstream/version-14'.

bash
Copy code
cd /opt/bench/frappe-bench/apps/frappe
git status
Output should be Your branch is up to date with 'upstream/version-14'.

Install and Upgrade to V14

Please note that the monolith is broken in V14, and you must install the payments and hrms modules in addition to ERPNext:

bash
Copy code
bench get-app payments
bench get-app hrms
bench update --reset
bench --site *sitename* install-app hrms
bench --site *sitename* install-app payments
bench --site v13.erpgulf.com migrate
Restart services:

bash
Copy code
sudo service supervisor restart
sudo service nginx restart
That's it! Check whether your ERPNext and Frappe have upgraded to V14.

Additional Notes
Note 1:
If you face issues with Nginx due to log type errors (e.g., log type main not found), edit /etc/nginx/nginx.conf and add the following to the http section:

nginx
Copy code
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';
Note 2:
From @flexy2ky:

Modify the command in your document:

bash
Copy code
bench switch-to-branch version-14
This command will fail if the site has custom apps installed because it will try to switch the apps to version-14 as well. Instead, use:

bash
Copy code
bench switch-to-branch frappe erpnext version-14
This ensures that the switch-to-branch command only affects Frappe and ERPNext and ignores any other installed app.
