# fabric-std
# fabric-study


1. Virtualbox or VM Ware + Ubuntu 설치 (v18.04.1)
1-1. 네트워크 1개더 추가 하여  Host-only 설정
2. 환경 설치

3. fabric 따라하기.

관리자 접속
> sudo -i  
go 다운로드
> wget https://storage.googleapis.com/golang/go1.10.4.linux-amd64.tar.gz  
압축해제
> tar -xvf go1.10.4.linux-amd64.tar.gz  

PATH 설정
> mkdir /root/gopath
> gedit /etc/profile

/etc/profile  하단작성
export GOPATH=/root/gopath
export GOROOT=/root/go
export PATH=$PATH:$GOROOT/bin

저장 종료 후 시스템 업데이트.
> source /etc/profile

환경변수 설정확인
> cd $GOPATH
> cd $GOROOT

기타 개발도구 설치 
> apt-get install python-pip
> apt-get install git
> apt-get install curl
> apt-get install libltdl-dev
> apt-get install tree
> apt-get install openssh-server
> apt-get install net-tools

docker, docker-compose 설치
도커 다운로드 및 테스트
> wget https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce_17.06.2~ce-0~ubuntu_amd64.deb
> dpkg -i docker-ce_17.06.2~ce-0~ubuntu_amd64.deb
> docker run hello-world
[그림 2-1]

docker-compose 설치
> pip install docker-compose
> docker-compose --version

Atom 
> add-apt-repository ppa:webupd8team/atom
> apt-get update
> apt-get install atom


하이퍼레저 패브릭 설치
1.3 버전 다운로드
> mkdir -p $GOPATH/src/github.com/hyperledger
> cd $GOPATH/src/github.com/hyperledger
> git clone -b release-1.3 https://github.com/hyperledger/fabric


하이퍼레저 패브릭 소스 컴파일
> cd fabric
git reset --hard 명령어를 통해 필자가 사용한 하이퍼레저 패브릭 프로그램과 동일한 커밋 시점으로 이동
> git reset --hard d942308df6302d3510e835bad62f861ad854b4b3
> make
unit-test_1이 반복적으로 나타나게 됩니다. 이때 Ctrl + C를 눌러 unit-test_1을 강제 종료합니다.
[그림 2-2]

하이퍼레저 패브릭 환경변수 설정
> gedit /etc/profile
파일 하단에 다음과 같이 입력합니다.
export FABRIC_HOME=/root/gopath/src/github.com/hyperledger/fabric
export PATH=$PATH:$GOPATH/src/github.com/hyperledger/fabric/.build/bin
저장 후 종료
> source /etc/profile

정상적 설치 확인 'cryptogen' 명령어 실행
> cryptogen
[그림 2-3]

실습전 e2e_cli 테스트 실행
> cd $FABRIC_HOME/examples/e2e_cli
> ./network_setup.sh up
[그림 2-4]
ctrl-c 테스트종료
> ./network_setup.sh down


작업 디렉터리 생성
> mkdir /root/testnet
> cd testnet

시스템 기본설정 파일 경로 지정
> cp /root/gopath/src/github.com/hyperledger/fabric/sampleconfig/core.yaml /root/testnet/core.yaml
> cp /root/gopath/src/github.com/hyperledger/fabric/sampleconfig/orderer.yaml /root/testnet/orderer.yaml
> gedit /etc/profile
파일 하단에 다음과 같이 입력한다
export FABRIC_CFG_PATH=/root/testnet
저장 후 종료
> source /etc/profile

네트워크 구축
hostname 및 hosts 파일 설정
> gedit /etc/hosts
아래내용 입력
127.0.0.1	localhost

10.0.1.11	peer0
10.0.1.12	peer1
10.0.1.21	peer2
10.0.1.22	peer3
10.0.1.31	orderer0
10.0.1.32	kafka-zookeeper
10.0.1.41	client


