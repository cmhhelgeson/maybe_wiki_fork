If running the project locally, follow these instructions to get started:

## Note on Linux Distros

This guide is specifically for `Ubuntu 22.04`. If you are using another distro, please [let us know in Discord](https://link.maybe.co/discord) so we can add instructions for it.

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

## Install Node.js

To handle JavaScript in the project, you'll need Node.js installed.

We recommend installing through [NVM](https://github.com/nvm-sh/nvm), which is similar to `rbenv` and allows you to manage Node versions. Install it with the following command:

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

Verify your installation:

```sh
command -v nvm
```

Now install the latest version of Node and set it as the default:

```sh
nvm install node # "node" is an alias for the latest version

nvm alias default node
```

And verify your Node version:

```sh
node -v
```

### Install Yarn

For Rails JS bundling and Webpacker, you'll need yarn, a package manager for the JS ecosystem:

```sh
npm install -g yarn
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
