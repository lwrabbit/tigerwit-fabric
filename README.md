# create first fabric network
cd tigerwit-first-fabric-network
step1:Generate Network Artifacts
./tigerwitnetwork.sh -m generate
step2:Generate GenesisBlock
./bin/cryptogen generate --config=./cry-config.yaml
export FABRIC_CFG_PATH=$PWD
./bin/configtxgen -profile TigerWitOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
step3:Generate Channel Config Info
export CHANNEL_NAME=mychannel
./bin/configtxgen -profile TigerWitChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
step4:Generate Anchor Peer Config Update File
./bin/configtxgen -profile TigerWitChannel -outputAnchorPeersUpdate ./channel-artifacts/TigerWitOrgMSPanchors.tx -channelID $CHANNEL_NAME -asOrg TigerWitOrgMSP
step5:Operate Network
CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=600 docker-compose -f docker-compose-cli.yaml up -d
step6:Create & Join Channel
     step6.1:Get Into Docker Container
     docker exec -it cli bash
     step6.2:Create Channel
     export CHANNEL_NAME=mychannel
     peer channel create -o orderer.tigerwit.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls      $CORE_PEER_TLS_ENABLED --cafile      /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/tigerwit.com/orderers/orderer.tigerwit.com     /msp/tlscacerts/tlsca.tigerwit.com-cert.pem
     step6.3:Join Channel
     peer channel join -b mychannel.block