vm 또는 버추얼박스 이미지 7개 clone 
각 복제된 이미지 네트워크 설정은 우분투 접속후 설정 > 네트워크 > 외부네트워크 ip설정 (Host-only 설정한 네트워크 X)
아이피/네트마스크  복제 이미지명
10.0.1.11/24 peer0
10.0.1.12/24 peer1
10.0.1.21/24 peer2
10.0.1.22/24 peer3
10.0.1.31/24 orderer0
10.0.1.32/24 kafka-zookeeper
10.0.1.41/24 client


MSP 생성 (client 노드에서 실행)
> cd testnet
crypto-config.yaml 파일 생성
~/testnet>  gedit crypto-config.yaml
========== crypto-config.yaml 내용 =========

OrdererOrgs:
  - Name: OrdererOrg0
    Domain: ordererorg0
    Specs:
      - Hostname: orderer0

PeerOrgs:
  - Name: Org0
    Domain: org0
    Template:
      Count: 2
    Users:
      Count: 1

  - Name: Org1
    Domain: org1
    Template:
      Count: 2
      Start: 2
    Users:
      Count: 1

==================================

Ordererorg0 이름의 조직 생성
orderer0 의 이름의 orderer0 노드 생성
Org0 이름의 조직 생성
peer0, peer1 
User1 클라이언트 생성

Org0 이름의 조직 생성
peer2, peer3 
User1 클라이언트 생성



MSP 생성 명령어(client 노드에서 실행)
~/testnet> cryptogen generate --config=./crypto-config.yaml


Genesis block 생성
~/testnet> gedit configtx.yaml



=========== configtx.yaml 내용 ==========
Organizations:
    - &OrdererOrg0
        Name: OrdererOrg0
        ID: OrdererOrg0MSP
        MSPDir: crypto-config/ordererOrganizations/ordererorg0/msp/
 
    - &Org0
        Name: Org0MSP
        ID: Org0MSP
        MSPDir: crypto-config/peerOrganizations/org0/msp/
        AnchorPeers:
            - Host: peer0
              Port: 7051
    - &Org1
        Name: Org1MSP
        ID: Org1MSP
        MSPDir: crypto-config/peerOrganizations/org1/msp/
        AnchorPeers:
            - Host: peer2
              Port: 7051

Orderer: &OrdererDefaults
    OrdererType: kafka
    Addresses:
        - orderer0:7050
    BatchTimeout: 1s
    BatchSize:
        MaxMessageCount: 30
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB
    Kafka:
        Brokers:
            - kafka-zookeeper:9092
    Organizations:

Application: &ApplicationDefaults
    Organizations:

Profiles:

    TwoOrgsOrdererGenesis:
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg0
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org0
                    - *Org1

    TwoOrgsChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org0
                - *Org1

==================================

configtx.yaml 설명
Organizations, Orderer, Profiles

Profiles   
ㄴ TwoOrgsordererGenesis - orderer 조직을 정의
ㄴ TwoOrgsChannel - 채널생성 정의

Organizations
- Profiles에서 참조 Organization의 설정과  orderer 설정 정의
- 각 조직의 Name, ID, MSP 참조경로 등이 정의.

Orderer 
- orderer가 사용하는 트랜잭션 정렬 후 블록 생성 방법(kafka), orderer 주소, 블록당 트랙잭션 수집 허용 시간, 블록의 트랜잭션 허용크기

 configtx.yaml 파일의 TwoOrgordererGenesis를 참조하여 genesis.block 생성(client 노드에서 실행)
~/testnet> configtxgen -profile TwoOrgsOrdererGenesis -outputBlock genesis.block
[그림 2-5]

genesis.block 파일 이동(client 노드에서 실행)
~/testnet> mv genesis.block /root/testnet/crypto-config/ordererOrganizations/ordererorg0/orderers/orderer0.ordererorg0/

configtx.yaml 파일의 TwoOrgsChannel을 참조하여 채널 구축을 위한 ch1.tx 트랜잭션 생성(client 노드에서 실행)
~/testnet> configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ch1.tx -channelID ch1
[그림 2-6]

configtx.yaml 파일의 TwoOrgsChannel을 참조
각 조직의 Anchor peer 설정 트랜잭션 생성
~/testnet> configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate Org0MSPanchors.tx -channelID ch1 -asOrg Org0MSP

