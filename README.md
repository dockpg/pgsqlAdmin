pgsqlAdmin
==========

pgsqlAdmin is forked from the https://git.postgresql.org/pgadmin4 project.

In the following documentation and examples, "$PGADMIN4_SRC/" is used to denote
the top-level directory of a copy of the pgsqlAdmin source tree, either from a
tarball or a git checkout.

Architecture
------------

pgsqlAdmin is written as a web application in Python, using jQuery and Bootstrap
for the client side processing and UI. On the server side, Flask is being
utilised.

Create Database Migrations
--------------------------

In order to make changes to the SQLite DB, navigate to the 'web' directory:

(pgsqladmin) $ cd $PGADMIN4_SRC/web

Create a migration file with the following command:

(pgsqladmin) $ FLASK_APP=pgsqlAdmin4.py flask db revision

This will create a file in: $PGADMIN4_SRC/web/migrations/versions/ .
Add any changes to the 'upgrade' function.
Increment the SCHEMA_VERSION in $PGADMIN4_SRC/web/pgadmin/model/__init__.py file.

There is no need to increment the SETTINGS_SCHEMA_VERSION.

Configuring the Python Environment
----------------------------------

In order to run the Python code, a suitable runtime environment is required.
Python versions - Python 2.7 or 3.5+ are currently supported. It is
recommended that a Python Virtual Environment is setup for this purpose,
rather than using the system Python environment. On Linux and Mac systems, the
process is fairly simple - adapt as required for your distribution:

1) Install the virtualenv packages into the system Python environment

    $ sudo pip install virtualenv virtualenvwrapper

2) Source the virtualenv wrapper tools script. You may want to add this command
   to your ~/.bash_profile file for future convenience:

    $ source /usr/local/bin/virtualenvwrapper.sh

3) Create a virtual environment:

    $ mkvirtualenv pgsqladmin

   To make use of the virtual environment in the future, use the following
   command to re-activate it:

    $ workon pgsqladmin

4) Ensure that a PostgreSQL installation's bin/ directory is in the path (so
   pg_config can be found for building psycopg2), and install the required
   packages:

    (pgsqladmin) $ PATH=$PATH:/usr/local/pgsql/bin pip install -r $PGADMIN4_SRC/requirements.txt

   If you are planning to run the regression tests, you also need to install
   additional requirements from web/regression/requirements.txt:

   (pgsqladmin) $ pip install -r $PGADMIN4_SRC/web/regression/requirements.txt

5) Create a local configuration file for pgsqlAdmin. Edit
   $PGADMIN4_SRC/web/config_local.py and add any desired configuration options
   (use the config.py file as a reference - any settings duplicated in
   config_local.py will override those in config.py). A typical development
   configuration may look like:

    from config import *

    # Debug mode
    DEBUG = True

    # App mode
    SERVER_MODE = True

    # Enable the test module
    MODULE_BLACKLIST.remove('test')

    # Log
    CONSOLE_LOG_LEVEL = DEBUG
    FILE_LOG_LEVEL = DEBUG

    DEFAULT_SERVER = '127.0.0.1'

    UPGRADE_CHECK_ENABLED = True

    # Use a different config DB for each server mode.
    if SERVER_MODE == False:
        SQLITE_PATH = os.path.join(
            DATA_DIR,
            'pgsqladmin-desktop.db'
        )
    else:
        SQLITE_PATH = os.path.join(
            DATA_DIR,
            'pgsqladmin-server.db'
        )

   This configuration allows easy switching between server and desktop modes
   for testing.

6) The initial setup of the configuration database is interactive in server
   mode,  and non-interactive in desktop mode. You can run it either by
   running:

    (pgsqladmin) $ python $PGADMIN4_SRC/web/setup.py

   or by starting pgsqlAdmin:

    (pgsqladmin) $ python $PGADMIN4_SRC/web/pgsqlAdmin4.py

   Whilst it is possible to automatically run setup in desktop mode by running
   the runtime, that will not work in server mode as the runtime doesn't allow
   command line interaction with the setup program.

At this point you will be able to run pgsqlAdmin from the command line in either
server or desktop mode, and access it from a web browser using the URL shown in
the terminal once pgsqlAdmin has started up.

Building the Web Assets
-----------------------

pgsqlAdmin is dependent on a number of third party Javascript libraries. These,
along with it's own Javascript code, SCSS/CSS code and images must be
compiled into a "bundle" which is transferred to the browser for execution
and rendering. This is far more efficient than simply requesting each
asset as it's needed by the client.

To create the bundle, you will need the 'yarn' package management tool to be
installed. Then, you can run the following commands on a *nix system to
download the required packages and build the bundle:

    (pgsqladmin) $ cd $PGADMIN4_SRC
    (pgsqladmin) $ make install-node
    (pgsqladmin) $ make bundle

Creating pgsqlAdmin themes
-----------------------

To create a pgsqlAdmin theme, you need to create a directory under
web/pgadmin/static/scss/resources.
Copy the sample file _theme.variables.scss.sample to the new directory and
rename it to _theme.variables.scss. Change the desired hexadecimal values of
the colors and bundle pgsqlAdmin. You can also add a preview image in the theme
directory with the name as <dir name>_preview.png. It is recommended that the
preview image should not be larger in size as it may take time to load on slow
networks. Run the yarn run bundle and you're good to go. No other changes are
required, pgsqlAdmin bundle will read the directory and create other required
entries to make them available in preferences.

The name of the theme is derived from the directory name. Underscores (_) and
hyphens (-) will be replaced with spaces and the result will be camel cased.

Configuring the Runtime
-----------------------

The pgsqlAdmin Runtime maintains it's own Python Path to avoid conflicts with
packages or other issues in the system Python installation. It will also search
a number of known locations for the pgsqlAdmin4.py file needed to run pgsqlAdmin
(including relative locations in a source code tree), however you can specify
an alternate path if needed.

If either a working environment or pgsqlAdmin4.py cannot be found at startup, the
runtime will prompt for the locations. Alternatively, you can click the try
icon and select the Configuration option to open the configuration dialogue.

On a Linux/Mac system, the Python Path will typically consist of a single path
to the virtual environment's site-packages directory, e.g.

    /Users/<USERNAME>/.virtualenvs/pgsqladmin/lib/python2.7/site-packages

If you wish to specify a specific copy of the Python code to run, you can set
the  Application Path to a directory containing pgsqlAdmin4.py, e.g.

    /Users/<USERNAME>/git/pgsqladmin-test/web/

Building the documentation
--------------------------

In order to build the docs, an additional Python package is required in the
virtual environment. This can be installed with the pip package manager:

    $ workon pgsqladmin
    (pgsqladmin) $ pip install Sphinx

The docs can then be built using the Makefile in $PGADMIN4_SRC, e.g.

    (pgsqladmin) $ make docs

The output can be found in $PGADMIN4_SRC/docs/en_US/_build/html/index.html

Building packages
-----------------

Most packages can be built using the Makefile in $PGADMIN4_SRC, provided all
the setup and configuration above has been completed.

To build a source tarball:

    (pgsqladmin) $ make src

To build a PIP Wheel, activate either a Python 2 or Python 3 virtual
environment as desired, configured with all the required packages, and then
run:

    (pgsqladmin) $ make pip
