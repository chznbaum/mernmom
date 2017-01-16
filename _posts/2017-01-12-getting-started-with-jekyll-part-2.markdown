---
layout: post
title: "Getting Started with Jekyll - Part 2: Meeting Jekyll's Demands"
date: 2017-01-12 07:00:00 -0500
description: "When this left off in Part 1, you had a basic Nginx server running and set up with remote access and a firewall."
image:
  feature: /assets/Screenshot-from-2017-01-15-23-01-37.png
categories:
- tutorials
tags:
- blogging
- capistrano
- jekyll
- nginx
- ruby
published: true
---
When this left off in [Part 1](http://mernmom.com/2017/01/09/getting-started-with-jekyll-part-1.html), you had a basic Nginx server running and set up with remote access and a firewall.

<!--more-->

So now that you have this awesome server, you should put some stuff on it.

## Precious Rubies ##

First you'll need to make sure you have the [Ruby](https://www.ruby-lang.org/en/) programming language installed both on your server *and* on your local computer. While you can have multiple versions of Ruby installed, you need both server and local to be using the same version. *This will take some time to correct later since you'll have to reinstall almost everything, so it's important to pay attention now.*

To install Ruby, you're going to use the *Ruby Version Manager (RVM)* which also makes it easy to install additional tools and make updates. To install both RVM and Ruby, follow the instruction on [the RVM website](https://rvm.io/) which will look something like the following but will change over time. You'll want to make sure `--ruby` is added like this:

~~~ bash
$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
$ \curl -sSl https://get.rvm.io | bash -s stable --ruby
~~~

Do the same for both your server and local computer. Before moving on, check which version of Ruby you are running by entering the following command for each system:

~~~ bash
$ ruby -v
~~~

You should have the exact same version listed for both. If you have multiple Rubies installed, say on your local computer, you can specify which version to use with the following, replacing the numbers with the actual version number you want to use:

~~~ bash
$ rvm use 2.3.3
~~~

Once you have the same versions being used for both, you can start working on dependencies.

## Crucial Pieces ##

From here, you're going to be installing and configuring packages called *gems*.

In your terminal for your local computer, enter the following:

~~~ bash
$ gem install jekyll capistrano bundler
~~~

Those are set to install the Jekyll site generator, the Capistrano deployment tool, and the Bundler gem manager. You may have received an error message stating you were missing some needed gem. As many times as you get this type of error, just go back into your terminal and install the gem it tells you, then re-attempt installing our three like so:

~~~ bash
$ gem install gem_name
$ gem install jekyll capistrano bundler
~~~

Now that you have those set up locally, it's time to see what you're working with in Jekyll. Go ahead and enter the following:

~~~ bash
$ jekyll new blog
$ cd blog
$ jekyll serve
~~~

This will generate and serve a basic Jekyll blog locally. Check it out! Go into your browser to `localhost:4000`, and you should see something like the following:

![Jekyll Blog Demo Site](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/Screenshot-from-2017-01-04-23-08-31.png)

When you're done, you're going to see that when you enter our server's address. To do that, you're going to need to configure a few things.

## Configuration ##

### Server-Side ###

In your terminal, log back into your server's non-root user account. You're going to need a directory for our site to go in, so go ahead and make one:

~~~ bash
$ mkdir www
$ cd www
~~~

You can title it whatever you like. In that directory, you're going to add a couple of additional ones that will be needed for your site:

~~~ bash
$ mkdir shared
$ mkdir shared/log
~~~

Next, you'll configure Nginx to know where to look for your site. Open the Nginx configuration file:

~~~ bash
$ sudo nano /etc/nginx/nginx.conf
~~~

And look for a section that says:

~~~
##
# Virtual Host Configs
##

# include /etc/nginx/conf.d/*.conf;
~~~

Make sure the next line reads as follows, uncommented and replacing your_site.com with the domain your site will use:

~~~
include /etc/nginx/sites-enabled/your_site.com;
~~~

Save the file, and you'll move on. Now, you need to go into the folders Nginx will look for our site configuration information:

~~~ bash
$ cd /etc/nginx/sites-available
~~~

There is already a file for the server demo page, but you're going to use that to create one for our Jekyll site:

~~~ bash
$ mv ./default ./your_site.com
$ sudo nano your_site.com
~~~

Once in the file, make it look something like the following, replacing `user_name` with your non-root user's name and `00.00.00.00` with your server's IP address or hostname if you've already configured it. Uncomment the lines `listen 443 ssl;` and `listen [::]:80;` if you have set up SSL certificates for your domain:

~~~
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# http://wiki.nginx.org/Pitfalls
# http://wiki.nginx.org/QuickStart
# http://wiki.nginx.org/Configuration
#
# Generally, you will want to move this file somewhere, and start with a clean
# file but keep this around for reference. Or just disable in sites-enabled.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
  # listen on http (port 80)
  # remove the "default_server" if you are running multiple sites off the same VPS
  listen 80;
  listen [::]:80;

  # SSL configuration
  #
  # listen 443 ssl;
  # listen [::]:443 ssl;
  #
  # Note: You should disable gzip for SSL traffic.
  # See: https://bugs.debian.org/773332
  #
  # Read up on ssl_ciphers to ensure a secure configuration.
  # See: https://bugs.debian.org/765782
  #
  # Self signed certs generated by the ssl-cert package
  # Don't use them in a production server!
  #
  # include snippets/snakeoil.conf;

  root /home/user_name/www/current/public;

  # Add index.php to the list if you are using PHP
  index index.html index.htm index.nginx-debian.html

  server_name 00.00.00.00;

  location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;
  }
  # how long should static files be cached for, see http://nginx.org/en/docs/http/ngx_http_headers_module.html for options
  expires 1d;

  # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
  #
  #location ~ \.php$ {
  #  include snippets/fastcgi-php.conf;
  #  
  #  # With php7.0-cgi alone:
  #  fastcgi_pass 127.0.0.1:9000;
  #  # With php7.0-fpm:
  #  fastcgi_pass unix:/run/php/php7.0-fpm.sock;
  #}

  # deny access to .htaccess files, if Apache's document root
  # concurs with nginx's one
  #
  #location ~ /\.ht {
  #  deny all;
  #}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#  listen 80;
