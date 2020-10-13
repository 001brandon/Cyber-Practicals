### Practical Premise
The point of this practical assignment is all about logging. We need to set up a LAMP server and central logging for
that server in order to monitor it for malicious activity. We need two virtual machines to do this running ubuntu, one called myvm and another Drupal
We set up an Apache2 webserver and use MYSQL as it's database and Drupal as the user interactive webpage
sending the logs to a second Virtual Machine to monitor them and flag any malicious activity.

## Execution
Create a Drupal VM cloned from myvm - call it "drupal"
Install a web site with a login - drupal8
enable drupal logging in /var/log (also logs into database and can be viewed from the web account)
Configure rsyslog to send logs to specified IP
Finding unusual event in logs (discovery)
Monitor different types of logs (in /var/log) on local machine and central logs
Know the difference of auth.log vs syslog vs rsyslog etc.



1. SSH unknown user login in /var/log/auth.log in Drupal.

Oct 12 19:05:42 drupal sshd[1971]: Invalid user L33Thacker from 192.168.174.1 port 60630
Oct 12 19:05:46 drupal sshd[1971]: Failed password for invalid user L33Thacker from 192.168.174.1 port 60630 ssh2
Oct 12 19:05:46 drupal sshd[1971]: Failed password for invalid user L33Thacker from 192.168.174.1 port 60630 ssh2
Oct 12 19:05:51 drupal sshd[1971]: Failed password for invalid user L33Thacker from 192.168.174.1 port 60630 ssh2
Oct 12 19:05:51 drupal sshd[1971]: Connection closed by invalid user L33Thacker 192.168.174.1 port 60630 [preauth]

2. SSH good user bad password in /var/log/auth.log in Drupal

Oct 12 19:12:18 drupal sshd[2072]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.174.1  user=brooks
Oct 12 19:12:19 drupal sshd[2072]: Failed password for brooks from 192.168.174.1 port 60722 ssh2
Oct 12 19:12:26 drupal sshd[2072]: message repeated 2 times: [ Failed password for brooks from 192.168.174.1 port 60722 ssh2]
Oct 12 19:12:26 drupal sshd[2072]: Connection closed by authenticating user brooks 192.168.174.1 port 60722 [preauth]

3. Console bad user bad password log in /var/log/auth.log in Drupal

Oct 12 19:17:12 drupal login[945]: pam_unix(login:auth): authentication failure; logname=LOGIN uid=0 euid=0 tty=/dev/tty1 ruser= rhost=  user=hacker
Oct 12 19:17:15 drupal login[945]: FAILED LOGIN (1) on '/dev/tty1' FOR 'hacker', Authentication failure


4. Console good user bad password in /var/log/auth.log in Drupal

Oct 12 19:19:13 drupal login[1505]: pam_unix(login:auth): authentication failure; logname=LOGIN uid=0 euid=0 tty=/dev/tty1 ruser= rhost=  user=brooks
Oct 12 19:19:17 drupal login[1505]: FAILED LOGIN (1) on '/dev/tty1' FOR 'brooks', Authentication failure

5. Sudo baduser in /var/log/auth.log in Drupal

Oct 12 19:23:34 drupal systemd-logind[870]: New session 7 of user hacker.
Oct 12 19:23:43 drupal sudo:   hacker : user NOT in sudoers ; TTY=pts/1 ; PWD=/home/hacker ; USER=root ; COMMAND=/bin/ls -a

6. Sudo bad password in /var/log.auth.log in Drupal

Oct 12 19:26:14 drupal sudo: pam_unix(sudo:auth): authentication failure; logname=brooks uid=1000 euid=0 tty=/dev/pts/0 ruser=brooks rhost=  user=brooks
Oct 12 19:26:20 drupal sudo:   brooks : TTY=pts/0 ; PWD=/home/brooks ; USER=root ; COMMAND=/bin/ls -l
Oct 12 19:26:20 drupal sudo: pam_unix(sudo:session): session opened for user root by brooks(uid=0)
Oct 12 19:26:20 drupal sudo: pam_unix(sudo:session): session closed for user root

7. Drupal Bad user

Oct 12 22:02:45 drupal drupal: http://192.168.174.132|1602540165|user|192.168.174.1|http://192.168.174.132/user/login|http://192.168.174.132/user/login|0||Login attempt failed from 192.168.174.1.

8. Drupal Bad Password

Oct 12 22:03:04 drupal drupal: http://192.168.174.132|1602540184|user|192.168.174.1|http://192.168.174.132/user/login|http://192.168.174.132/user/login|0||Login attempt failed for brooks.



9. Drupal system log in and log out as seen in myvm in /var/log/syslog

brooks@myvm:~$ tail -f /var/log/syslog
Oct 12 18:44:08 drupal systemd[1]: Stopped System Logging Service.
Oct 12 18:44:08 drupal systemd[1]: Starting System Logging Service...
Oct 12 18:44:08 drupal systemd-resolved[776]: Server returned error NXDOMAIN, mitigating potential DNS violation DVE-2018-0001, retrying transaction with reduced feature level UDP.
Oct 12 18:44:08 drupal systemd-resolved[776]: Server returned error NXDOMAIN, mitigating potential DNS violation DVE-2018-0001, retrying transaction with reduced feature level UDP.
Oct 12 18:44:09 drupal systemd[1]: Started System Logging Service.
Oct 12 18:44:09 drupal rsyslogd: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3) from systemd.  [v8.32.0]
Oct 12 18:44:09 drupal rsyslogd: rsyslogd's groupid changed to 106
Oct 12 18:44:09 drupal rsyslogd: rsyslogd's userid changed to 102
Oct 12 18:44:09 drupal rsyslogd:  [origin software="rsyslogd" swVersion="8.32.0" x-pid="1590" x-info="http://www.rsyslog.com"] start
Oct 12 18:44:34 myvm systemd[1]: Started Session 3 of user brooks.



Oct 12 18:48:49 drupal drupal: http://192.168.174.132|1602528529|user|192.168.174.1|http://192.168.174.132/user/login|http://192.168.174.132/user/login|0||Login attempt failed from 192.168.174.1.
Oct 12 18:49:07 drupal drupal: http://192.168.174.132|1602528547|user|192.168.174.1|http://192.168.174.132/user/login|http://192.168.174.132/user/login|0||Login attempt failed for brooks.
Oct 12 18:49:14 drupal drupal: http://192.168.174.132|1602528554|user|192.168.174.1|http://192.168.174.132/user/login|http://192.168.174.132/user/login|1||Session opened for brooks.
Oct 12 18:49:51 drupal drupal: http://192.168.174.132|1602528591|user|192.168.174.1|http://192.168.174.132/user/logout|http://192.168.174.132/user/1|1||Session closed for brooks.


