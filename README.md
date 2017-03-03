# release_repo_mirror

This repository is a set of roles to mirror yum repositories into production.

## Getting Started

These roles are designed to take a new server and mirror our 4.1 release yum
repositories onto it.

Assumptions:
- target machine is a RHEL 7 (or CentOS 7) server
- if RHEL 7, target machine is already entitled
- target machine has sufficient space in `/var/www/html` to store the
repositories. Currently, each release takes about 42 GB of disk space for
all its repositories. Mounting a 1 TB volume there is probably a good start
- Ansible >= 2.2 is installed on the workstation being used to configure the
target machine

### QuickStart

The default configuration is designed to mirror all current 4.1 releases to
a target machine. To do this, once the assumptions are met, create an
Ansible inventory file defining the target server in `~/hosts`:
```
[mirror_servers]
mirror ansible_ssh_host=192.168.200.10
```

Then run: `ansible-playbook -i ~/hosts site.yml`

When the run finishes, the target `192.168.200.10` server should have all
current releases mirrored under `/var/www/html` and shared out via a very
basic Apache configuration for validation.

## Role overview

Primary envisioned use cases are to mirror either a specific release ("I want
the 4.1.8 release only") or to mirror all 4.1 releases. The site.yml has:
```
- hosts: mirror

  roles:
    - common
    - apache
    - { role: mirror_full_release, repo_version: '4.1.7-dev' }
    - { role: mirror_full_release, repo_version: '4.1.6' }
    - { role: mirror_full_release, repo_version: '4.1.5' }
    - { role: mirror_full_release, repo_version: '4.1.4' }
    - { role: mirror_full_release, repo_version: '4.1.3' }
    - { role: mirror_full_release, repo_version: '4.1.2' }
    - { role: mirror_full_release, repo_version: '4.1.1' }
```

Add or remove mirror_full_release entries with appropriate repo_version
definition to modify which releases get mirrored.

### common role

The common role ensures that basic commands needed for repo management are
installed.

### apache role

The apache role does a very basic install of Apache for validation of the
repositories once mirrored. Expectation is that you most likely have a more
sophisticated Apache role implementing auth and certificates as appropriate
for your site which should be used in place of this for production.

### mirror_full_release role

The mirror_full_release role defines each of the individual repositories
that make up a release. If an additional repository is added, this role
should be extended to include the extra repository. This role is also where
various properties of the repositories are defined, such as the GPG keys
which sign the RPMs inside each repository.

### mirror_repo role

The mirror_full_release role leverages the mirror_repo role, which does all
the actual work required to mirror a repo. It contains tasks to create
individual repo files, mirror the repo based on its repo file definition,
and then remove the repo file once mirrored. This role specifies the default
location of the repository source and should be customized to mirror from a
different location.

## Notes

You may notice that `yum clean all` is run frequently (before any addition
of a repo file, and after every removal of a repo file). Logically only the
clean after removal should be required but empirically both are needed.
Similarly, cleaning only metadata should be sufficient but worked
inconsistently.

