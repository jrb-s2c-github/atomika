WITH THANKS TO Asif Mahmud, amended from https://github.com/asifmahmud/ansible-git-clone

Clone Private Git Repository
=========
An Ansible role that can be used to clone a private GitHub repository. 


Requirements
------------
* This role requires a GitHub personal access token. Generate a new token (Settings -> Developer Settings -> Personal Acess Token). Make sure the token has the scopes outlined in the "Role Variables" section below.

* Enter this access token when requested by Ansible

Role Variables
--------------

* key_title: The title of the SSH key to be added to the GitHub account
* key_path: Full path of the directory where the SSH key should be stored. A typical location would be `~/.ssh/<key_name>`
* github_access_token: The personal access token associated with the GitHub account being used. The token should have the following scopes: 
  - repo
  - admin:public_key
  - user
  - admin:gpg_ley
* git_repo: The SSH url of the GitHub repository to be cloned. An example would be `ssh://git@github.ibm.com/IBMZSoftware/nazare-demo-imsapp.git`
* clone_dest: The folder where repository should be cloned to
* known_hosts_path: Location of the SSH `known_hosts` file on the target machine. A typical location would be `~/.ssh/known_hosts`
* git_executable: The location of the git binary on the target machine. For example `/usr/bin/git`


Example Playbook
----------------
    - hosts: servers
      roles:
         - git-clone

License
-------

BSD

Author Information
------------------

Author: Asif Mahmud

Email: asif@mahmudasif.com
