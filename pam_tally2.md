#How to configure pam_tally2 to lock user account after certain number of failed login attempts ?

###0 Issue

- How to configure pam_tally2 to lock user account after certain number of failed login attempts ?

###Environment

- Red Hat Enterprise Linux 5
- Red Hat Enterprise Linux 6
- pam

###, Resolution

To configure pam_tally2 to lock a user account after certain number of failed login attempts, refer the steps below :

1.Add the following line in auth and account section of */etc/pam.d/system-auth* and */etc/pam.d/password-auth files*.

	auth        required      pam_tally2.so deny=3 onerr=fail unlock_time=500
	account     required      pam_tally2.so

2.The sample system-auth file will looks as follows :

	#%PAM-1.0
	# This file is auto-generated.
	# User changes will be destroyed the next time authconfig is run.
	auth        required      pam_env.so
	auth        required      pam_tally2.so deny=3 onerr=fail unlock_time=300
	auth        sufficient    pam_unix.so nullok try_first_pass
	auth        requisite     pam_succeed_if.so uid >= 500 quiet
	auth        required      pam_deny.so
	
	account     required      pam_tally2.so    
	account     required      pam_unix.so broken_shadow
	account     sufficient    pam_succeed_if.so uid < 500 quiet
	account     required      pam_permit.so
	
	password    requisite     pam_cracklib.so try_first_pass retry=3
	password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
	password    required      pam_deny.so
	
	session     optional      pam_keyinit.so revoke
	session     required      pam_limits.so
	session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
	session     required      pam_unix.so

The order of the pam rules are important. *auth required 
pam\_tally2.so* should be above of  *auth sufficient pam\_unix.so*.

On RHEL6, pam_tally2 entries needs to be present in both  *system-auth* and *password-auth* files.

3.The pam\_tally2 is not compatible with the old pam_tally faillog file format. By default the file that keeps the failed login counter is /var/log/tallylog.

Make sure tallylog permission is 600.

	# chmod 600 /var/log/tallylog ; chown root:root /var/log/tallylog

else It will log error message like below in  */var/log/secure*.

	var/log/secure:Nov 20 18:43:17 localhost login: pam_tally2(login:auth): /var/log/tallylog is either world writable or not a normal file
	var/log/secure:Nov 20 18:43:23 localhost login: pam_tally2(login:auth): /var/log/tallylog is either world writable or not a normal file

To check the list of users hitting maximum attempts command is "pam_tally2".

	# pam_tally2 

	# pam_tally2  -u testuser

To reset the number of fail login counter by following command.

	# pam_tally2 -r -u testuser

*If you want to lock root user, please add "even\_deny\_root" to the pam_tally2.so line in the auth section of the /etc/pam.d/system-auth file (and also the password-auth file if needed).

	auth        required      pam_tally2.so deny=3 onerr=fail unlock_time=60 even_deny_root
	account     required      pam_tally2.so

**Note:** *no\_magic\_root* option is not required to be configured in *pam_tally2* in RHEL 6 since normally, failed attempts to access root will not cause the root account to become blocked.

For more detail of pam_tally2:

/usr/share/doc/pam-{Version}/txts/README.pam_tally2