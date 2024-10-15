# Scheduling Tasks

For the exam, don't study and learn about crond. Study and learn about crond for real world legacy systems that you work on everyday. **Systemd timers** is what is primarily used on RHEL 9 and that's reflected on the exam.

``systemctl list-units -t timer``

**Logrotate example.**

"/usr/lib/systemd/system/"
``ls logro*``
You will see "logrotate.timer"

``systemctl cat logrotate.timer``

``systemctl status logrotate.service``
You will see a line.
"TriggeredBy: ● logrotate.timer"



You're absolutely right. In RHEL 9 and newer versions, systemd timers have largely replaced the traditional cron jobs for scheduling tasks. While crond is still available and often used on older systems or for backward compatibility, systemd timers are now the primary tool for task scheduling in modern Red Hat systems, including RHEL 9. For the RHCSA (EX200) exam and real-world scenarios involving RHEL 9, understanding systemd timers is crucial.

Understanding systemd Timers
Systemd timers offer several advantages over traditional cron:

Greater flexibility: They can trigger services based on events, not just time.
Better integration: Systemd timers are managed by the systemd process, providing better logging, monitoring, and control.
Precision: Timers have more granular control over intervals, including millisecond accuracy and monotonic timers.
How to Work with systemd Timers
1. Basic Components of a Timer
A systemd timer consists of two units:

A service unit (.service) that defines the action to be performed.
A timer unit (.timer) that defines when the action should be performed.
The timer unit triggers the service unit based on a schedule, event, or condition.

2. Creating a Systemd Timer
Suppose you want to run a script located at /usr/local/bin/myscript.sh every day at midnight.

Step 1: Create the Service Unit
The service unit defines what task will be executed. Create a file called /etc/systemd/system/myscript.service: \


[Unit]
Description=Run my custom script

[Service]
ExecStart=/usr/local/bin/myscript.sh \
Step 2: Create the Timer Unit
The timer unit defines when the service should run. Create a file called /etc/systemd/system/myscript.timer:


[Unit]
Description=Run myscript daily at midnight

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
OnCalendar=daily: This triggers the timer daily at midnight (00:00). You can also use other calendar expressions like OnCalendar=weekly, OnCalendar=hourly, or custom times like OnCalendar=Mon *-*-* 12:00:00.
Persistent=true: If the system was down when the timer was supposed to run, the service will run immediately after boot (as soon as possible). \

Step 3: Enable and Start the Timer
After creating both the service and timer files, enable and start the timer:


``sudo systemctl daemon-reload``           # Reload systemd to recognize new units \
``sudo systemctl enable myscript.timer``   # Enable the timer to start at boot \
``sudo systemctl start myscript.timer``    # Start the timer now \
3. Viewing and Managing Timers \
You can view active timers and their status with: \


``systemctl list-timers``
This command will show when the timer was last triggered and when it is scheduled to run next.

To check the status of a specific timer: \


systemctl status myscript.timer
To manually stop or disable a timer:


``sudo systemctl stop myscript.timer``
``sudo systemctl disable myscript.timer``\
4. Using OnBootSec and OnUnitActiveSec for Intervals\
Instead of OnCalendar, you can use monotonic timers like OnBootSec and OnUnitActiveSec to schedule tasks at intervals relative to boot time or the last activation.\

For example, to run the script 5 minutes after boot and then every 30 minutes: \


[Timer]
OnBootSec=5min
OnUnitActiveSec=30min \
5. Logging and Troubleshooting \
Systemd timers benefit from systemd's logging infrastructure: \

You can check the logs for the service using journalctl: \

journalctl -u myscript.service \ 
- If the service failed, you'll see detailed logs and error messages here.
#### Comparison: cron vs systemd Timers
| Feature               | Cron                                   | Systemd Timers                                    |
|-----------------------|----------------------------------------|--------------------------------------------------|
| **Configuration**      | `/etc/crontab`, `cron.d`, `crontab` files | Timer and service units (`.timer`, `.service`)    |
| **Granularity**        | Minute-level                          | Millisecond-level, with complex time expressions  |
| **Integration**        | Separate system                       | Fully integrated with systemd, better logging and control |
| **Event-based triggers**| No                                    | Yes (e.g., `OnActiveSec`, `OnBootSec`, `OnCalendar`) |
| **Persistent scheduling**| No                                  | Yes (can run missed tasks after boot)             |
| **Logging**            | Separate log files or mail            | Integrated with `journalctl`                      |

Summary:
For the RHCSA (EX200) exam: Focus on systemd timers, as they are now the preferred method of scheduling tasks in RHEL 9.
Legacy systems: For real-world environments running older RHEL versions, it's still important to know crond. However, be aware that systemd timers are the primary method in modern RHEL systems.
By practicing both cron and systemd timers, you’ll be ready for both legacy systems and modern RHEL 9 environments.
