myloginuser
===========

Least-privilege login user add-on SELinux policy module

This is a third-party SELinux policy module that aims to be a stand-alone add-on to Reference Policy.

This is an efford to bring enforcement of integrity to the user domain. This module should be installable on any
SELinux enabled system, although it currently provides only a small base that has been tested on Fedora 18 and Fedora 19
using gnome-shell, OpenSSH and console.

The goal is to confine as many user session processes as possible with the exception of core utilities, providing
enabling least-privilege login user sessions.

To provide flexible least privilege user sessions we need to establish what least privilege of each user session process
is and apply that. Only then we can provide customizable least-privilege user sessions.

The more user session processes are confined, the more functionality we can provide for restricted login user sessions.

There is currently no or very little integrity for user session processes provided by upstream Reference policy. Instead
a comprize is provided which provides little to no integrity in user sessions.

This policy module is designed to be as self-contained as possible. It does however depend on various types and
typeattributes provided by Reference policy, however the goal is to depend on upstream as little as possible.

The goal of this policy module is not to cripple user sessions. Instead the goal is to map the least privileges required
of as many user session processes as possible.

By doing this we provide a base for a broad set of scenarios for restricted user sessions by enabling end-users to
customize this base policy by making functionality conditional.

Were just mapping what user session processes need, and establishing rules accordingly. Our goal is to provide
user sessions that provide the same experience as any other user session environment, plus enforcement of mandatory 
integrity provided by the SELinux Flexible MAC framework and this policy module.

End-user can then modify this to meet their requirements.

Installing this policy module on any SELinux enabled system, but in particular Fedora 18 or Fedora 19 should not, in
any way break the existing policy characteristics. It aims to add extra functionality, andnot break existing
functionality.

To use this policy module:
==========================

1. Clone this repository and chage directory to the cloned repository
2. Build the policy module: make -f /usr/share/selinux/devel/Makefile myloginuser.pp
  Note that you may need to install the selinux-policy-devel package on some Redhat distributions.
  Note that other distributions may provide the makefile in different packages or paths.
3. Install the policy module: sudo semodule -i myloginuser.pp
4. Create a new login user: useradd myloginuser
5. Create a password for the new login user: passwd myloginuser
6. Optionally add the new login user to the wheel group: usermod -a -G wheel myloginuser
  Note this provides the new login user access to privileged user session applications via policykit.
  Note the only such privileged user session application supported is gnome logviewer.
7. Create default contexts:

  sudo cat /etc/selinux/targeted/contexts/users/myloginuser_u <<EOF
  myloginuser_r:myloginuser_t:s0            myloginuser_r:myloginuser_t:s0
  system_r:sshd_t:s0              myloginuser_r:myloginuser_t:s0
  system_r:local_login_t:s0       myloginuser_r:myloginuser_t:s0
  system_r:xdm_t:s0               myloginuser_r:myloginuser_t:s0
  EOF
  
  Note the above example is based on the targeted policy model situated at /etc/selinux/targeted.
8. Create a new SELinux identity mapping: sudo semanage user -a -L s0 -r s0 -R myloginuser_r -P user myloginuser_u
  Note Multi Compartment Security functionality is currently not supported for this user session.
9. Map the linux login user identity to the SELinux user identity: sudo semanage login -a -s myloginuser_u -r s0 myloginuser
10. Restore the security contexts of each path in the enclosed myloginuser.fc file using sudo restorecon -R -v -F "$PATH"
  Note alternately use: sudo fixfiles onboot && reboot.
  Note it is of vital importance that locations are labeled in accordance with the specifications provided in the
  enclose myloginuser.fc file.
11. Enable pam_sepermit for the login user: sudo echo "myloginuser" >> /etc/security/sepermit.conf
  Note this prevents the specified login user from login in if SELinux is not enforcing.
  Note requires pam_sepermit, and expects the pam_sepermit configuration file to be in /etc/security/sepermit.conf
  Note that SELinux is the first and last line of defence for this user, Therefore we must ensure that the user
  can only operate in SELinux enforced environments.

Currenlty this policy is tested in very limited environments ( Fedora 18/19 with gnome desktop) and with very limited
hardware (Intel and AMD GPU using non-proprietory drivers)

Again the goal is to support any SELinux enabled system that implements policy based off of Reference policy but
currently it is only tested with Fedora 18 and 19.

This policy is not for the faint hearted. Use it at your own risk.

The policy is self-contained as much as possible and thus you should be able to determine all the policy's
characteristics by just studying the myloginuser.te and myloginuser.fc files.