~/testnet> configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate Org1MSPanchors.tx -channelID ch1 -asOrg Org1MSP
[그림 2-7]

peer와 orderer 노드의 홈 디렉터리로 MSP 전송(clent 노드에서 실행)
~/testnet> scp -rq /root/testnet fabric@peer0:/home/fabric/
~/testnet> scp -rq /root/testnet fabric@peer1:/home/fabric/
~/testnet> scp -rq /root/testnet fabric@peer2:/home/fabric/
~/testnet> scp -rq /root/testnet fabric@peer3:/home/fabric/
~/testnet> scp -rq /root/testnet fabric@orderer0:/home/fabric/

전송받은 작업 디렉터리를 Root 홈으로 이동(각각의  peer, orderer 노드에서 실행)
root@peer0:~> mv /home/fabric/testnet/ /root/
root@peer1:~> mv /home/fabric/testnet/ /root/
root@peer2:~> mv /home/fabric/testnet/ /root/
root@peer3:~> mv /home/fabric/testnet/ /root/
root@orderer:~> mv /home/fabric/testnet/ /root/


Peer 구동
작업 디렉터리로 이동 후 runpeer0 스크립트 생성(peer0 노드에서 실행)
> cd /root/testnet
> gedit runPeer0.sh

=========== runPeer0.sh 내용 ==========
CORE_PEER_ENDORSER_ENABLED=true \
CORE_PEER_PROFILE_ENABLED=true \
CORE_PEER_ADDRESS=10.0.1.11:7051 \
CORE_PEER_CHAINCODELISTENADDRESS=10.0.1.11:7052 \
CORE_PEER_ID=org0-peer0 \
CORE_PEER_LOCALMSPID=Org0MSP \
CORE_PEER_GOSSIP_EXTERNALENDPOINT=10.0.1.11:7051 \
CORE_PEER_GOSSIP_USELEADERELECTION=true \
CORE_PEER_GOSSIP_ORGLEADER=false \
CORE_PEER_TLS_ENABLED=false \
CORE_PEER_TLS_KEY_FILE=/root/testnet/crypto-config/peerOrganizations/org0/peers/peer0.org0/tls/server.key \
CORE_PEER_TLS_CERT_FILE=/root/testnet/crypto-config/peerOrganizations/org0/peers/peer0.org0/tls/server.crt \
CORE_PEER_TLS_ROOTCERT_FILE=/root/testnet/crypto-config/peerOrganizations/org0/peers/peer0.org0/tls/ca.crt \
CORE_PEER_TLS_SERVERHOSTOVERRIDE=10.0.1.11 \
CORE_VM_DOCKER_ATTACHSTDOUT=true \
CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/peers/peer0.org0/msp \
peer node start
==================================

CORE_PEER_ENDORSER_ENABLED : peer의 Endorsing peer 역할 여부를 결정합니다.
CORE_PEER_ADDRESS : peer의 주소값입니다.
CORE_PEER_CHAINCODELISTENADDRESS : 체인코드 관련 정보를 받기 위한 주소값입니다.
CORE_PEER_ID : peer를 식별하는 ID입니다.
CORE_PEER_LOCALMSPID : peer의 Local MSP ID 입니다.
CORE_PEER_GOSSIP_EXTERNALENDPOINT : 외부 조직과 통신을 위해 광고하는 주소값입니다.
CORE_PEER_GOSSIP_USELEADERELECTION : Gossip 프로토콜의 리더 선출 방법을 수동 혹은 자동으로 설정하는 값입니다.
CORE_PEER_GOSSIP_ORGLEADER : 프로토콜 리더를 수동으로 했을시 해당  peer를 리더로 설정할지 여부를 정하는 값입니다. (CORE_PEER_GOSSIP_USELEADERELECTION 값이  true이면 fasle로 설정)

