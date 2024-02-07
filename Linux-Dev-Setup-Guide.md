If running the project locally, follow these instructions to get started:

## Note on Linux Distros

This guide is specifically for `Ubuntu 22.04`. If you are using another distro (other than Ubuntu) and cannot get the project started, please [let us know in Discord](https://link.maybe.co/discord) so we can add instructions for it.

## Install Ruby

You will need a Ruby version >=3 to run this project. Ideally, the latest version.

We recommend installing `rbenv` using Homebrew with the following commands:

```sh
sudo apt-get update

sudo apt-get install git-core zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev

sudo apt install rbenv

rbenv init
```

Follow the instructions in your shell.

Now, install a Ruby version:

```sh
rbenv install 3.3.0
```

And finally, set this as your default Ruby version for your machine if desired:

```sh
rbenv global 3.3.0
```

Check your Ruby version:

```sh
ruby -v
```

## Install PostgreSQL

Install Postgres and start it as a service:

```sh
sudo apt install postgresql libpq-dev

sudo systemctl start postgresql

# (Optional) You can start automatically when your system boots with:
sudo systemctl enable postgresql
```

## Start the Project

Start by forking and then cloning the repo locally. Then run the following commands to start the app:

```sh
cd maybe
cp .env.example .env
bundle install
rails db:setup
bin/dev
```

## Potential Issues

*I'm attempting to install version 3.3.0 of Ruby, but my terminal says version 3.3.0 is not available?*

Run the following command to list the available versions of ruby that can be installed by the rbenv's ruby-build plugin.

```sh
rbenv install --list
```

If version '3.3.0' is not listed as one of the options within the list, you may need to upgrade the ruby-build plugin within rbenv to gain access to a downloadabe version of ruby 3.3.0. To update ruby-build, simply enter the following command:

```sh
cd "$(rbenv root)"/plugins/ruby-build && git pull
```

Run the --list command again to see determine whether Ruby version 3.3.0 can now be downloaded.

*I've installed Ruby 3.3.0 and set it as my default Ruby version. However, a different version of Ruby is still listed as my default?*

If rbenv global 3.3.0 seems to run but doesn't change the Ruby version, your shell might be using a different ruby installation (such as one installed via *apt*). Confirm that you are using the rbenv managed version of ruby by running this command:

```sh
which ruby
```

When using rbenv, this command should usually list the home directory */root/.rbenv/shims/ruby* as the location of your ruby installation. If the command lists the */usr/bin/ruby* directory, or another separate directory, this likely means you are using an outside installation of ruby rather than the one managed by rbenv. 

To remedy this, add the following lines to the end of your /root/.bashrc file:

```sh
export PATH="/root/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
```

Save the file, reload your shell profile and verify that you're using the rbenv managed ruby installation by running these commands:

```sh
source /root/.bashrc
echo $PATH
which ruby
```








