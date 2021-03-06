# Kiwi Fabric Network Setup



## Generate crypto material command
./bin/cryptogen generate --config=./crypto-config.yaml

## Generate genesis block 
./bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

## Generate Channel tx
./bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID kiwi-channel

## Update Peer Anchors for org1 and org2


# Starting the Network

'docker-compose -f docker-compose-cli.yaml up'


# Starting the Network

'docker-compose -f docker-compose-cli.yaml up'


## Devmode

### Setting up Channel

1. Get the channel.block from the network orderer

`peer channel create -o orderer.kiwi.com:7050 -c kiwi-channel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/kiwi.com/orderers/orderer.kiwi.com/msp/tlscacerts/tlsca.kiwi.com-cert.pem`

2. Join the channel
`peer channel join -b kiwi-channel.block`

3. Add the other peer from Org2 to the channel
`CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.kiwi.com/users/Admin@org2.kiwi.com/msp CORE_PEER_ADDRESS=peer0.org2.kiwi.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.kiwi.com/peers/peer0.org2.kiwi.com/tls/ca.crt peer channel join -b kiwi-channel.block`

4. Add the rest of the peers by adjusting the numbers accordingly

5. Update anchor peers
    -  This command updates the anchor peer for org1.
    `peer channel update -o orderer.kiwi.com:7050 -c kiwi-channel -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/kiwi.com/orderers/orderer.kiwi.com/msp/tlscacerts/tlsca.kiwi.com-cert.pem`
    - The second command updates the anchor peer for org2
    `CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.kiwi.com/users/Admin@org2.kiwi.com/msp CORE_PEER_ADDRESS=peer0.org2.kiwi.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.kiwi.com/peers/peer0.org2.kiwi.com/tls/ca.crt peer channel update -o orderer.kiwi.com:7050 -c kiwi-channel -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/kiwi.com/orderers/orderer.kiwi.com/msp/tlscacerts/tlsca.kiwi.com-cert.pem`



## Terminal 2: CLI container 
Go back to the CLI container terminal window

1. Install Chaincode
`peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/cc`

2. Instantiate chaincode
`peer chaincode instantiate --tls -o orderer.kiwi.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/kiwi.com/orderers/orderer.kiwi.com/msp/tlscacerts/tlsca.kiwi.com-cert.pem -C kiwi-channel -n mycc -v 1.0 -c '{"Args":["init","1"]}' -P "AND ('Org1MSP.peer')"`

    This command will instantiate the chaincode to the channel so that it can be accesible by other peers

3. Invoke Chaincode (test if a person can be added)
`peer chaincode invoke -o orderer.kiwi.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/kiwi.com/orderers/orderer.kiwi.com/msp/tlscacerts/tlsca.kiwi.com-cert.pem -C kiwi-channel -n mycc3  -c '{"Args":["addPerson","Bob","asd123","Apple","12343.13","[1,3]"]}'`

4. Test business 
`peer chaincode invoke -o orderer.kiwi.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/kiwi.com/orderers/orderer.kiwi.com/msp/tlscacerts/tlsca.kiwi.com-cert.pem -C kiwi-channel -n mycc3  -c '{"Args":["addBusiness","Apple","bus1234","[asd123,qew132]","{}","1300000000000.00"]}'`

5. Query for person
`peer chaincode query -o orderer.kiwi.com:7050 -C kiwi-channel -n mycc  -c '{"Args":["queryPerson","asd123"]}'`





# Testing the Node SDK and interacting with the network

The commands are taken from the balance-transfer folder from fabric-samples and have been modified to interact with our network.

## Open a new terminal for running the node application

Navigate to the  directory that contains app.js and then start the node application on port 4000:

`PORT=4000 node app`

## Open another terminal for invoking the node commands to interact with the network

## Login Request

### Enrolling a user for Org1

The following command registers and enrolls a user with the name `User` for an Organization, in this case Org1:
`curl -s -X POST   http://localhost:4000/users   -H "content-type: application/x-www-form-urlencoded" -d 'username=UserOrg1&orgName=Org1'`

