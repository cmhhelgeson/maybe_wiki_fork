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

*I'm attempting to install version 3.3.0 of Ruby, but my terminal says version 3.3.0 is not available*