CORE_PEER_TLS_ENABLED : TLS 통신 활성화 여부 설정합니다.
CORE_PEER_TLS_KEY_FILE :  peer 개인키 저장된 경로
CORE_PEER_TLS_CERT_FILE : peer의 디지털 인증서 파일이 저장된 경로입니다.
CORE_PEER_TLS_ROOTCERT_FILE : CA의 디지털 인증서 파일이 저장된 경로입니다.
CORE_PEER_TLS_SERVERHOSTOVERRIDE : TLS 인증서의 CN(Common Name) 입니다. 인증서가 저장된 디렉터리에서 'openssl x509 -in 인증서이름 -text -noout' 명령어를 이용해 CN을 확인할 수 있습니다.
CORE_PEER_MSPCONFIGPATH : peer의  MSP가 저장되어 있는 경로입니다.


runPeer0.sh 권한 변경 후 실행(peer0 노드에서 실행)
root@peer0:~> chmod 777 runPeer0.sh
root@peer0:~> ./runPeer0.sh

위 내용 peer1~3도 같은작업수행
=========== runPeer1.sh 내용 ==========
CORE_PEER_ENDORSER_ENABLED=true \
CORE_PEER_PROFILE_ENABLED=true \
CORE_PEER_ADDRESS=10.0.1.12:7051 \
CORE_PEER_CHAINCODELISTENADDRESS=10.0.1.12:7052 \
CORE_PEER_ID=org0-peer1 \
CORE_PEER_LOCALMSPID=Org0MSP \
CORE_PEER_GOSSIP_EXTERNALENDPOINT=10.0.1.12:7051 \
CORE_PEER_GOSSIP_USELEADERELECTION=true \
CORE_PEER_GOSSIP_ORGLEADER=false \
CORE_PEER_TLS_ENABLED=false \
CORE_PEER_TLS_KEY_FILE=/root/testnet/crypto-config/peerOrganizations/org0/peers/peer1.org0/tls/server.key \
CORE_PEER_TLS_CERT_FILE=/root/testnet/crypto-config/peerOrganizations/org0/peers/peer1.org0/tls/server.crt \
CORE_PEER_TLS_ROOTCERT_FILE=/root/testnet/crypto-config/peerOrganizations/org0/peers/peer1.org0/tls/ca.crt \
CORE_PEER_TLS_SERVERHOSTOVERRIDE=10.0.1.12 \
CORE_VM_DOCKER_ATTACHSTDOUT=true \
CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/peers/peer1.org0/msp \
peer node start
==================================
=========== runPeer2.sh 내용 ==========
CORE_PEER_ENDORSER_ENABLED=true \
CORE_PEER_PROFILE_ENABLED=true \
CORE_PEER_ADDRESS=10.0.1.21:7051 \
CORE_PEER_CHAINCODELISTENADDRESS=10.0.1.21:7052 \
CORE_PEER_ID=org1-peer2 \
CORE_PEER_LOCALMSPID=Org1MSP \
CORE_PEER_GOSSIP_EXTERNALENDPOINT=10.0.1.21:7051 \
CORE_PEER_GOSSIP_USELEADERELECTION=true \
CORE_PEER_GOSSIP_ORGLEADER=false \
CORE_PEER_TLS_ENABLED=false \
CORE_PEER_TLS_KEY_FILE=/root/testnet/crypto-config/peerOrganizations/org1/peers/peer2.org1/tls/server.key \
CORE_PEER_TLS_CERT_FILE=/root/testnet/crypto-config/peerOrganizations/org1/peers/peer2.org1/tls/server.crt \
CORE_PEER_TLS_ROOTCERT_FILE=/root/testnet/crypto-config/peerOrganizations/org1/peers/peer2.org1/tls/ca.crt \
CORE_PEER_TLS_SERVERHOSTOVERRIDE=10.0.1.21 \
CORE_VM_DOCKER_ATTACHSTDOUT=true \
CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org1/peers/peer2.org1/msp \
peer node start
==================================