#  listen [::]:80;
#
#  server_name example.com;
#  root /var/www/example.com;
#  index index.html;
#
#  location / {
#    try_files $uri $uri/ =404;
#  }
#}
}
~~~

I've left a lot in here that is not absolutely necessary but is helpful to learn from. Once the file is configured, save it, and you'll need to use a *symlink* in your sites-enabled directory so Nginx can find it:

~~~ bash
$ sudo ln -s /etc/nginx/sites-available/your_site.com /etc/nginx/sites-enabled/your_site.com
~~~

Using a symlink (symbolic link) basically places a file that references back to another file, so the same information can be used in two places (`sites-available` and `sites-enabled`) but only needs to be updated in one (`sites-available`). It's a great way to save time and reduce errors.

Now you need to make sure your Nginx server can access and the files it needs to run, as well as all parent folders involved. To do this, you're going to make the Nginx user the owner of the directory you're going to deploy to, and then give it read and write access to all the directories it will need:

~~~ bash
$ sudo chown -R -www-data:www-data /home/user_name/www/current
$ sudo chmod 755 -R /home
~~~

Finally go ahead and check to make sure your setup of Nginx doesn't create any errors, and then restart the Nginx worker:

~~~ bash
$ sudo nginx -t
$ sudo kill -HUP `cat /var/run/nginx.pid`
~~~

Now that your server is configured properly, you can move on to setting up your deployment tool, Capistrano.

### Capistrano ###

Back on your local machine, in the directory your blog is going in (say, `/user_name/blog`), go ahead and edit what's called a *Gemfile*, which is basically just a file that lists what and what versions of Ruby and gems you'll need:

~~~ bash
$ nano Gemfile
~~~

Make it look something like this, keeping any versions, plugins or themes already listed in the file:

~~~ ruby
source "https://rubygems.org"
ruby "2.3.3"
    
# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#   bundle exec jekyll serve
#
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
gem "jekyll", "3.3.1"
gem "capistrano"
gem "capistrano-rvm"
gem "capistrano-bundler"
gem "rvm1-capistrano3", require: false
    