This should Output
```
{
    "success":true,
    "secret":"",
    "message":"UserOrg1 enrolled Successfully",
    "token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mzc3NDIwMDUsInVzZXJuYW1lIjoiVXNlck9yZzEiLCJvcmdOYW1lIjoiT3JnMSIsImlhdCI6MTUzNzcwNjAwNX0.j_VlemeawMG79sWiqmdTwLVizLlH9QdQwgsuXLd5OIA"
}
```
The token value will be unique and not the same as above.

The response contains the success/failure status, an **enrollment Secret** and a **JSON Web Token (JWT)** that is a required string in the Request Headers for subsequent requests. 

### Enrolling a user for Org12

The following command registers and enrolls a user with the name `User` for an Organization, in this case Org1:
`curl -s -X POST   http://localhost:4000/users   -H "content-type: application/x-www-form-urlencoded"   -d 'username=UserOrg2&orgName=Org2'`

This should Output
```
{
    "success":true,
    "secret":"",
    "message":"UserOrg2 enrolled Successfully",
    "token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mzc3NDA4MTcsInVzZXJuYW1lIjoiVXNlck9yZzIiLCJvcmdOYW1lIjoiT3JnMiIsImlhdCI6MTUzNzcwNDgxN30.KzdIhyNFaLh9lDMVVuceWI69jnMYmVbzVoXaG4qTwpE"
}
```


## Channel Configuration
### Create Channel Request
```
curl -s -X POST \
  http://localhost:4000/channels \
  -H "authorization: Bearer " \
  -H "content-type: application/json" \
  -d '{
	"channelName":"kiwi-channel",
	"channelConfigPath":"../artifacts/channel/channel-artifacts/channel.tx"
}'
```

### Request for Org1 to Join Channel 
```
curl -s -X POST \
  http://localhost:4000/channels/kiwi-channel/peers \
  -H "authorization: Bearer " \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org1.kiwi.com","peer1.org1.kiwi.com"]
}'
```

### Request for Join Channel on Org2 (Not necessary for testing)
```
curl -s -X POST \
  http://localhost:4000/channels/kiwi-channel/peers \
  -H "authorization: Bearer <Token Value Org2>" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org2.kiwi.com","peer1.org2.kiwi.com"]
}'
```


## Chaincode Interactions

### Install Chaincode on Org1 Peers
```
curl -s -X POST \
  http://localhost:4000/chaincodes \
  -H "authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mzc3NDIwMDUsInVzZXJuYW1lIjoiVXNlck9yZzEiLCJvcmdOYW1lIjoiT3JnMSIsImlhdCI6MTUzNzcwNjAwNX0.j_VlemeawMG79sWiqmdTwLVizLlH9QdQwgsuXLd5OIA" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org1.kiwi.com","peer1.org1.kiwi.com"],
	"chaincodeName":"mycc",
	"chaincodePath":"chaincode/cc",
	"chaincodeVersion":"1.0",
    "chaincodeType":"golang"
}'
```

### Install Chaincode on Org2 Peers
```
curl -s -X POST \
  http://localhost:4000/chaincodes \
  -H "authorization: Bearer <Token Value UserOrg2>" \
  -H "content-type: application/json" \
  -d '{
	"peers": ["peer0.org2.kiwi.com","peer1.org2.kiwi.com"],
	"chaincodeName":"mycc",
	"chaincodePath":"chaincode/cc",
	"chaincodeVersion":"1.0",
    "chaincodeType":"golang"
}'
```