=========== runPeer3.sh 내용 ==========
CORE_PEER_ENDORSER_ENABLED=true \
CORE_PEER_PROFILE_ENABLED=true \
CORE_PEER_ADDRESS=10.0.1.22:7051 \
CORE_PEER_CHAINCODELISTENADDRESS=10.0.1.22:7052 \
CORE_PEER_ID=org1-peer3 \
CORE_PEER_LOCALMSPID=Org1MSP \
CORE_PEER_GOSSIP_EXTERNALENDPOINT=10.0.1.22:7051 \
CORE_PEER_GOSSIP_USELEADERELECTION=true \
CORE_PEER_GOSSIP_ORGLEADER=false \
CORE_PEER_TLS_ENABLED=false \
CORE_PEER_TLS_KEY_FILE=/root/testnet/crypto-config/peerOrganizations/org1/peers/peer3.org1/tls/server.key \
CORE_PEER_TLS_CERT_FILE=/root/testnet/crypto-config/peerOrganizations/org1/peers/peer3.org1/tls/server.crt \
CORE_PEER_TLS_ROOTCERT_FILE=/root/testnet/crypto-config/peerOrganizations/org1/peers/peer3.org1/tls/ca.crt \
CORE_PEER_TLS_SERVERHOSTOVERRIDE=10.0.1.22 \
CORE_VM_DOCKER_ATTACHSTDOUT=true \
CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org1/peers/peer3.org1/msp \
peer node start
==================================



Kafka-Zookeeper 구동
Kafka-Zookeeper 노드 구동에 사용할 도커파일 생성(Kafka-Zookeeper 노드에서 실행)
> cd testnet
> gedit docker-compose.yaml 
=========== docker-compose.yaml 내용 ==========
version: '2'
services:
    zookeeper:
        image: hyperledger/fabric-zookeeper
>        restart: always
        ports:
            - "2181:2181"
    kafka0:
        image: hyperledger/fabric-kafka
>        restart: always
        environment:
            - KAFKA_ADVERTISED_HOST_NAME=10.0.1.32
            - KAFKA_ADVERTISED_PORT=9092
            - KAFKA_BROKER_ID=0
            - KAFKA_MESSAGE_MAX_BYTES=103809024 > 99 * 1024 * 1024 B
            - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 > 99 * 1024 * 1024 B
            - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
            - KAFKA_NUM_REPLICA_FETCHERS=1
            - KAFKA_DEFAULT_REPLICATION_FACTOR=1
            - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
        ports:
            - "9092:9092"
        depends_on:
            - zookeeper
========================================


Orderer 구동
작업 디렉터리로 이동 후 runoderer 스크립트 생성(orderer0 노드에서 실행)
> cd /root/testnet
> gedit runOrderer0.sh



채널 생성
채널을 생성하는 스크립트 제작(client 노드에서 실행)
> gedit /root/testnet/create-channel.sh
============ create-channel.sh 내용 ============
CORE_PEER_LOCALMSPID="Org0MSP" \
CORE_PEER_TLS_ROOTCERT_FILE=/root/testnet/crypto-config/peerOrganizations/org0/peers/peer0.org0/tls/ca.crt \
CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp \
CORE_PEER_ADDRESS=peer0:7051 \
peer channel create -o orderer0:7050 -c ch1 -f ch1.tx
========================================
> chmod 777 create-channel.sh
> ./create-channel.sh

Peer의 채널 참여
peer0를 채널에 참여시키는 스크립트
> gedit /root/testnet/peer0-join.sh
============ peer0-join.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org0MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp
export CORE_PEER_ADDRESS=peer0:7051
peer channel join -b ch1.block
=====================================
> gedit /root/testnet/peer1-join.sh
============ peer1-join.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org0MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp
export CORE_PEER_ADDRESS=peer1:7051
peer channel join -b ch1.block
=====================================
> gedit /root/testnet/peer2-join.sh
============ peer2-join.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org1/users/Admin@org1/msp
export CORE_PEER_ADDRESS=peer2:7051
peer channel join -b ch1.block
=====================================
> gedit /root/testnet/peer3-join.sh
============ peer3-join.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org1/users/Admin@org1/msp
export CORE_PEER_ADDRESS=peer3:7051
peer channel join -b ch1.block
=====================================
> chmod 777 peer0-join.sh
> ./peer0-join.sh
> chmod 777 peer1-join.sh
> ./peer1-join.sh
> chmod 777 peer2-join.sh
> ./peer2-join.sh
> chmod 777 peer3-join.sh
> ./peer3-join.sh

