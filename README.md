# Phusion Passenger APT repository automation tools

This repository contains tools for automatically creating a multi-distribution APT repository for Phusion Passenger. It is meant to be run on Ubuntu 12.04.

First, install prerequisites and setup a user:

    sudo apt-get install ubuntu-dev-tools reprepro
    sudo adduser psg_apt_automation

Add this to /etc/sudoers:

    Cmnd_Alias PBUILDER_CREATE = /usr/sbin/pbuilder --create *
    Cmnd_Alias PBUILDER_UPDATE = /usr/sbin/pbuilder --update *
    Cmnd_Alias PBUILDER_BUILD = /usr/sbin/pbuilder --build *
    Cmnd_Alias PBUILDER=PBUILDER_CREATE,PBUILDER_UPDATE,PBUILDER_BUILD
    Defaults!PBUILDER env_keep="ARCHITECTURE DISTRIBUTION ARCH DIST DEB_BUILD_OPTIONS HOME"
    psg_apt_automation ALL=(root)NOPASSWD:PBUILDER

Now install the pbuilder distributions:

    sudo -u psg_apt_automation -H ./setup-pbuilder-dist

Configure your GPG signing settings:

    nano configuration
    echo your-gpg-key-passphrase > passphrase
    sudo chown psg_apt_automation:psg_apt_automation passphrase
    sudo chmod 600 passphrase

Then, every time a new Phusion Passenger version is released, run the following command to update the APT repository in `apt/`, as `psg_apt_automation`:

    ./new_release <GIT_URL> <REPO_DIR> <APT_DIR> [REF]

where:

 * `GIT_URL` is the Phusion Passenger git repository URL.
 * `REPO_DIR` is the directory that the git repository should be cloned to.
 * `APT_DIR` is the APT repository directory.
 * `REF` is the commit in git for which you want to build packages. If `REF` is not specified, then it is assumed to be `origin/master`.

For example:

    ./new_release https://github.com/phusion/passenger.git passenger.repo passenger.apt

The `new_release` script is near-atomic: it is very unlikely that users will see an intermediate state in which only some packages have been built.