## Instantiate Chaincode
```
curl -s -X POST \
  http://localhost:4000/channels/kiwi-channel/chaincodes \
  -H "authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mzc3MzkyMDgsInVzZXJuYW1lIjoiVXNlck9yZzEiLCJvcmdOYW1lIjoiT3JnMSIsImlhdCI6MTUzNzcwMzIwOH0.CB1ji0ltD-xo4gB8DL59X3psmA2jrGKyQNhNByrCd-w" \
  -H "content-type: application/json" \
  -d '{
    "peers": ["peer0.org1.kiwi.com","peer1.org1.kiwi.com"],
    "chaincodeName":"mycc",
    "chaincodeVersion":"1.0",
    "chaincodeType":"golang",
    "args":["init", "a"]
  }'

curl -s -X POST \
  http://localhost:4000/channels/kiwi-channel/chaincodes \
  -H "authorization: Bearer " \
  -H "content-type: application/json" \
  -d '{
  "peers": ["peer0.org1.kiwi.com","peer1.org1.kiwi.com"],
	"chaincodeName":"mycc",
	"chaincodeVersion":"1.0",
  "chaincodeType":"golang",
	"args":["init", "a"]
}'  
```

## Invoke Chaincode
```
curl -s -X POST \
  http://localhost:4000/channels/kiwi-channel/chaincodes/mycc \
  -H "authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mzc3Mzk4ODIsInVzZXJuYW1lIjoiVXNlck9yZzEiLCJvcmdOYW1lIjoiT3JnMSIsImlhdCI6MTUzNzcwMzg4Mn0.NugsaVt--jKlCE5NGo78NRatWlrwp0RsXDdS1TasYng" \
  -H "content-type: application/json" \
  -d '{
    "peers": ["peer0.org1.kiwi.com","peer1.org1.kiwi.com"],
	"fcn":"addBusiness",
	"args":["Apple","bus1234","["asd123","qew132"]","{}","1300000000000.00"]
}'


curl -s -X POST \
  http://localhost:4000/channels/kiwi-channel/chaincodes/mycc \
  -H "authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mzc3NDIwMDUsInVzZXJuYW1lIjoiVXNlck9yZzEiLCJvcmdOYW1lIjoiT3JnMSIsImlhdCI6MTUzNzcwNjAwNX0.j_VlemeawMG79sWiqmdTwLVizLlH9QdQwgsuXLd5OIA" \
  -H "content-type": "application/json" \
  -d '{
    "peers": ["peer0.org1.kiwi.com","peer1.org1.kiwi.com"],
	"fcn":"addBusiness",
	"args":["Apple","bus1234","[asd123,qew132]","{}","1300000000000.00"]
}'
```

## Query Chaincode by Name

curl -s -X GET \
  "http://localhost:4000/channels/kiwi-channel/chaincodes/mycc?peer=peer0.org1.kiwi.com&fcn=query&args=%5B%22asd%22%5D" \
  -H "authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mzc3NDIwMDUsInVzZXJuYW1lIjoiVXNlck9yZzEiLCJvcmdOYW1lIjoiT3JnMSIsImlhdCI6MTUzNzcwNjAwNX0.j_VlemeawMG79sWiqmdTwLVizLlH9QdQwgsuXLd5OIA" \
  -H "content-type: application/json"

## Query Chaincode for List of businesses

curl -s -X GET \
  "http://localhost:4000/channels/kiwi-channel/chaincodes/mycc?peer=peer0.org1.kiwi.com&fcn=queryBusinessesList&args=%5B%2220870%22%5D" \
  -H "authorization: Bearer <Bearer Token>" \
  -H "content-type: application/json"

## Query Chaincode for List of People

curl -s -X GET \
  "http://localhost:4000/channels/kiwi-channel/chaincodes/mycc?peer=peer0.org1.kiwi.com&fcn=queryPersonsList&args=%5B%2220870%22%5D" \
  -H "authorization: Bearer <Bearer Token>" \
  -H "content-type: application/json"


## Query Chaincode for List of Services

curl -s -X GET \
  "http://localhost:4000/channels/kiwi-channel/chaincodes/mycc?peer=peer0.org1.kiwi.com&fcn=queryServicesList&args=%5B%2220870%22%5D" \
  -H "authorization: Bearer <Bearer Token>" \
  -H "content-type: application/json"

  ## 

  {
   "selector": {
      "docType": "Person"
   },
   "fields": [
      "personName",
      "netWorth"
   ]
}