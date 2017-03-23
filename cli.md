1. Clone git repo for the dcos cli:

git clone git@github.com:dcos/dcos-cli.git

2. Change directory to the repo directory:

cd dcos-cli
git checkout 0.4.10

3. Make sure that you have virtualenv installed. If not type:

sudo pip install virtualenv

4. Create a virtualenv and packages for the dcos project:

make env
make packages

5. Create a virtualenv for the dcoscli project:

cd cli
make env

6. source the setup file to add the dcos command line interface to your PATH and create an empty configuration file:

source bin/env-setup-dev

7. Configure the CLI, changing the values below as appropriate for your local installation of DC/OS:

dcos config set core.dcos_url http://dcos-ea-1234.us-west-2.elb.amazonaws.com

8. Get started by calling the DC/OS CLI help:

dcos help

9. Running tests
cd dcos-cli/cli
tox

10. Changes to fix test failures
Failure 1 => - b"Permissions '0o644' for configuration file 'tests/data/dcos.toml' are too open. File must only be accessible by owner. Aborting...\n"
Fix => chmod 600 tests/data/dcos.toml

Failure 2 =>  AssertionError: assert 'http:/9.47.78.73' == 'http://dcos.snakeoil.mesosphere.com'
Fix =>  vi tests/integrations/test_config.py
Replace "dcos.snakeoil.mesosphere.com" to 9.47.78.73

Failure 3 => 
Fix => vi tests/data/dcos.toml
Replace "dcos.snakeoil.mesosphere.com" to 9.47.78.73
