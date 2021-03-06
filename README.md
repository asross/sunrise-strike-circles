Pledge to Vote
--------------

This is the Sunrise Movement's Pledge to Vote website. It allows hub organizers (or anyone else with access) to collect information from people who want to pledge to vote at the next election. It also tags each "pledge" with the location where that pledge was entered.

It will eventually push all this data to EveryAction, but that has yet to be implemented.

## Usage
The main portion of the site is protected by a site-wide password. There isn't a user login system, but anyone visiting the site has to enter a password before they can enter any data. When the site is first set up, the active password is `sunrise`. Anyone with admin access (more on that later) can add/remove passwords, and mark certain passwords as active. This allows for different events to use different passwords, even if they're using the site at the same time.

Once logged in, there are a few main views.
* The homepage shows the list of pledges, in descending chronological order.
* Clicking on any of the pledges shown on the homepage brings you to a form where you can edit that pledge.
* Clicking the New Pledge header link brings you to a form where you can enter a new link.
* Clicking the Set Location header link brings you to a form where you can specify your current location (i.e., where you're entering pledges from). You can either choose a pre-existing location, or create a new one. Once you set your location, it will be saved for 24 hours, after which you'll have to re-enter it. This location is associated with each pledge that you submit. (If your location is not set/was set more than 24 hours ago, you'll be redirected to the Set Location page when you try to create a new pledge.)

To create new passwords or disable old passwords, and for more granular editing of the different parts of the system, go to `/admin/`. You have to be a superuser to log in here -- you can create a new superuser with `./manage.py createsuperuser`. I'll talk about that in more detail in the installation/setup instructions below.


## Installation and Setup

I'm going to assume you're using Ubuntu, and are comfortable with the command line here. Feel free to shoot me an email (jesse [at] jesseevers [dot] com) if you need any more guidance. I've included separate instructions for installing locally and on a server.

### Local installation

* Install system-level dependencies: `sudo apt install python3 python3-dev python3-pip python3.7 nodejs npm`
* Install Pipenv: `pip3 install --user pipenv`
* Add Pipenv to the path, by adding this line to `~/.bashrc` (or `~/.zshrc`, if you use `zsh`): `export PATH="$HOME/.local/bin/:$PATH"`
* Clone this repository and change directory into it: `git clone https://github.com/jlevers/sunrise-pledge-to-vote && cd sunrise-pledge-to-vote`
* Copy `.env.example` to `.env`, and fill in the variables in it. You can generate a secret key with [Djecrety](https://djecrety.ir/), and `SITE_URL` should be set to `localhost`.
* From inside the repository directory, install Python packages with `pipenv install`.
* Activate the project's virtual environment with `pipenv shell`.
* Install the packages needed to compile Sass: `./manage.py bulma install`
* Run migrations with `./manage.py migrate`
* Create a Django superuser with `./manage.py createsuperuser` (then follow the prompts)
* Start the Django server: `./manage.py runserver`
* Finally, if you're doing active development, start the Sass watch server to make sure Sass gets recompiled when you make changes. Do this in a new terminal, inside the repository: `./manage.py bulma start`

Go to `localhost:8000`, and you should be able to see the site! To access the admin site, go to `localhost:8000/admin`, and enter the credentials you specified when you created a superuser.


### Deploying to a Digital Ocean server

Quick note: these instructions were made well after the site was actually deployed, so some steps might be missing. If you need any help/clarification, shoot me an email at jesse [at] jesseevers [dot] com.

Many of these instructions overlap with the local deploy process.

* Log into [Digital Ocean](https://digitalocean.com). (If you're using the sunrise DO account, ask Andrew Jones for credentials.)
* Create a new Ubuntu 18.04 droplet. The smallest/cheapest option should be fine (1GB RAM, 25GB storage). When you're creating it, there should be an option to add an SSH key -- add yours. This will allow you to SSH into the droplet once it has been created.
* Once you have a droplet, follow the steps in Digital Ocean's [initial server setup guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04).
* Install system-level dependencies: `sudo apt install curl python3 python3-dev python3-pip python3.7 nginx nodejs npm`
* Install Pipenv: `pip3 install --user pipenv`
* Add Pipenv to the path, by adding this line to `~/.bashrc` (or `~/.zshrc`, if you use `zsh`): `export PATH="$HOME/.local/bin/:$PATH"`
* Clone this repository and change directory into it: `git clone https://github.com/jlevers/sunrise-pledge-to-vote && cd sunrise-pledge-to-vote`
* Copy `.env.example` to `.env`, and fill in the variables in it. You can generate a secret key with [Djecrety](https://djecrety.ir/), and `SITE_URL` should be set to the IP address of your droplet.
* Then, from inside the repository directory, install Python packages with `export PIPENV_VENV_IN_PROJECT="enabled" && pipenv install`. Setting the `PIPENV_VENV_IN_PROJECT` environment variable forces Pipenv to create the virtual environment for the project inside the project directory (in this case the virtual environment will be stored in `~/sunrise-pledge-to-vote/.venv/`).
* Activate the project's virtual environment with `pipenv shell`.
* Install the packages needed to compile Sass: `./manage.py bulma install`
* Run migrations with `./manage.py migrate`
* Create a Django superuser with `./manage.py createsuperuser` (then follow the prompts)
* Add your site's IP address to the `ALLOWED_HOSTS` list in `sunrise/settings/production.py`.
* Collect all static files so that they can be served by Nginx: `./manage.py collectstatic`
* Then, follow Digital Ocean's [guide for deploying a Django app](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04) using Gunicorn, starting where they tell you to "create an exception for port 8000". They use a lot of placeholder names -- don't forget to replace them all with their actual values in this app! For instance, replace `myproject` with `sunrise`, `sammy` with the name of the user you created in the initial setup guide, etc.