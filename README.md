# Hyperledger Composer Development Tutorial (MAC OSX)

Use these commands while following along with [Part 5 of the Hyperledger Development Series](https://www.youtube.com/playlist?list=PLYQSCk-qyTW3zBSNYmcQ62kv89kFaGydI)

## Prereqs (important!)

See [first video in series](https://www.youtube.com/watch?v=nS_MRqAeEbQ) or visit the [official Mac setup documentation for Composer](https://hyperledger.github.io/composer/installing/prereqs-mac.html)

Please note that if you have been following along with this tutorial series, you will need to run `nvm install --lts` again as Node has upgraded their long term support version.

## Environment Setup Commands

Download hyperledger client and tools (or `npm update` if you have already installed)

```
npm install -g composer-cli
npm install -g generator-hyperledger-composer
npm install -g composer-rest-server
npm install -g yo
```

Clean up your docker containers (careful if using docker containers for other projects!)

```
docker kill $(docker ps -q)
docker rm $(docker ps -aq)
docker rmi $(docker images dev-* -q)
```

Clean up any composer credentials and connection profiles you might have created previously (again, this assumes you don't have any important composer projects already started!)

```
rm -rf ~/.composer-connection-profiles
rm -rf ~/.composer-credentials
```

Start Hyperledger Fabric (this is what the business network connects to and is a series of Docker containers).

```
mkdir fabric-tools
cd fabric-tools
curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.zip
unzip fabric-dev-servers.zip
./downloadFabric.sh
./startFabric.sh
./createPeerAdminCard.sh
```

## Project Setup/Deployment Commands

Generate a template with Yo

```
yo hyperledger-composer:businessnetwork

> Business network name: perishable-network
> Description: A perishable tracking network
> Author name:  zach
> Author email: zach@email.com
> License: Apache-2.0
> Namespace: org.acme.shipping.perishable
```

This is based off of perishable sample network [found here](https://github.com/hyperledger/composer-sample-networks/tree/master/packages/perishable-network).

Create your `.bna` file

```
mkdir dist
composer archive create --sourceType dir --sourceName . -a ./dist/perishable-network.bna
```

Deploy the network

```
composer runtime install --card PeerAdmin@hlfv1 --businessNetworkName perishable-network
composer network start --card PeerAdmin@hlfv1 -A admin -S adminpw -a perishable-network.bna -f networkadmin.card
composer card import networkadmin.card
composer card list
composer network ping --card admin@perishable-network
```

## The REST server

To connect to REST server with authorization (Github), create new credentials for your application in github, and then put those credentials in the following environment variable and run it.

Install github-authentication

```
npm install -g passport-github
```

Export your COMPOSER_PROVIDERS env variable

```
export COMPOSER_PROVIDERS='{
  "github": {
    "provider": "github",
    "module": "passport-github",
    "clientID": "d25c49eac0dbb40f09b1",                    
    "clientSecret": "1041a7c497721de8e258b1a820d701664b07c698",               
    "authPath": "/auth/github",
    "callbackURL": "/auth/github/callback",
    "successRedirect": "/",
    "failureRedirect": "/"
  }
}'
```

Now, run the REST server with authentication

```
composer-rest-server -n perishable-network --card admin@perishable-network -a true -m true
```

You will need to import your identity card to use the REST server.