# cogobuyfabric
Cogobuy Hyperledger Fabric Project

# Prerequisites  
Operation System:  
Ubuntu 16.04 LTS, Core: GNU/Linux 4.13.0-36-generic x86_64  

# Environment Prepared:  
$ sudo apt update
$ sudo apt install git  
$ sudo apt install curl
$ sudo apt install docker.io  
$ sudo apt install docker-compose  
$ wget https://dl.google.com/go/go1.10.3.linux-amd64.tar.gz  
$ sudo tar -zxvf go1.10.3.linux-amd64.tar.gz -C /usr/local/  
$ sudo vim /etc/profile  
         export GOPATH=$HOME/go  
         export GOROOT=/usr/local/go  
         export PATH=$GOROOT/bin:$PATH  
$ source /etc/profile  

# Fabric Function Produce  
$ mkdir -p $GOPATH/src/github.com/hyperledger/  
$ cd $GOPATH/src/github.com/hyperledger/  
$ git clone https://github.com/hyperledger/fabric.git

$ cd fabric  
$ vim Makefile  
         BASE_VERSION = 1.2.1  
         PREV_VERSION = 1.2.0    
         CHAINTOOL_RELEASE=1.1.1  
         BASEIMAGE_RELEASE=0.4.10  
$ make orderer  
$ make peer  
$ make configtxgen  
$ make cryptogen  
$ make configtxlator  


# Fabric Docker Installation
$ sudo usermod -aG docker cogoadmin         # should be logout the current session  
$ sudo apt-get install libltdl-dev  
$ mkdir -p $GOPATH/src/golang.org/x    
$ cd $GOPATH/src/golang.org/x       
$ git clone https://github.com/golang/tools.git
$ vim ~/.bashrc     
         export PATH=$PATH:$GOPATH/bin    
$ source ~/.bashrc   

$ git clone https://github.com/golang/net.git   
$ cd $GOPATH     
$ go get github.com/kardianos/govendor   
$ go get github.com/onsi/ginkgo/ginkgo   
$ go get github.com/golang/protobuf/protoc-gen-go    
$ go get -u github.com/axw/gocov/...    
$ go get -u github.com/AlekSi/gocov-xml    
$ go get -u github.com/client9/misspell/cmd/misspell     
$ go get -u golang.org/x/tools/cmd/goimports     
$ go get -u github.com/golang/lint/golint   
$ cd $GOPATH/src/github.com/hyperledger/fabric/    
$ mkdir -p .build/docker/gotools/bin   
$ cp ~/go/bin/* .build/docker/gotools/bin    
#Manually compile docker image:    
$ cd $GOPATH/src/github.com/hyperledger/fabric/      
$ make docker   
#Next to take a look the result  
$ docker images   

# Now let's deploy Production Hyperledger Fabric    
$ cd $GOPATH/src/github.com/hyperledger       
$ mkdir cogobuyfabric && cd cogobuyfabric     
$ sudo vim crypto-config.yaml          
#Contents should be referred to file     

#after prepared crypto-config.yaml file, then run this command to generate lots of certificates   
$ sudo /home/cogoadmin/go/src/github.com/hyperledger/fabric/.build/bin/cryptogen generate --config=./crypto-config.yaml    

#if everything goes well, the result should be as follows:
Ingdan.cogobuy.com    
Foxsaas.cogobuy.com    
$ tree crypto-config   

$ sudo vim configtx.yaml             
#Contents should be referred to file     
$ mkdir channel-artifacts            
#this folder will keep genesis block and some tx files     

$ sudo /home/cogoadmin/go/src/github.com/hyperledger/fabric/.build/bin/configtxgen -profile CogobuyOrdererGenesis -outputBlock ./channel-artifacts/genesis.block     

$ sudo /home/cogoadmin/go/src/github.com/hyperledger/fabric/.build/bin/configtxgen -profile CogobuyChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID cogobuy01      

$ sudo /home/cogoadmin/go/src/github.com/hyperledger/fabric/.build/bin/configtxgen -profile CogobuyChannel -outputAnchorPeersUpdate ./channel-artifacts/IngdanMSPanchors.tx -channelID cogobuy01 -asOrg IngdanMSP      

$ sudo /home/cogoadmin/go/src/github.com/hyperledger/fabric/.build/bin/configtxgen -profile CogobuyChannel -outputAnchorPeersUpdate ./channel-artifacts/FoxsaasMSPanchors.tx -channelID cogobuy01 -asOrg FoxsaasMSP     

$ mkdir base && cd base    

$ sudo vim peer-base.yaml      
#Contents should be referred to file under base folder    
$ sudo vim docker-compose-base.yaml   
#Contents should be referred to file under base folder  
$ cd ..        
#return to parent directory /cogobuyfabric    
$ sudo vim docker-compose-cli.yaml    
$ sudo docker-compose -f docker-compose-cli.yaml up -d   
$ sudo docker-compose -f docker-compose-cli.yaml down   

# Fabric Command Operation,should enter into docker inside
$ docker exec -it cli bash     
$ peer channel create -o orderer.cogobuy.com:7050 -c cogobuy01 -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/cogobuy.com/orderers/orderer.cogobuy.com/msp/tlscacerts/tlsca.cogobuy.com-cert.pem    
$ peer channel join -b cogobuy01.block 
#the above command is just for adding peer0.Ingdan.cogobuy.com into cogobuy01 channel
$ peer channel list   


#then modify below Environment variables step by step, had better confirm last value like      
$ echo $CORE_PEER_LOCALMSPID     

CORE_PEER_LOCALMSPID=IngdanMSP    
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/Ingdan.cogobuy.com/peers/peer1.Ingdan.cogobuy.com/tls/ca.crt     
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/Ingdan.cogobuy.com/users/Admin@Ingdan.cogobuy.com/msp     
CORE_PEER_ADDRESS=peer1.Ingdan.cogobuy.com:7051    

$ peer channel list    
#looping modify with peer1.Ingdan.cogobuy.com,peer2.Ingdan.cogobuy.com,peer0.Foxsaas.cogobuy.com,peer1.Foxsaas.cogobuy.com    

#after revise back to orignial environment variables, then update anchor node setting     
$ peer channel update -o orderer.cogobuy.com:7050 -c cogobuy01 -f ./channel-artifacts/IngdanMSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/cogobuy.com/orderers/orderer.cogobuy.com/msp/tlscacerts/tlsca.cogobuy.com-cert.pem    
$ peer channel update -o orderer.cogobuy.com:7050 -c cogobuy01 -f ./channel-artifacts/FoxsaasMSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/cogobuy.com/orderers/orderer.cogobuy.com/msp/tlscacerts/tlsca.cogobuy.com-cert.pem  

#Install chaincode to node peer0   
$ peer chaincode install -n cogobuy01 -v 1.0 -p github.com/chaincode   
#then instantiate to channel
$ peer chaincode instantiate -o orderer.cogobuy.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/cogobuy.com/orderers/orderer.cogobuy.com/msp/tlscacerts/tlsca.cogobuy.com-cert.pem -C cogobuy01 -n cogobuy01 -v 1.0 -c '{"Args":["init","Custx01", "Cogobuy Custx01"]}' -P "OR ('IngdanMSP.peer','FoxsaasMSP.peer')"   

 









