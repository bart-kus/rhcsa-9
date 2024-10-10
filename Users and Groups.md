# Users and Groups

## Users 

### Adding users

``useradd john`` to create user john.

useradd johny -g kilo - adding johny with the suplementary grup kilo (groupadd kilo first)

[root@server2 ~]# id johny
uid=1241(johny) gid=1245(kilo) groups=1245(kilo)

file created by johny:

[johny@server2 ~]$ mkdir jj

[johny@server2 ~]$ ls -l

total 0

drwxr-xr-x. 2 johny kilo 6 Oct  9 23:00 jj


Checkout useradd --help to see more available options.

Files in **/etc/default/useradd** apply to useradd only.  
Alternatively, write default settings to **/etc/login.defs**. This is the main settings config file.

Changing this will not affect previously created users, only users that will be created in the future.

Files in **/etc/skel** are copied to the user home directory upon creation. If we want to send a message, scripts, etc. We can use **/etc/skel**.

## Managing users

### Passwords
To view password settings for user John.
``chage -l john``

To set password options for John.
``chage john``


[root@server2 ~]# chage johny
Changing the aging information for johny
Enter the new value, or press ENTER for the default

        Minimum Password Age [0]: 2y
chage: error changing fields
[root@server2 ~]# chage johny
Changing the aging information for johny
Enter the new value, or press ENTER for the default

        Minimum Password Age [0]: 2
        Maximum Password Age [99999]: 9
        Last Password Change (YYYY-MM-DD) [2024-10-09]:
        Password Expiration Warning [7]: 8
        Password Inactive [-1]: 5
        Account Expiration Date (YYYY-MM-DD) [-1]: 2024-12-24


The chage command is used to change the password aging information of a user in Linux. Here's what each of the fields in your command output means:

1. Minimum Password Age [0]: 2
This specifies the minimum number of days a user must wait before changing their password after it has been set or changed.
In this case, Johny will not be able to change his password until 2 days have passed since the last change.
2. Maximum Password Age [99999]: 9
This specifies the maximum number of days a password can be used before the user is required to change it.
Here, the maximum password age is set to 9 days, meaning Johny will need to change his password every 9 days.
3. Last Password Change (YYYY-MM-DD) [2024-10-09]:
This is the date when the user's password was last changed. You can either press Enter to accept the default value (current date: 2024-10-09) or enter a different date.
The value indicates that Johny’s password was last changed on October 9, 2024.
4. Password Expiration Warning [7]: 8
This is the number of days before the password expires that the user will receive a warning.
You’ve set this to 8 days, meaning Johny will be warned 8 days before his password expires (on the 1st day after setting the password, he'll get a warning).
5. Password Inactive [-1]: 5
This defines the number of days after the password has expired during which the user can still log in without changing their password.
A value of 5 means Johny can still log in and use the system for 5 days after the password expires, but after that, the account will be locked until the password is changed.
A value of -1 would mean there is no grace period.
6. Account Expiration Date (YYYY-MM-DD) [-1]: 2024-12-24
This is the date on which the user account will be disabled.
Here, the account will expire on December 24, 2024. After this date, Johny will no longer be able to log in, regardless of the password status.
A value of -1 means the account would never expire.
In summary, after the above changes:

Johny will have to wait 2 days to change his password after setting it.
His password will expire after 9 days.
He will be warned 8 days before his password expires.
After the password expires, he will have 5 days to log in before being locked out.
His account will be disabled on December 24, 2024.

You can also view the password options in **/etc/shadow**. You can see if the user account is locked out. The second field is the password hash. If the password hash starts with **!** The user account is locked out. As you can see, the user John is locked out.

![The shadow file](pictures/shadow.png)

You can also see if the account is locked with ``passwd -S armann`` 

If you want to transfer a password from another server to another one, simply copy the password hash in **/etc/shadow** from the server with the correct password and paste it into field number 2. 

To edit **/etc/passwd** use ``vipw``. Do not edit the file directly.

### User account management

To lock a user account.
``usermod -L john``

To unlock an account.
``usermod -U john``

See previous logged in users.
``last``

See currently logged in users.
``w`` or ``who``

### User file management

**/etc/login.defs**: Used for default settings like UID settings, passwd default settings, and other things.

**/etc/profile**: Used for default settings for all users when starting a login shell.

**/etc/bashrc**: Used to define defaults for all users when starting a subshell.

**~/.profile**: Specific settings for one user applied when starting a login shell.

**~/.bashrc**: Specific settings for one user applied when starting a subshell.

## Groups

To create a new group. ``groupadd groupname``

To see members of a group. ``groupmems -g sales`` or ``lid -g groupname``. For a specific user you can use ``id john`` or ``groups john``. The first group listed is the primary group.

Add John to the group sales. ``usermod -aG sales john``. The new group for the user is applied when they log out and back in. If they don't want to do that and use the new group right away, that's when we use the ``newgrp`` command. Remember to use the ``-a`` option when adding people to groups. If you don't, it will **override** all secondary groups the member is a part of.

Remove user John from the group printers.
``gpasswd -d john printers``

You can use ``newgrp sales`` to change the primary group to sales. This is only a **temporary** primary group change, when you exit, it's back to your original primary group. Remember that the ``newgrp`` command opens a subshell where the user is a member of the group sales.

Use ``vigr`` to change the **/etc/groups** file.  

## SUDO 

To have sudo rights the user needs to be a part of the wheel group.

To see current members of the wheel group.

``lid -g wheel``

Let's add John to the wheel group. ``usermod -aG wheel john``

Let's remove John from the wheel group. ``gpasswd -d john wheel``
