## CyberArk Conjur Open Source Lab


# 1) Create an account:

## In your browser:
 - Go to https://www.conjur.org/get-started/try-conjur.html
 - Create an account.
 - Stay on this page when the account info is displayed.
 - You will need the Account ID and API key.


# 2) Create the CLI container:

## Create a working directory
- mkdir workshop
- cd workshop

## Download docker-compose.yml file
- curl -k -o docker-compose.yml https://www.conjur.org/get-started/eval/docker-compose.yml

## Start CLI client container - this may take a while
- docker-compose run conjur


# 3) Initialize the client environment

## Initialize client resource files
- conjur init -u https://eval.conjur.org -a \<Account-ID-from-browser\>

## Login as admin user – paste API key when prompted
- conjur authn login -u admin


# 4) Create and load a policy that defines a secret

##  Download and load one-variable.yml
- curl -k -o one-variable.yml https://www.conjur.org/get-started/eval/one-variable.yml

## Load the policy
- conjur policy load root one-variable.yml


# 5) Store and fetch a secret according to policy

## Create a value:
- secret_val=$(openssl rand -hex 12)

## Store the value in the secret defined by the “eval” policy:
- conjur variable values add eval/secret ${secret_val}

## Fetch the value you just stored:
- conjur variable value eval/secret


# 6) Create a machine identity

## Download and load variable-and-host.yml (from demo page)
- conjur policy load root variable-and-host.yml | tee roles.json

## Get machine ID API key for login
- api_key=$(jq -r '.created_roles | .[].api_key' roles.json')

## Authenticate & log in as the machine 
- conjur authn login -u host/eval/machine -p ${api_key}


# 7) Machine Identity authentication and authorization

## Fetch the secret as the Machine ID you created
- conjur variable value eval/secret

## Create a new value:
- secret=$(openssl rand -hex 12)

## Now attempt to modify the secret as the Machine ID:
- conjur variable values add eval/secret ${secret}