# This is the default theme for new Jekyll sites. You may change this to anything you like.
gem "minima", "~> 2.0"
    
# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
# gem "github-pages", group: :jekyll_plugins
    
# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.6"
end
~~~

Once that's in place, in your terminal, run the following to create some files Capistrano will need to deploy the site:

~~~ bash
$ cap install
~~~

Here, you'll have a bit more editing to do. In your terminal, type:

~~~ bash
$ nano Capfile
~~~

Like the Gemfile, the Capfile will need to list some pieces required for Capistrano to work. Make it look kind of like this:

~~~ ruby
# Load DSL and set up stages
require "capistrano/setup"

# Include default deployment tasks
require "capistrano/deploy"

# Load the SCM plugin appropriate to your project:
#
# require "capistrano/scm/hg"
# install_plugin Capistrano::SCM::Hg
# or
# require "capistrano/scm/svn"
# install_plugin Capistrano::SCM::Svn
# or
require "capistrano/scm/git"
install_plugin Capistrano::SCM::Git

# Include tasks from other gems included in your Gemfile
#
# For documentation on these, see for example:
#
#   https://github.com/capistrano/rvm
#   https://github.com/capistrano/rbenv
#   https://github.com/capistrano/chruby
#   https://github.com/capistrano/bundler
#   https://github.com/capistrano/rails
#   https://github.com/capistrano/passenger
#
require "capistrano/rvm"
require "rvm1/capistrano3"
# require "capistrano/rbenv"
# require "capistrano/chruby"
require "capistrano/bundler"
# require "capistrano/rails/assets"
# require "capistrano/rails/migrations"
# require "capistrano/passenger"

# Load custom tasks from `lib/capistrano/tasks` if you have any defined
Dir.glob("lib/capistrano/tasks/*.rake").each { |r| import r }
~~~

Making these changes will tell Capistrano that you're using RVM to manage ruby versions and bundler to manage gems, as well as which version of Capistrano to expect.

To make sure all the necessary gems are properly set up for use, in your terminal type:

~~~ bash
$ bundle update
~~~

Then the next file to edit:

~~~ bash
$ nano config/deploy/production.rb
~~~

Edit it so it looks like this, replacing `00.00.00.00` with your server's address and `user_name` with your non-root user:

~~~ ruby
set :stage, :production

# Extended Server Syntax
# ======================
# This can be used to drop a more detailed server definition into the
# server list. The second argument is a, or duck-types, Hash and is
# used to set extended properties on the server.

server "00.00.00.00", user: "user_name", port: 22, roles: %w{web app}

set :bundle_binstubs, nil

set :bundle_flags, "--deployment --quiet"
set :rvm_type, :user

SSHKit.config.command_map[:rake] = "bundle exec rake"
SSHKit.config.command_map[:rails] = "bundle exec rails"

namespace :deploy do

  desc "Restart application"
  task :restart do
    on roles(:app), in: :sequence, wait: 5 do
      # execute :touch, release_path.join("tmp/restart.txt")
    end
  end

  after :finishing, "deploy:cleanup"
end

# server-based syntax
# ======================
# Defines a single server with a list of roles and multiple properties.
# You can define all roles on a single server, or split them:

# server "example.com", user: "deploy", roles: %w{app db web}, my_property: :my_value
# server "example.com", user: "deploy", roles: %w{app web}, other_property: :other_value
# server "db.example.com", user: "deploy", roles: %w{db}



# role-based syntax
# ==================

# Defines a role with one or multiple servers. The primary server in each
# group is considered to be the first unless any hosts have the primary
# property set. Specify the username and a domain or IP for the server.
# Don't use `:all`, it's a meta role.

# role :app, %w{deploy@example.com}, my_property: :my_value
# role :web, %w{user1@primary.com user2@additional.com}, other_property: :other_value
# role :db,  %w{deploy@example.com}



# Configuration
# =============
# You can set any configuration variable like in config/deploy.rb
# These variables are then only loaded and set in this stage.
# For available Capistrano configuration variables see the documentation page.
# http://capistranorb.com/documentation/getting-started/configuration/
# Feel free to add new variables to customise your setup.



