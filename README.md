# switchenv: An environment manger for bash
In my analysis work, I will frequently put authentification credentials into environment variables.  This allows me to check my analysis code into github without divulging any secrets.  So, for example, I might have code in a Jupyter notebook that looks something like
```python
import os
from my_module import get_database_connection

connection = get_database_connection(
    port=os.environ('PGPORT'),
    password=os.environ('PGPASSWORD'),
    user=os.environ('PGUSER'),
    database=os.environ('PGDATABASE'),
    host=os.environ('PGHOST'),
)
```
The database I connect to will be completely determined by the environment variables I have defined.

`switchenv` gives me a way to easily navigate between different environments so that, for example, I can quickly switch between development and production databases.

# Use Case
Imagine I have the following bash files I can source in order to set up my bash
environment the way I'd like.  Typically I would just run `source rc_development_db.sh` before running my code in order to set up my dev environmnt.  The problem with that is that I have to be very careful about where I place my rc files so I don't accidentally check them into github.  Furthermore, it can be annoying to keep track of these files when I'd like to reuse them for different projects.  This is where `switchenv` comes in.

`rc_development_db.sh`
```bash
export MY_MESSAGE="You are in dev profile"
export PGPORT=5432
export PGPASSWORD=my_dev_password
export PGUSER=my_dev_username
export PGDATABASE=my_dev_database
export PGHOST=my_dev_host
```

`rc_production_db.sh`
```bash
export MY_MESSAGE="You are in prod profile"
export PGPORT=5432
export PGPASSWORD=my_prod_password
export PGUSER=my_prod_username
export PGDATABASE=my_prod_database
export PGHOST=my_prod_host
```

`rc_bash_functions.sh`
```bash

print_message () {
   echo "Current message is ${MY_MESSAGE}"
}
```

# Setting up `switchenv`
Below is copy-paste from an interactive bash session showing how to set up the `switchenv` workflow.


```bash
bash>
bash> # Make sure I'm in a directory with the rc files I want
bash> ls *.sh
rc_development_db.sh  rc_production_db.sh  rc_bash_functions.sh
bash>
bash> # Load my rc scripts into switchenv giving them profile names
bash> switchenv add -p dev -f ./rc_development_db.sh
bash> switchenv add -p prod -f ./rc_production_db.sh
bash>
bash> # Profiles can hold any legal bash code, including function definitions.
bash> switchenv add -p functions -f ./rc_bash_functions.sh
bash>
bash> # Show list of stored profile names
bash> switchenv list
dev
prod
salesforce
bash>
bash> # Show contents of single profile
bash> switchenv show -p prod


#========================================
# prod
#========================================
export PGPORT=5432
export PGPASSWORD=my_prod_password
export PGUSER=my_prod_username
export PGDATABASE=my_prod_database
export PGHOST=my_prod_host
```

Under the hood, switchenv placed a json file in a hidden directory off of your home
directory.
```bash
~/.switchenv/profiles.json
```
This json file serves as the centralized data-store for all of your profile information.





Switchenv allows you to manage your bash environment with user-defined profiles. A pattern I frequently use in my analysis work is to have

## The way you do it now
Take as an example the case where you are doing interactive work that requires
you to be connected to either a production or development database.  Instead
of remembering your credentials, you have two bash scripts that you source.  These
scripts will populate environment variables so that your db client will
automatically connect without you having to manually specify credentials.  You also
occasionally need to download data from Salesforce and have a third file you can source for loading those credentials.  Examples of what those files might look like
are shown below.

`rc_development_db.sh`
```bash
export PGPORT=5432
export PGPASSWORD=my_dev_password
export PGUSER=my_dev_username
export PGDATABASE=my_dev_database
export PGHOST=my_dev_host
```

`rc_production_db.sh`
```bash
export PGPORT=5432
export PGPASSWORD=my_prod_password
export PGUSER=my_prod_username
export PGDATABASE=my_prod_database
export PGHOST=my_prod_host
```

`rc_salesforce.sh`
```bash
export SALESFORCE_SECURITY_TOKEN=my_long_sfdc_token
export SALESFORCE_PASSWORD=my_sfdc_password
export SALESFORCE_USERNAME=my_sfdc_username
```

Your current workflow might look something like
```bash
# Set up your credentials
source rc_development_db.sh
source rc_salesforce.sh

# Run a script that needs salesforce and db connection
python push_my_salesforce_stuff_to_db.py

# Run a notebook for interactively working with db and salesforce
jupyter notebook
```

This workflow is fine, and as long as you keep track of all the different
files you need to source for each task, it works pretty well.  `switchenv` is designed to make this workflow easier.

## The way you could be doing it.
Different components of your environment setup can be stored and sourced with
`switchenv`.  Invoking `switchenv` will drop you into a bash subshell with an environment that includes all specifications in that profile.  You can also compose profiles as we will show below.

### Set up profiles
First, define all the profiles you might care about
```
# Use the rc_developement script to define a profile named dev
switchenv add -p dev -f ./rc_development_db.sh

# Use the rc_production script to define a profile named dev
switchenv add -p prod -f ./rc_production_db.sh

# Define a salesforce profile
switchenv add -p salesforce -f ./rc_salesforce.sh
```

### Use profiles
```bash
bash>
bash> ls *.sh
rc_development_db.sh  rc_production_db.sh  rc_salesforce.sh
bash> # Use the rc_developement script to define a profile named dev
bash> switchenv add -p dev -f ./rc_development_db.sh
bash>
bash> # Use the rc_production script to define a profile named dev
bash> switchenv add -p prod -f ./rc_production_db.sh
bash>
bash> # Define a salesforce profile
bash> switchenv add -p salesforce -f ./rc_salesforce.sh
bash>
bash> # Show list of profile names
bash> switchenv list
dev
prod
salesforce
bash>
bash> # Show contents of single profile
bash> switchenv show -p prod


#========================================
# prod
#========================================
export PGPORT=5432
export PGPASSWORD=my_prod_password
export PGUSER=my_prod_username
export PGDATABASE=my_prod_database
export PGHOST=my_prod_host

bash>