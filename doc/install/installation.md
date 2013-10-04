# Important notes

This installation guide was created for and tested on **FreeBSD 8.3 and 9.1** operating systems. Please read [`doc/install/requirements.md`](./requirements.md) for hardware and operating system requirements.

This is **NOT** the official installation guide to set up a production server since upstream doesnt support **FreeBSD**. To set up a **development installation** or for many other installation options please consult [the installation section in the readme](https://github.com/gitlabhq/gitlabhq#installation).

The following steps have been known to work. Please **use caution when you deviate** from this guide. Make sure you don't violate any assumptions GitLab makes about its environment.



If you find a bug/error in this guide please **submit a pull request** following the [`contributing guide`](../../CONTRIBUTING.md).

- - -

# Overview

The GitLab installation consists of setting up the following components:

1. Packages / Dependencies
2. Ruby
3. System Users
4. GitLab shell
5. Database
6. GitLab
7. Nginx


# 1. Packages / Dependencies3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
This installation guide was created for and tested on **FreeBSD 8.3 and 9.1** operating systems. Please read [`doc/install/requirements.md`](./requirements.md) for hardware and operating system requirements.
This is **NOT** the official installation guide to set up a production server since upstream doesnt support **FreeBSD**. To set up a **development installation** or for many other installation options please consult [the installation section in the readme](https://github.com/gitlabhq/gitlabhq#installation).
The following steps have been known to work. Please **use caution when you deviate** from this guide. Make sure you don't violate any assumptions GitLab makes about its environment.
If you find a bug/error in this guide please **submit a pull request** following the [`contributing guide`](../../CONTRIBUTING.md).
- - -
# Overview
The GitLab installation consists of setting up the following components:
1. Packages / Dependencies
2. Ruby
3. System Users
4. GitLab shell
5. Database
6. GitLab
7. Nginx
# 1. Packages / Dependencies
Change to `root` using `su`
We will use `portmaster` to install ports, and by default `portmaster` is not installed on FreeBSD. Make sure your system is up-to-date and install it.
        cd /usr/ports/ports-mgmt/portmaster && make install clean
**Note:**
Vim is an editor that is used here whenever there are files that need to be
edited by hand. But, you can use any editor you like instead.
    # Install vim as root
    portmaster editors/vim
Install the required packages:
        
        portmaster devel/git databases/redis devel/icu textproc/libxml2 mail/postfix textproc/libxslt
        
Commit summary: Extended description: (optional)
nerzhul loic.blot@frostsapphirestudios.com

Change to `root` using `su`

We will use `portmaster` to install ports, and by default `portmaster` is not installed on FreeBSD. Make sure your system is up-to-date and install it.

	cd /usr/ports/ports-mgmt/portmaster && make install clean

**Note:**
Vim is an editor that is used here whenever there are files that need to be
edited by hand. But, you can use any editor you like instead.

    # Install vim as root
    portmaster editors/vim

Install the required packages:
	
	portmaster devel/git databases/redis devel/icu textproc/libxml2 mail/postfix textproc/libxslt
	
Setup redis server:

	cp /usr/local/etc/redis.conf.sample /usr/local/etc/redis.conf
	echo "redis_enable="YES"" >> /etc/rc.conf
	service redis start
	
Make sure you have the right version of Python installed.

    # Install Python
    portmaster lang/python

    # Make sure that Python is 2.5+ (3.x is not supported at the moment)
    python --version

    # If it's Python 3 you might need to install Python 2 separately
    portmaster lang/python27



# 2. Ruby
Set Ruby 1.9 the default version if not already:

	echo RUBY_DEFAULT_VER=1.9 >> /etc/make.conf

Install ruby and rubygems:
	
	portmaster lang/ruby19 devel/ruby-gems sysutils/rubygem-bundler


# 3. System Users

Create a `git` user and group for Gitlab:

    pw addgroup git
    pw adduser git -g git -m -d /home/git -c "GitLab"


# 4. GitLab shell

GitLab Shell is a ssh access and repository management software developed specially for GitLab.

    # Go to home directory
    cd /home/git

    # Clone gitlab shell
    su -m git -c "git clone https://github.com/gitlab-freebsd/gitlab-shell.git"

    cd gitlab-shell

    # switch to right version

    su -m git -c "cp config.yml.example config.yml"

    # Edit config and replace gitlab_url
    # with something like 'http://domain.com/'
    
    
    # edit and make sure that is pointing to /usr/local/bin/redis-cli
    su -m git -c "vim config.yml"

    # Do setup
    su -m git -c ./bin/install


# 5. Database

To setup the MySQL/PostgreSQL database and dependencies please see [`doc/install/databases.md`](./databases.md).


# 6. GitLab

    # We'll install GitLab into home directory of the user "git"
    cd /home/git

## Clone the Source

    # Clone GitLab repository
    su -m git -c "git clone https://github.com/gitlab-freebsd/gitlabhq.git gitlab"


## Configure it

    cd /home/git/gitlab

    # Copy the example GitLab config
    su -m git -c "cp config/gitlab.yml.example config/gitlab.yml"

    # Make sure to change "localhost" to the fully-qualified domain name of your
    # host serving GitLab where necessary
    # make sure git path field is /usr/local/bin/git
    su -m git -c "vim config/gitlab.yml"

    # Make sure GitLab can write to the log/ and tmp/ directories
    chown -R git log/
    chown -R git tmp/
    chmod -R u+rwX log/
    chmod -R u+rwX tmp/

    # Create directory for satellites
    su -m git -c "mkdir /home/git/gitlab-satellites"

    # Create directory for pids and make sure GitLab can write to it
    su -m git -c "mkdir tmp/pids/"
    su -m git -c "mkdir tmp/sockets/"
    chmod -R u+rwX tmp/pids/
    chmod -R u+rwX tmp/sockets/

    # Copy the example Puma config
    su -m git -c "cp config/puma.rb.example config/puma.rb"

**Important Note:**
Make sure to edit both files to match your setup.

## Configure GitLab DB settings

    # Mysql
    su -m git -c "cp config/database.yml.mysql config/database.yml"

    # PostgreSQL
    su -m git -c "cp config/database.yml.postgresql config/database.yml"

Make sure to update username/password in config/database.yml.

## Install Gems

    cd /home/git/gitlab

    gem install charlock_holmes --version '0.6.9'

    # For MySQL (note, the option says "without")
    su -m git -c "bundle install --deployment --without development test postgres"

    # Or for PostgreSQL
    su -m git -c "bundle install --deployment --without development test mysql"


## Initialise Database and Activate Advanced Features

    su -m git -c "bundle exec rake gitlab:setup RAILS_ENV=production"


## Install rc Script


ATTENTION!!! THIS IS A NASTY SCRIPT!!
This is a temporary ugly-dirty-pseudo script "converted" from the original init.d, doesn't start on boot, only works if you do start manually:
	
	Download the rc script (will be /usr/local/etc/rc.d/gitlab):
	
    curl --output /usr/local/etc/rc.d/gitlab https://raw.github.com/gitlab-freebsd/gitlabhq/freebsd/lib/support/rc.d/gitlab
    chmod +x /usr/local/etc/rc.d/gitlab

Make GitLab start on boot:

    echo "gitlab_enable="YES"" >> /etc/rc.conf


## Check Application Status

Check if GitLab and its environment are configured correctly:

    su -m git -c "bundle exec rake gitlab:env:info RAILS_ENV=production"

To make sure you didn't miss anything run a more thorough check with:

    su -m git -c "bundle exec rake gitlab:check RAILS_ENV=production"

If all items are green, then congratulations on successfully installing GitLab!
However there are still a few steps left.

## Start Your GitLab Instance

    service gitlab start
    # or
    /etc/rc.d/gitlab restart


# 7. Nginx

**Note:**
If you can't or don't want to use Nginx as your web server, have a look at the
[`Advanced Setup Tips`](./installation.md#advanced-setup-tips) section.

## Installation

Install nginx and enable HTTP_GZIP_STATIC option:

    portmaster www/nginx
	echo "nginx_enable="YES"" >> /etc/rc.conf

## Site Configuration

Turn your nginx configuration into the sites-available/enabled method:

	curl --output /usr/local/etc/nginx/nginx.conf https://raw.github.com/gitlab-freebsd/gitlabhq/freebsd/lib/support/nginx/nginx.conf
	mkdir /usr/local/etc/nginx/sites-available
	mkdir /usr/local/etc/nginx/sites-enabled

Download an example site config:
	
    curl --output /usr/local/etc/nginx/sites-available/gitlab https://raw.github.com/gitlab-freebsd/gitlabhq/freebsd/lib/support/nginx/gitlab
    
Enable the gitlab site:

    ln -s /usr/local/etc/nginx/sites-available/gitlab /usr/local/etc/nginx/sites-enabled/gitlab

Make sure to edit the config file to match your setup:

    # Edit the config file
    vim /etc/nginx/sites-available/gitlab

## Restart

    service nginx restart


# Done!

Visit YOUR_SERVER for your first GitLab login.
The setup has created an admin account for you. You can use it to log in:

    admin@local.host
    5iveL!fe

**Important Note:**
Please go over to your profile page and immediately chage the password, so
nobody can access your GitLab by using this login information later on.

**Enjoy!**


- - -


# Advanced Setup Tips

## Custom Redis Connection

If you'd like Resque to connect to a Redis server on a non-standard port or on
a different host, you can configure its connection string via the
`config/resque.yml` file.

    # example
    production: redis://redis.example.tld:6379

## Custom SSH Connection

If you are running SSH on a non-standard port, you must change the gitlab user's SSH config.

    # Add to /home/git/.ssh/config
    host localhost          # Give your setup a name (here: override localhost)
        user git            # Your remote git user
        port 2222           # Your port number
        hostname 127.0.0.1; # Your server name or IP

You also need to change the corresponding options (e.g. ssh_user, ssh_host, admin_uri) in the `config\gitlab.yml` file.

## LDAP authentication

You can configure LDAP authentication in config/gitlab.yml. Please restart GitLab after editing this file.

## Using Custom Omniauth Providers

GitLab uses [Omniauth](http://www.omniauth.org/) for authentication and already ships with a few providers preinstalled (e.g. LDAP, GitHub, Twitter). But sometimes that is not enough and you need to integrate with other authentication solutions. For these cases you can use the Omniauth provider.

### Steps

These steps are fairly general and you will need to figure out the exact details from the Omniauth provider's documentation.

* Add `gem "omniauth-your-auth-provider"` to the [Gemfile](https://github.com/gitlabhq/gitlabhq/blob/master/Gemfile#L18)
* Run `su -m git -c "bundle install"` to install the new gem(s)
* Add provider specific configuration options to your `config/gitlab.yml` (you can use the [auth providers section of the example config](https://github.com/gitlabhq/gitlabhq/blob/master/config/gitlab.yml.example#L53) as a reference)
* Add icons for the new provider into the [vendor/assets/images/authbuttons](https://github.com/gitlabhq/gitlabhq/tree/master/vendor/assets/images/authbuttons) directory (you can find some more popular ones over at https://github.com/intridea/authbuttons)
* Restart GitLab