# Custom SSH Options
# ==================
# You may pass any option but keep in mind that net/ssh understands a
# limited set of options, consult the Net::SSH documentation.
# http://net-ssh.github.io/net-ssh/classes/Net/SSH.html#method-c-start
#
# Global options
# --------------
#  set :ssh_options, {
#    keys: %w(/home/rlisowski/.ssh/id_rsa),
#    forward_agent: false,
#    auth_methods: %w(password)
#  }
#
# The server-based syntax can be used to override options:
# ------------------------------------
# server "example.com",
#   user: "user_name",
#   roles: %w{web app},
#   ssh_options: {
#     user: "user_name", # overrides user setting above
#     keys: %w(/home/user_name/.ssh/id_rsa),
#     forward_agent: false,
#     auth_methods: %w(publickey password)
#     # password: "please use keys"
#   }
~~~

And last but not least, you'll edit:

~~~ bash
$ nano config/deploy.rb
~~~

Make sure it looks like the following. You're going to replace `user_name` with your local username, `ruby-2.3.3` with whichever version you are using, the application name of `"blog"` with your choice if you want to change it, and the address listed next to `repo_url` to the address of your own repository if you use git (otherwise leave those quotes empty):

~~~ ruby
# config valid only for current version of Capistrano
lock "3.7.1"

set :rvm1_ruby_version, "ruby-2.3.3"

set :application, "blog"
set :repo_url, "https://github.com/user_name/blog.git"

# Default branch is :master
# ask :branch, `git rev-parse --abbrev-ref HEAD`.chomp

# Default deploy_to directory is /var/www/my_app_name
set :deploy_to, "/home/user_name/www"

# Default value for :format is :airbrussh.
# set :format, :airbrussh

# You can configure the Airbrussh format using :format_options.
# These are the defaults.
# set :format_options, command_output: true, log_file: "log/capistrano.log", color: :auto, truncate: :auto

# Default value for :pty is false
# set :pty, true

# Default value for :linked_files is []
# append :linked_files, "config/database.yml", "config/secrets.yml"

# Default value for linked_dirs is []
# append :linked_dirs, "log", "tmp/pids", "tmp/cache", "tmp/sockets", "public/system"

# Default value for default_env is {}
# set :default_env, { path: "/opt/ruby/bin:$PATH" }

# Set stages
set :stages, ["staging", "production"]
set :default_stage, "production"

# Default value for :log_level is :debug
set :log_level, :debug

# Default value for keep_releases is 5
set :keep_releases, 5

namespace :deploy do

  desc "Restart application"
  task :restart do
    on roles(:app), in: :sequence, wait: 5 do
      # Your restart mechanism here, for example:
      # execute :touch, release_path.join("tmp/restart.txt")
    end
  end

  before :restart, :build_public do
    on roles(:app) do
      within release_path do
        execute "/home/user_name/.rvm/gems/ruby-2.3.3/wrappers/jekyll", "build --destination public"
      end
    end
  end

  after :publishing, :restart

end
~~~

Finally, go ahead and tell Capistrano to deploy your current site by typing:

~~~ bash
$ cap production deploy
~~~

Open your browser window and type your server's address. You should see this:

![Jekyll Blog Demo Site](https://raw.githubusercontent.com/chznbaum/mernmom/master/assets/Screenshot-from-2017-01-04-23-08-31.png)

If you did, great! You set up your Jekyll site to push your local content to your server!

You're finished for today; on Monday you'll learn more about updating and deploying changes, as well as getting familiar with Jekyll's filesystem! Want something to do in the meantime? If you set up your domain name last time, here's a useful [guide](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04) by [Mitchell Anicas](https://twitter.com/thisismitch) on securing your server with a free [Lets Encypt](https://letsencrypt.org/) SSL certificate.

Questions, comments or concerns? Continue the conversation on [Twitter](https://twitter.com/intent/tweet?text=%40chznbaum&url=http%3A%2F%2Fmernmom.com%2F2017%2F01%2F12%2Fgetting-started-with-jekyll-part-2.html). Clicking that link will open a Twitter prompt directed at me including the link to this article for your convenience.