dudefellah.rt
=========

Install and configure Request Tracker on your system.

Requirements
------------------------

CentOS 7:

I've allowed the option to install CPAN dependencies (see
`rt_cpan_modules` in [default/main.yml](default/main.yml) and/or
`__rt_cpan_modules` in [vars/main.yml](vars/main.yml)) to get installed in
a custom locallib path (see `rt_cpan_locallib`), but that option doesn't apply
to CentOS 7. The issue is that the cpanminus yum package will cause some older
Perl module dependencies to get installed, and that makes it difficult to do
some other things. So for this role, I've avoided installing cpanminus on my
own. I've made sure to fail the role if you attempt to use the
`rt_cpan_locallib` value in CentOS 7 to help avoid confusion. You may prefer
installing your CPAN dependencies globally anyway.

Also On CentOS 7, gnupg2 is only available up to version 2.0.x, but the
dependency module [GnuPG::Interface](https://metacpan.org/dist/GnuPG-Interface)
wants version 2.2+. In this case, you'll need to manually install a more recent
version of gnupg2. An example of how you might do this in your Ansible playbook
is available in the `Install gnupg2 2.2+` task in
[molecule/defaults/prepare.yml](molecule/defaults/prepare.yml).

If you're running _only_ gnupg1, then you should be able to get by with that,
since it's of the required version 1.4, but you will need to make sure that
/usr/bin/gpg (or whatever is coming back for `which gpg` from your installing
user) is that gpg1 binary. In other words, the installer isn't going to
specifically look for a binary called `gpg1`, and it's not going to distinguish
versions if you have another `gnupg2` installation using that `gpg` binary name.

Role Variables
--------------

The configurable values and their descriptions can be read as comments in
[defaults/main.yml](defaults/main.yml).
You might glean a little bit more information on the defaults that are
automatically determined for certain null values in
[vars/main.yml](vars/main.yml) as well. These values are partitioned by
distribution and version in a way that (I hope) should be obvious to the reader.

You'll also need to be somewhat careful about editing the `rt_cpan_modules`
value during your install. The existing module defaults (listed in
[vars/main.yml](vars/main.yml)) are there to make the `make fixdeps` part of
the install happy. The official RT documentation recommends running
`make fixdeps` multiple times to ensure that everything gets installed, but
I would prefer to not have this role run that command multiple times if I
can help it, hence the `rt_cpan_modules` value. So if you change the list of
modules in that array, you should check that you're not losing dependencies
that help the smooth (or as smooth as I've gotten it) installation flow.

I guess a more concise way to put this is that if you customize
`rt_cpan_modules`, you should still include what's listed in `vars/main.yml`,
and add to it unless you know what you're doing.

Dependencies
------------

* Due to a recent update/attempt to make ansible-lint happy, I've renamed
  tasks to their fully qualified collection name (fqcn).  This means that
  the following collections are required:

  * `ansible.posix`
  * `community.general`
  * `community.mysql` - this is only used for backups during version upgrades

* On CentOS 7, since we're not using cpanminus, you will need to have CPAN
  installed and configured before using this role.

* As mentioned in the requirements section, CentOS 7 systems will probably need
  an updated gpg2 version to make RT happy. This will need to be installed
  beforehand.

Example Playbook
----------------

The general flow would look something like this:

    - hosts: db_servers
      tasks:
        - block:
            - name: Install and configure the database
              ansible.builtin.include_role:
                name: dudefellah.postgresql
              vars:
                ...
          become: true

    - hosts: rt_servers
      tasks:
        - block:
            - name: Install RT
              ansible.builtin.include_role:
                name: dudefellah.rt
              vars:
                rt_version: 5.0.1
                ...

            - name: Configure Apache
              ansible.builtin.include_role:
                name: geerlingguy.apache
              vars:
                apache_vhosts: |
                  ...

                apache_vhosts_ssl: |
                  ...

License
-------

GPLv2+

Author Information
------------------

Dan - github.com/dudefellah
