# __**CyberArk Conjur Open Source Lab**__
---

# 1) Create an account:

## \<In a browser window\>

## Go to https://www.conjur.org/get-started/try-conjur.html
 - Create an account.
 - Leave this page open when the account info is displayed.
 - You will need the Account ID and API key.


# 2) Create the CLI container:

## \<In a shell window\>

## Create a working directory
````
mkdir workshop
cd workshop
````

## Download docker-compose.yml file
````
curl -k -o docker-compose.yml https://www.conjur.org/get-started/eval/docker-compose.yml
````
 
## Start CLI client container - this may take a while
````
docker-compose run conjur
````


# 3) Initialize the client environment

## Initialize client resource files
````
# note escaped special char in account name
conjur init -u https://eval.conjur.org -a foo\!@bar.com
````

## Login as admin user – paste API key when prompted (you won't see it)
````
conjur authn login -u admin
````

## List objects in Conjur 
````
conjur list
````


# 4) Create and load a policy that defines a secret

##  Download the one-variable.yml policy file
````
curl -k -o one-variable.yml https://www.conjur.org/get-started/eval/one-variable.yml
````

## Load the policy
````
conjur policy load root one-variable.yml
````

## List objects in Conjur now
````
conjur list
````


# 5) Store and fetch a secret according to policy

## Create a value:
````
secret_val=$(openssl rand -hex 12); echo $secret_val
````

## Store the value in the secret defined by the “eval” policy:
````
conjur variable values add eval/secret ${secret_val}
````

## Fetch the value you just stored:
````
conjur variable value eval/secret; echo
````


# 6) Create a machine identity

## Download the variable-and-host.yml policy
````
curl -k -o variable-and-host.yml https://www.conjur.org/get-started/eval/variable-and-host.yml
````

## Look at permissions for Machine ID - verify no update permission
````
cat variable-and-host.yml
````

## Load the variable-and-host.yml policy - note version number increments
````
conjur policy load root variable-and-host.yml | tee roles.json
````

## Get machine ID API key for login
````
api_key=$(jq -r '.created_roles | .[].api_key' roles.json)
````

## Authenticate & log in as the machine 
````
conjur authn login -u host/eval/machine -p ${api_key}
````

## List objects in Conjur now
````
conjur list
````


# 7) Machine Identity authentication and authorization

## Fetch the secret as the Machine ID you created
````
conjur variable value eval/secret; echo
````

## Create a new value:
````
secret_val=$(openssl rand -hex 12); echo $secret_val
````

## Now attempt to modify the secret as the Machine ID:
````
conjur variable values add eval/secret ${secret_val}
````

## Look at permissions for Machine ID - verify no update permission
````
cat variable-and-host.yml
````


# 8) Using Summon for secrets injection into variables

## Retrieve the secret with Summon into an environment variable
````
summon --yaml 'SECRET: !var eval/secret' bash -c "echo \$SECRET"
````

## Show that the value is ephemeral
````
echo $SECRET
````

## Retrieve the secret with Summon into a memory mapped file
````
summon --yaml 'SECRET: !var:file eval/secret' bash -c "echo \$SECRET; cat \$SECRET; echo"
````

## Show that the file is ephemeral
````
cat $SECRET
````

---

# Next steps:
## Try running the Conjur server locally:
- https://www.conjur.org/get-started/install-conjur.html
## Check out Summon:
- https://cyberark.github.io/summon/