Anchor peer 업데이트
Org0의 Anchor peer를 업데이트하는 스크립트 생성(client 노드에서 실행)
> gedit /root/testnet/org0-anchor.sh
============ org0-anchor.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org0MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp
export CORE_PEER_ADDRESS=peer0:7051
peer channel create -o orderer0:7050 -c ch1 -f Org0MSPanchors.tx
======================================
> chmod 777 org0-anchor.sh
> ./org0-anchor.sh

> gedit /root/testnet/org1-anchor.sh
============ org1-anchor.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org1/users/Admin@org1/msp
export CORE_PEER_ADDRESS=peer2:7051
peer channel create -o orderer0:7050 -c ch1 -f Org1MSPanchors.tx
======================================
> chmod 777 org1-anchor.sh
> ./org1-anchor.sh

체인코드 설치
peer0 노드에 체인코드를 설치할 스크립트 생성(client 노드에서 실행)
> gedit /root/testnet/installCCpeer0.sh
============ installCCpeer0.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org0MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp
export CORE_PEER_ADDRESS=peer0:7051
peer chaincode install -n testnetCC -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/example02/cmd
======================================
> chmod 777 installCCpeer0.sh
> ./installCCpeer0.sh

> gedit /root/testnet/installCCpeer1.sh
============ installCCpeer1.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org0MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp
export CORE_PEER_ADDRESS=peer1:7051
peer chaincode install -n testnetCC -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/example02/cmd
======================================
> chmod 777 installCCpeer1.sh
> ./installCCpeer1.sh

> gedit /root/testnet/installCCpeer2.sh
============ installCCpeer2.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org1/users/Admin@org1/msp
export CORE_PEER_ADDRESS=peer2:7051
peer chaincode install -n testnetCC -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/example02/cmd
======================================
> chmod 777 installCCpeer2.sh
> ./installCCpeer2.sh

> gedit /root/testnet/installCCpeer3.sh
============ installCCpeer3.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org1/users/Admin@org1/msp
export CORE_PEER_ADDRESS=peer3:7051
peer chaincode install -n testnetCC -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/example02/cmd
======================================
> chmod 777 installCCpeer3.sh
> ./installCCpeer3.sh

peer0 노드에 설치된 체인코드를 확인하는 스크립트 생성(client 노드에서 실행)
> gedit /root/testnet/installedCClist.sh
============ installedCClist.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org0MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp
export CORE_PEER_ADDRESS=peer0:7051
peer chaincode list -C ch1 --installed
======================================
> chmod 777 installedCClist.sh
> ./installedCClist.sh

체인코드 인스턴스 생성
> gedit /root/testnet/instantiateCC.sh
============ instantiateCC.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org0MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp
export CORE_PEER_ADDRESS=peer0:7051
peer chaincode instantiate -o orderer0:7050 -C ch1 -n testnetCC -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org0MSP.member','Org1MSP.member')"
======================================
> chmod 777 instantiateCC.sh
> ./instantiateCC.sh


분산원장의 데이터 읽기
peer0을 통하여 분산원장 데이터를 읽어오는 스크립트 생성(client 노드에서 실행)
> gedit /root/testnet/query.sh
============ query.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org0MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp
export CORE_PEER_ADDRESS=peer0:7051
peer chaincode query -C ch1 -n testnetCC -c '{"Args":["query","b"]}'
======================================
> chmod 777 query.sh
> ./query.sh

분산원장에 a와 b의 데이터를 변경하는 스크립트 생성(client 노드에서 실행)
> gedit /root/testnet/invoke.sh
============ invoke.sh 내용 ============
export CORE_PEER_LOCALMSPID="Org0MSP"
export CORE_PEER_MSPCONFIGPATH=/root/testnet/crypto-config/peerOrganizations/org0/users/Admin@org0/msp
export CORE_PEER_ADDRESS=peer0:7051
peer chaincode invoke -o orderer0:7050 -C ch1 -n testnetCC -c '{"Args":["invoke","a","b","20"]}'
======================================
> chmod 777 invoke.sh
> ./invoke.sh




