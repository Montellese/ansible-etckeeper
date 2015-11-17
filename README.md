ansible-etckeeper
=================
[![Build Status](https://travis-ci.org/chriswayg/ansible-etckeeper.svg?branch=master)](https://travis-ci.org/chriswayg/ansible-etckeeper)

Ansible role to install, configure, and use etckeeper.

This fork is a somewhat simplified version of the original role.



Requirements
------------

* Developed and tested with Ansible 1.4 - 1.9
* Debian/Ubuntu system (python-apt needed)

Role Variables
--------------

* **install** - *boolean* (default is **false**)

  This should be set as **true** in the first play.

* **commit** - *boolean* (default is **true**)

  Enables notification to "Record changes in etckeeper" handler,
  which generates an etckeeper commit for the play.
  Rather than setting this to false, just omit the etckeeper role entirely
  (unless this is the first play).

There is one variable (with default in ``defaults/main.yml``)
that control the commit messages used by etckeeper when *commit* is true:

* **etckeeper_message** - *string*
  (default is "changes from Ansible play running as {{ ansible_user_id }}")

  This message is used for commits that take place at the end of a play.

There is a configuration variable (with default in ``defaults/main.yml``)
that controls the version control system that etckeeper will use.

* **etckeeper_vcs** - *string*  (default is "git", or "hg", "bzr", or "darcs")

  This determines the version control system that etckeeper will use.
  Although the etckeeper package default is Mercurial ("hg"),
  this Ansible role has only been tested with Git.
  If etckeeper has already been installed, this variable has no effect.

There is a configuration variable (with example in ``defaults/main.yml``) 
for specific additional files to ignore by git:

* **etckeeper_gitignore** - *string*  (

Dependencies
------------

None.


Example Playbook
-------------------------

This role (and nothing else) should be the first play in a playbook,
and ideally you would run it as the very first play on any server,
since the first time it runs it generates an initial commit
with (nearly) the entire contents of the ``/etc/`` directory.

Adding the etckeeper role to other plays generates etckeeper commits
at the end of the play (and possibly the start, if there are unsaved changes).
For accurate unsaved change detection, the etckeeper role should be the first.
Using the etckeeper role, the etckeeper commits run in a handler,
(once) after all tasks, but possibly before some other handlers.

Running etckeeper commits in tasks allows more than one in a play,
but since all roles in a play run before any playbook tasks in the play,
breaking the playbook into many smaller plays with one role each
is still necessary to achieve finer-grained commits.

Here is an example playbook that uses the etckeeper role to perform
installations and commits, and also uses a shell action to perform commits.

    ---
    # This should be the first play in the playbook
    - hosts: all
      vars:
      - etckeeper_vcs: git
      roles:
      - { role: etckeeper, install: true }
      # Do not add any other roles to this play

    # This is the second play
    - hosts: all
      roles:
      - { role: etckeeper, etckeeper_message: '2nd play of playbook' }
      # additional roles here

    # This is the third play
    - hosts: all
      tasks:
      - name: Install and/or configure something
        command: cp /dev/null /etc/null
        register: result
      - name: Record changes for previous task in etckeeper commit
        shell: if etckeeper unclean; then etckeeper commit '3rd play pt. 1'; fi
        when: result|changed
      - name: Do something that doesn't change anything in /etc
        action: true
      - name: Do something else that could change something in /etc
        action: cp /dev/null /etc/zero
        notify: Record other changes for this play in etckeeper commit
     handlers:
     - name: Record other changes for this play in etckeeper commit
       shell: if etckeeper unclean; then etckeeper commit '3rd play pt. 2'; fi

License
-------

MIT (Expat) - see LICENSE file for details

Author Information
---------------------------

You can contact the original author at [alex.dupuy at mac.com](mailto:alex.dupuy%40mac.com).

This fork modified by Christian Wagner

