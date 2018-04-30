# CyberArk Conjur Open Source Lab

## Create a working directory
mkdir workshop
cd workshop

## Download docker-compose.yml file
curl -k -o docker-compose.yml https://www.conjur.org/get-started/eval/docker-compose.yml

## Start CLI client container - this may take a while
docker-compose run conjur

## Set “account” environment variable
account=“you@yourcompany.com”

## Initialize client resource files
conjur init -u https://eval.conjur.org -a ${account}

## Login as admin user – paste API key when prompted
conjur authn login -u admin

##  Download and load one-variable.yml
curl -k -o one-variable.yml https://www.conjur.org/get-started/eval/one-variable.yml

## Load the policy
conjur policy load root one-variable.yml

## Create a value:
secret_val=$(openssl rand -hex 12)

## Store the value in the secret defined by the “eval” policy:
conjur variable values add eval/secret ${secret_val}

## Fetch the value you just stored:
conjur variable value eval/secret

## Download and load variable-and-host.yml (from demo page)
conjur policy load root variable-and-host.yml | tee roles.json

## Get machine ID API key for login
api_key=$(jq -r '.created_roles | .[].api_key' roles.json')

## Authenticate & log in as the machine 
conjur authn login -u host/eval/machine -p ${api_key}

## Fetch the secret as the Machine ID you created
conjur variable value eval/secret

## Create a new value:
secret=$(openssl rand -hex 12)

## Now attempt to modify the secret as the Machine ID:
conjur variable values add eval/secret ${secret}