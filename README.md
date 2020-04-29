coffeesprout.sshd
=========

Ensure consistent settings for SSHD; FreeBSD and CentOS (7)
Please keep in mind that the defaults will deny password authentication and enforce other security settings. This may lock you out of your system if you're not careful.

Requirements
------------

Manages the SSH deamon. Expects SSHD to be installed and running

Role Variables
--------------

As you configure this role, having the [sshd_config(5)](https://www.freebsd.org/cgi/man.cgi?sshd_config(5)) manpage open might be helpful

    sshd_default_options:
        - name: "PasswordAuthentication"
          value: "{{ ssh_password_authentication | default('no') }}"
        - name: "PermitRootLogin"
          value: "{{ ssh_permit_root_login | default('no') }}"
        - name: "Port"
          value: "{{ sshd_port | default('22') }}"
        - name: "UseDNS"
          value: "{{ ssh_usedns | default('no') }}"
        - name: "PermitEmptyPasswords"
          value: "{{ ssh_permit_empty_password | default('no') }}"
        - name: "ChallengeResponseAuthentication"
          value: "{{ ssh_challenge_response_auth | default('no') }}"
        - name: "GSSAPIAuthentication"
          value: "{{ ssh_gss_api_authentication | default('no') }}"
        - name: "X11Forwarding"
          value: "{{ ssh_x11_forwarding | default('no') }}"
        - name: "Banner"
          value: "{{ ssh_banner_path | default('/etc/issue.net') }}"

These are some defaults that will always be set on every platform

    ssh_issue_template: issue.net
    
Configure which template is used to template /etc/issue.net; Defaults to the issue.net in templates/

    #Expects a list with name and optionally additional IP's; ssh_allow_ips are always allowed; Default allow all
    ssh_allow_users:
    - name: "*"

By default we allow all users, but it's relatively cheap to just list the users that are allowed. Optionally you can add a list of allow_ips to each user to add specific IP's / hostnames

    ssh_deny_users:
    - "root"

A list of denied users. Useful for quickly revoking and ensuring a specific user can never login, even when something else is misconfigured.

    sshd_custom_options: []
    
Allows a name / value pair with additional configuration you might want to add to sshd_config

    #List of IP's to always allow connections from. These will be added to each user; Default allow all
    ssh_allow_ips:
    - "*"

Useful for defining jumphosts or steppingstones.  
**Note: By overriding this you will lock down all access to the listed IP's (and those per user)**

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
      - { role: coffeesprout.sshd }

See also tests with examples

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
