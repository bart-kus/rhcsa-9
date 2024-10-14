# Systemd

## Common commands

``systemctl status``
The systemctl status command provides information about the status of a service or unit in a Linux system that uses systemd

List all services
``systemctl list-units -t service``

List dependencies
``systemctl list-dependencies``
A good overview of what is "on" and what is "off". 

To enable the service to start at boot
``systemctl enable name.service``

To manually start a service
``systemctl start name.service``

To restart a service
``systemctl restart name.service``

Reload config
``systemctl reload name.service``
`reload` will reload a specific service. That means that the systemd will send a SIGHUP signal to a service, and that signal will tell the service to reload its configuration files, which has nothing to do with systemd config files.

Reload custom service files
``systemctl daemon-reload``
Location of custom systemd units is "/etc/systemd/system". This command reloads the service files in that directory.

Check the configuration of a unit file
``systemctl cat sshd.service``

Edit unit file
``systemctl edit sshd.service``
By default it uses Nano. To change that to Vim.
``export SYSTEMD_EDITOR=/usr/bin/vim``
This creates a drop-in-file in "/etc/systemd/system/"
If it does not create the drop-in-file automatically, do ``systemctl daemon-reload``

To see tunables for a service do ``systemctl show httpd.service``

/user/lib/systemd/system/ is for configuration files provided by packages.
Do not edit those directly since they can be overwritten by newer packages.

/etc/tuned/ and /usr/lib/tuned/ need an explanation for it here.

Mask
To prevent certain units from starting up, use ``systemctl mask``. It links a unit to the /dev/null device, which ensures it cannot be started. For instance ``systemctl mask nginx``

``systemctl unmask`` removes the unit mask.

## Systemd Journal (journalctl)

The journalctl command is used to view logs managed by systemd's journal in Linux systems. Let's break down the specific options you mentioned:

1. Persistent Journaling
For journalctl to show logs from previous boots, persistent journaling must be enabled. By default, the logs are kept only in memory and are lost upon reboot. To enable persistent journaling, you need to:

Create the directory that holds persistent journal logs:


``sudo mkdir -p /var/log/journal``
Restart the systemd-journald service:


``sudo systemctl restart systemd-journald``
After enabling this, the logs will be stored in /var/log/journal/, and you’ll be able to see logs from all boots.

2. journalctl --list-boots
This command lists all the boots that have been logged by systemd. It shows the boot number, boot ID, and the timestamp for when the boot happened. Each boot is assigned a boot ID, and you can reference that ID when querying logs from specific boots.


``journalctl --list-boots``
Example output:

yaml
Copy code
-1 e92c5d0c19294d98b8a5e7c9d705f822 Sun 2024-10-06 12:10:45 UTC—Sun 2024-10-06 14:15:32 UTC
 0 734d3a5f59f54f75a5ebf50f9f1e3b60 Sun 2024-10-07 09:00:10 UTC—Sun 2024-10-07 10:30:12 UTC
Boot -1: Logs for the previous boot (e92c5d0c...).
Boot 0: Logs for the current boot (734d3a5...).
3. journalctl -x (Augment Log Lines with Explanation Texts)
The -x flag in journalctl adds explanatory texts from the system message catalog. This can be useful when you’re trying to understand what specific log messages mean. Systemd provides these descriptions to give context to the messages.

Example:
``journalctl -x``
This will show logs with added explanations where available, making it easier to understand errors and warnings in the logs.

4. journalctl -r (Reverse Output, Show Newest First)
The -r flag reverses the order of log output, so the most recent log entries appear first. Normally, logs are shown in chronological order, with the oldest entries first, but reversing the order allows you to quickly see the latest logs.

Example:
``journalctl -r``
This is useful if you're troubleshooting and want to immediately see what happened most recently, instead of scrolling through older logs first.

5. journalctl -b (Show Logs from a Specific Boot)
The -b option filters logs to show entries from a specific boot. Every time the system boots, a unique boot ID is generated. You can specify which boot to display logs for by using the boot ID or the relative boot number.

journalctl -b (no argument): Shows logs from the current boot.
journalctl -b -1: Shows logs from the previous boot.
journalctl -b 0: Also shows logs from the current boot.
journalctl -b <boot_id>: Shows logs from the specified boot ID.
Example:

``journalctl -b``
This will show logs for the current boot.

If you wanted logs from the previous boot, you could use:


``journalctl -b -1``
6. Combining Options (journalctl -xrb)
You can combine these options to enhance your log view:

-x: Add explanatory help texts.
-r: Reverse the output so the newest logs are displayed first.
-b: Show logs from a specific boot.
Example:

``journalctl -xrb``
This will display logs from the current boot, with the most recent entries shown first, and explanatory texts included for better understanding.

Alternatively, if you want logs from the previous boot, you can use:


``journalctl -xrb -1``
Summary of the Options:
-x: Provides explanations for log entries, helping to clarify errors or warnings.
-r: Reverses the output to show the newest logs first, useful for recent events.
-b: Filters logs by specific boot instances using boot ID or relative numbers.
By combining these flags, you can quickly filter and understand system logs, focusing on the newest or most relevant events from a specific boot session, with helpful context included.







Show all boots that have been logged. Needs to have persistent journalling.
``journalctl --list-boots``

``journalctl -xrb``
-x = Augment log lines with explanation texts from the message catalog. This will add explanatory help texts to log messages
-r = Reverse output so that the newest entries are displayed first.
-b = Show messages from a specific boot. This will add a match for "_BOOT_ID=".
The argument may be empty, in which case logs for the current boot will be shown.

### How to make the journal persistent

You could use rsyslog to make the journal persistent.

"/etc/systemd/journal.conf"
The setting "Storage=auto" ensures that persistent storage is happening automatically **after manually creating the directory** "/var/log/journal"

Then we need to restart the journal service.
``systemctl restart systemd-journal-flush.service``

### Common commands

View only messages with a priority error and higher.
``journalctl -p err``

View the last 10 lines, and adds new messages when they are added.
``journalctl -f``

Show messages for the sshd service only.
``journalctl -u sshd.service``

### See space used by Journalctl

See current settings for growth and what it's currently using.
``journalctl | grep -E 'Runtime Journal|System Journal'``


## Systemd Timers (Scheduling)

When using systemd timers, the timer should be started, and **not** the service unit.

``systemctl list-units -t timer``
``systemctl list-unit-files dnf-makecache*``
``systemctl status dnf-makecache.timer``
Checkout the "Trigger and Triggers".

To schedule and activate a timer you use the "OnCalendar" option.

``OnCalendar=*:00/20`` runs every 20 minutes.

You can use "OnUnitActiveSec" to start the unit at a specific time after the unit was last activated.

You can use "OnBootSec" or "OnStartupSec" to start the unit a specific time after booting.

## Working with Tuned

Install tuned. ``dnf install tuned``

To see available commands, type this in and press double tab. ``tuned-adm`` 

To see available profiles. ``tune-adm list``

Config files for Tuned are located in "/usr/lib/tuned/".








