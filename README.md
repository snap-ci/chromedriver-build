[![Build Status](https://snap-ci.com/snap-ci/chromedriver-build/branch/master/build_image)](https://snap-ci.com/snap-ci/chromedriver-build/branch/master)

This repo contains scripts needed to build deb and rpms for various versions of herokutoolbelt-build. To file an issue, [please contact the Snap.ci support team.](https://snap-ci.com/contact-us)

# CentOS/RHEL

<pre>
$ yum install -y rpm-build rpmdevtools readline-devel ncurses-devel gdbm-devel tcl-devel openssl-devel db4-devel byacc
$ bundle install --path .bundle
</pre>

# Ubuntu

<pre>
$ apt-get install build-essentials dpkg-dev
$ bundle install --path .bundle
</pre>

# How to build one of the packages

<pre>
$ rake
</pre>
