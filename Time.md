# General information

The time zone configured on the server is found in /etc/localtime. This is a symbolic link that points to one of the time zones files in /usr/share/zoneinfo.

# Configure time service clients

``hwclock`` sets the hardware time.
If the hardware clock is not correct but the system time is you can sync the hardware time to the 
system time with ``hwclock --systohc``

Use ``date`` so show current date and time.

Use ``timedatectl`` to manage time and time zone configuration.

``timedatectl status`` show all time properties in use.
``timedatectl list-timezones`` show all available timezones.
``timedatectl set-timezone Europe/Rome``
``timedatectl set-time`` \
``timedatectl set-timezone`` \
``timedatectl set-ntp`` enables or disables NTP sync.

An NTP service must be configured. The default for RHEL is Chrony.
If your server is running **systemd-timesyncd.service** you must disable that service before enabling Chrony.

## Chronyd
Chronyd is the default RHEL 9 NTP service.
Use "/etc/chrony.conf" to change sync parameters.

Use iburst to permit fast synchronization.

After changing the conf file restart the chronyd service.

``chronyc sources -v`` to see the servers you are synchronizing with.



## 1. Check Current Date and Time Settings
First, check the current date, time, and time synchronization settings:


``timedatectl``
This will show the current system time, local time, time zone, and if NTP (Network Time Protocol) synchronization is enabled.

#### 2. Change Date
To change the system date, use the following command:


``sudo timedatectl set-time YYYY-MM-DD``
For example, to set the date to October 9, 2024:


``sudo timedatectl set-time 2024-10-09``

#### 3. #Change Time
To set the system time, use:

``sudo timedatectl set-time HH:MM:SS``
For example, to set the time to 15:30:00 (3:30 PM):


``sudo timedatectl set-time 15:30:00``

#### 4. Set Both Date and Time Simultaneously
You can also set both date and time in a single command:


``sudo timedatectl set-time "YYYY-MM-DD HH:MM:SS"``
For example, to set the date to October 9, 2024, and the time to 15:30:00:


``sudo timedatectl set-time "2024-10-09 15:30:00"``

#### 5. Change Time Zone
To change the system’s time zone, first list all available time zones:

``timedatectl list-timezones``
Once you find the correct time zone (e.g., America/New_York), set it using:


``sudo timedatectl set-timezone Timezone``

For example, to set the time zone to New York:


``sudo timedatectl set-timezone America/New_York``
#### 6. Enable or Disable NTP Synchronization
To ensure your system's time is automatically synchronized with NTP servers, you can enable NTP:


``sudo timedatectl set-ntp true``
To disable NTP synchronization (if you want to manually set the date and time):


``sudo timedatectl set-ntp false``

#### 7. Verify Changes
After making changes, you can verify the current date, time, and time zone settings:


``timedatectl``
This will show the updated settings, including the current date, time, time zone, and NTP status.
