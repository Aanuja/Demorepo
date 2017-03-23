Clone git repo for the dcos cli:

git clone git@github.com:dcos/dcos-cli.git
Change directory to the repo directory:

cd dcos-cli
git checkout 0.4.10

Make sure that you have virtualenv installed. If not type:

sudo pip install virtualenv
Create a virtualenv and packages for the dcos project:

make env
make packages
Create a virtualenv for the dcoscli project:

cd cli
make env

source the setup file to add the dcos command line interface to your PATH and create an empty configuration file:

source bin/env-setup-dev
Configure the CLI, changing the values below as appropriate for your local installation of DC/OS:

dcos config set core.dcos_url http://dcos-ea-1234.us-west-2.elb.amazonaws.com
Get started by calling the DC/OS CLI help:

dcos help

cd dcos-cli/cli
tox
