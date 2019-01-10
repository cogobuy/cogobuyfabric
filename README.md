# cogobuyfabric
Cogobuy Hyperledger Fabric Project

Prerequisites  
Operation System:  
Ubuntu 16.04 LTS, Core: GNU/Linux 4.13.0-36-generic x86_64  

Installation:  
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

Install Fabric  
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


Install Docker  
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


Manually compile docker image:    
$ cd $GOPATH/src/github.com/hyperledger/fabric/      
$ make docker   
Next to take a look the result  
$ docker images   

Now let's deploy Production Hyperledger Fabric    
$ cd $GOPATH/src/github.com/hyperledger       
$ mkdir cogobuyfabric && cd cogobuyfabric     
$ sudo vim crypto-config.yaml          #Contents should be referred to file    


