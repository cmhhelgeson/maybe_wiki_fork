***<u>I've installed postgresql but I am unable to start the server?<u>***

The standard Linux setup instructs you to initialize postgresql with the command *sudo systemctl start postgresql*. However, this assumes that you're Linux distribution is using 'systemd' as its initialization system, when it may be using 'init'. To determine whether your system is utilizing 'init' or 'systemd' as its initialization system, run the following command:

```sh
ps -p 1
```

If you're Linux distribution is using init, the output should look something like this:

```sh
PID TTY          TIME CMD
  1 ?        00:00:00 init
```

The text underneath the CMD header indicates the intialization system your Linux distribution is using. If the initialization system is listed as 'init' rather than 'systemd', then you can start the PSQL server by running this command:

```sh
sudo service postgresql start
```