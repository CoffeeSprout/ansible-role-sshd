coffeesprout.sshd
=========

![Molecule Tests](https://github.com/CoffeeSprout/ansible-role-sshd/actions/workflows/molecule.yml/badge.svg)

Ensure consistent settings for SSHD across FreeBSD, Debian, and RHEL-based systems.

**⚠️ Security Notice:** This role enforces secure defaults including:
- Password authentication disabled
- Root login disabled
- Restrictive user access controls

**These settings may lock you out if SSH keys are not properly configured.** Always test in a safe environment first.

Requirements
------------

- SSHD installed on target system
- SSH key-based authentication configured before applying restrictive settings
- Sudo/root access to modify SSH configuration

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

## Testing

This role includes comprehensive Molecule tests with 70+ verification checks covering:

### Test Categories

#### 1. **File Existence & Validation**
- Configuration file creation (`/etc/ssh/sshd_config`)
- Banner file creation (`/etc/issue.net`)
- Backup file creation
- Syntax validation using `sshd -t`

#### 2. **Security Hardening Verification**
- PasswordAuthentication disabled
- PermitRootLogin disabled
- UseDNS disabled
- X11Forwarding disabled
- PermitEmptyPasswords disabled
- ChallengeResponseAuthentication disabled
- GSSAPIAuthentication disabled

#### 3. **Negative Security Tests**
- Verify insecure options (e.g., `PasswordAuthentication yes`) are NOT present
- Verify `PermitRootLogin yes` is NOT configured
- Ensure configuration is restrictive, not permissive

#### 4. **User Access Control Tests**
- AllowUsers with IP restrictions (`admin@10.0.0.0/8`)
- AllowUsers with global IPs (`deploy@*`)
- AllowUsers combining specific + global IPs
- DenyUsers format and content
- Verify 6+ user@ip combinations generated correctly

#### 5. **Configuration Integrity Tests**
- No duplicate directives (Port, PasswordAuthentication, etc.)
- All custom options applied
- Banner content includes project name and legal warning
- File permissions (0644 for issue.net, root ownership)

#### 6. **Template Logic Tests**
- Users without specific IPs get global `ssh_allow_ips`
- Users with specific IPs get both specific AND global IPs
- Multiple user@ip combinations correctly generated
- All configured users present in final AllowUsers directive

#### 7. **Platform Support**
- Debian 12 (tested)
- AlmaLinux 9 (tested)
- FreeBSD 13/14 (vars configured, not CI tested)
- RHEL 8/9 (vars configured)

### Running Tests

```bash
# Activate the molecule virtual environment
source ../molecule-venv/bin/activate

# Navigate to role directory
cd coffeesprout.sshd

# Run full test suite (create → prepare → converge → idempotency → verify → destroy)
molecule test

# Development workflow
molecule create      # Start test containers
molecule converge    # Apply role to containers
molecule verify      # Run verification tests
molecule destroy     # Clean up containers

# Quick syntax check
molecule syntax
```

### Test Results

Latest test run: **70 tests passed** on both Debian 12 and AlmaLinux 9

- ✅ Configuration generation and validation
- ✅ Security hardening verification
- ✅ User access control logic
- ✅ Template rendering edge cases
- ✅ Idempotency (no changes on second run)
- ✅ File permissions and ownership
- ✅ Backup creation
- ✅ Negative security tests

### What's NOT Tested

Due to container limitations:
- **SSH service restart**: Tagged with `molecule-notest` (systemd unavailable in containers)
- **Actual SSH connections**: Requires running sshd service
- **FreeBSD jail support**: Requires FreeBSD host
- **Jumphost configuration**: Feature disabled in role (`ssh_config.yml` commented out)

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
