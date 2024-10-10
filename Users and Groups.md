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

In RHCSA 9, locking a user account can be done in a few ways using different commands. Locking a user prevents them from logging in to the system without deleting the account. Here's how you can lock a user account in RHEL 9:

1. Using passwd Command
The easiest and most common way to lock a user account is by using the passwd command with the -l (lock) option.

bash
Copy code
sudo passwd -l username
For example, to lock the user johny:

bash
Copy code
sudo passwd -l johny
This will place a ! character in front of the encrypted password in /etc/shadow, which effectively disables the password and prevents the user from logging in.

2. Using usermod Command
Another way to lock a user account is by using the usermod command with the -L (lock) option.

bash
Copy code
sudo usermod -L username
Example:

bash
Copy code
sudo usermod -L johny
This command also works by placing a ! in front of the encrypted password in /etc/shadow.

3. Temporarily Locking with chage Command
The chage command can also be used to set the account expiration date to a past date, effectively locking the account.

bash
Copy code
sudo chage -E 0 username
Example:

bash
Copy code
sudo chage -E 0 johny
This will set the expiration date of the user johny to "0", meaning the account is expired and can't be used to log in.

4. Manually Editing /etc/shadow File
As a last resort, you can manually edit the /etc/shadow file to lock the account.

Open the /etc/shadow file with a text editor:

bash
Copy code
sudo vi /etc/shadow
Locate the user’s entry (e.g., johny) and place a ! in front of the hashed password:

perl
Copy code
johny:!$6$abcdefgh....:...
Verifying the Account Lock Status
You can verify if the account is locked by checking the user status with the passwd -S command:

bash
Copy code
passwd -S johny
If the output shows L (Locked), the account is locked.

Unlocking the Account
To unlock a user account, you can use:

passwd -u username
usermod -U username
For example:

bash
Copy code
sudo passwd -u johny
or

bash
Copy code
sudo usermod -U johny
These methods should cover how to lock a user account in RHEL 9 (RHCSA 9).


usermod -L john
ChatGPT said:
ChatGPT
The command usermod -L john is used to lock the user account named john in Linux. When you run this command, it adds a ! in front of the user's encrypted password in the /etc/shadow file, preventing the user from logging in with their password.

Breakdown of the Command:
usermod: The command to modify user account settings.
-L: Option to lock the user account by disabling the password.
john: The username of the account you want to lock.
What Happens:
The user's password is "disabled," meaning they won't be able to log in using their password until the account is unlocked.
Other login methods (like SSH keys) may still work unless you disable them separately.
The lock does not delete or disable the account itself, it only prevents password-based login.
Example:
bash
Copy code
sudo usermod -L john
Verifying the Lock:
You can check if the user is locked by using the following command:

bash
Copy code
passwd -S john
If the user is locked, the output will show L (Locked).
Unlocking the Account:
To unlock the account later, you can use the -U option:

bash
Copy code
sudo usermod -U john
This removes the ! from the /etc/shadow file and allows the user to log in with their password again.

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

## change the user suplementary group:

Change the User’s GID: Once the group with GID 1245 exists, you can change the user's primary group to that GID:

sudo usermod -g 1245 username

if user is assignet to more that one gorup but you want to change the suplementary group you need to check ``id`` of the user and find the number of lets say third gropup ``groups=1246(kilos)``

then ownership on second group will be changed -rw-r--r--. 1 johny kilo 0 Oct  9 23:36 aa


## SUDO 

To have sudo rights the user needs to be a part of the wheel group.

To see current members of the wheel group.

``lid -g wheel``

Let's add John to the wheel group. ``usermod -aG wheel john``

Let's remove John from the wheel group. ``gpasswd -d john wheel``
