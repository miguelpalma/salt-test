# Salt *state tree*

This repository is a test [Salt](http://www.saltstack.com/) *state tree* designed for basic admin
tasks.

## Install a development/test environment

1. Clone this repository **with the recursive flag** (`git clone --recursive …`) so that submodules
    in directory `formulas` are also cloned. If you need to individually init an already added
    submodule, use `git submodule update --init formulas/${FORMULA_REPO_NAME}`.

2. Start a virtual environment en SSH into it (next items will assume you are logged into this box).
    This Git repository contains a `Vagrantfile` that you can use by installing vagrant and then
    running:

    ```
    # Downloads a image from Vagrant servers (or the one you like: https://vagrantcloud.com/discover/featured
    # event though we will refer to a Fedora 20 base image in the rest of this document)
    vagrant box add chef/fedora-20
    vagrant up
    # SSH into the box with one of the following:
    vagrant ssh
    ssh vagrant@localhost -p 2222           # password: 'vagrant'
    ssh vagrant@localhost -p 2222 -i ~/.vagrant.d/insecure_private_key
    # User `vagrant` usually has `sudo` passwordless privileges so to become root:
    sudo -s
    # You should then update all installed packages before you do anything else:
    yum update
    ```

3. Install *salt minion*: `yum install salt-minion`

4. Edit `/etc/salt/minion` to set the following values (or just copy `/vagrant/conf/etc_salt_minion`
    to `/etc/salt/minion`. Better yet, remove `/etc/salt/minion` and link this path to
    `/vagrant/conf/etc_salt_minion`)

    ```
    # …
    file_client: local    # we are going to run masterless so the minion needs to know salt states are local
    # …
    file_roots:
      base:
        - /vagrant/salt/base
    # 'pillars' are salt files with custom values
    pillar_roots:
      base:
        - /vagrant/pillar/base
      dev:
        - /vagrant/pillar/dev
        - /vagrant/pillar/base
      prod:
        - /vagrant/pillar/prod
        - /vagrant/pillar/base
    # …
    # Optional – turn verbosity down:
    state_verbose: False
    state_output: terse
    ```

5. Optional: if you plan on contributing to this repository, set your Git name and email:

    ```
    git config user.name ${USERNAME}
    git config user.email ${EMAIL}
    ```

6. Optional: if you plan on wiping your VM to test your Salt tree, it would be wise to create a
    snapshot of your VM now so that you will not need to go through these steps anymore.
    [Vagrant can handle the job for VirtualBox](https://github.com/dergachev/vagrant-vbox-snapshot):

    ```
    vagrant plugin install vagrant-vbox-snapshot
    vagrant snapshot take fedora20_salt_updated
    ```

7. Place your own pillar files in directory `pillar` with the following hierarchy:

  - `pillar/*/admins.sls`: see `salt/formula/users-formula/pillar.example`
  - `pillar/*/openssh.sls`: see `salt/formula/openssh-formula/pillar.example`

8.  Salt uses the minion's id to determine which environment this minion belongs to (`dev`, `prod`).
    It is thus vital to set it in file `/etc/salt/minion_id` with some string starting with either
    `dev` or `prod` depending of course in which environment you are currently working on.

    FYI, environments this id will be matched against are declared in `salt/base/top.sls`.

9. Run Salt:

    ```
    # This directory is a mount of the Git directory on your local machine
    cd /vagrant
    salt-call state.highstate
    ```

## Adding a formula to use

```
git submodule add ${GIT_REPO} salt/formula/${APP_NAME}-formula
ln -s ../formula/${APP_NAME}-formula/${APP_NAME} salt/base/${APP_NAME}
```

## Updating a submodule to latest commit

```
cd salt/formula/${APP_NAME}-formula
git pull origin master
cd ../../..
git add salt/formula/${APP_NAME}-formula
git commit -m 'Set ${APP_NAME} to latest commit'
git submodule update salt/formula/${APP_NAME}-formula
```

## TODO

- services to add:
  - Link `/etc/localtime` to `/usr/share/zoneinfo/Europe/Paris` ??
  - ntp (use `timedatectl`??)
  - nagios
  - fail2ban
  - monit
    - system
    - sshd
    - fail2ban

- See how to automatically include formulas into top.sls as explained [in here](https://git.forgeservicelab.fi/jrodrigu/masterless-formulas/blob/e70e4e11d4c4ec1f66b1374d2335e745f4827bce/ml-wordpress/init.sls)
- See whether adding external formulas as Git submodules is a good idea as
  [this page from official Salt doc](http://docs.saltstack.com/en/latest/topics/best_practices.html)
  says:

  > Formulas should never be referenced from the main repository, and should be forked to a repo
  > where unintended changes will not take place.

- Formula `openssh-formula` only allow setup of one `Port`. Patch at https://github.com/saltstack-formulas/openssh-formula/blob/master/openssh/files/sshd_config#L51  ???
- Formula `openssh-formula` does not have `subsystem sftp …` in pillar. Patch?
- Make this conf common with monit? Is it possible?
- Formula monit: get *_host_* key paths found in sshd_config from openssh conf
- Add ability to ship .zshrc etc…
- Monit formula: add ability to specify other httpd login and connection options

## Usefull resources:

- [Google groups discussion on how to lay files out](https://groups.google.com/forum/#!msg/salt-users/k8tYoBQM_Fo/TtncOjwkBc4J)
- [Official doc: *pillar setup that enables to promote conf to environments*](http://docs.saltstack.com/en/latest/topics/tutorials/states_pt4.html#environment-configuration)

- if `nagios`:

    ```
    yum install
      nagios
      nagios-plugins-users
      nagios-plugins-load
      nagios-plugins-http
      nagios-plugins-ssh
    nagiosadmin/nagiosadmin
    ```
