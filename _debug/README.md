
## Prerequisite

```bash
# github.com/hyperledger/fabric
make configtxlator && make cryptogen && make configtxgen

# github.com/hyperledger/fabric/_debug/first-network
./byfn down
./byfn up
```

## 1 native peer, 3 dockerized peers, 3 native raft orderers

### 3 native raft orderers

```bash
# https://yq.aliyun.com/articles/739846

# /etc/hosts 加 orderer0.example.com 等到 127.0.0.1 的映射
# github.com/hyperledger/fabric/_debug/first-network
./byfn.sh generate -o etcdraft 

# github.com/hyperledger/fabric/_debug/first-network
docker-compose -f docker-compose-cli-raft-native-3-orderers.yaml up -d
docker exec cli scripts/script.sh mychannel 3 golang 10 false


docker-compose -f docker-compose-cli-raft-native-3-orderers.yaml down -v
./byfn.sh down
sudo rm -rf ../orderer/orderer-data/* 


# https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html#create-join-channel

# 创建通道
# github.com/hyperledger/fabric/_debug/peer/bin
export FABRIC_CFG_PATH=/Users/slackbuffer/go/src/github.com/hyperledger/fabric/_debug/sampleconfig
export CORE_PEER_MSPCONFIGPATH=/Users/slackbuffer/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/Users/slackbuffer/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CHANNEL_NAME=mychannel
./peer0 channel create -o orderer0.example.com:7050 -c $CHANNEL_NAME -f /Users/slackbuffer/go/src/github.com/hyperledger/fabric/_debug/first-network/channel-artifacts/channel.tx --tls --cafile /Users/slackbuffer/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```



### 1 native peer, 3 dockerized peers

```bash
# /etc/hosts 加 peer0.org1.example.com 等到 127.0.0.1 的映射

# github.com/hyperledger/fabric/_debug/first-network
docker-compose -f docker-compose-cli-raft-native-3-orderers-1-peer.yaml up -d
# ./scripts/script-1-native-peer.sh mychannel 3 golang 10 false

docker-compose -f docker-compose-cli-raft-native-3-orderers-1-peer.yaml down -v
sudo rm -rf ../peer/peer-data/*
```

- [x] 先关闭 discovery 服务
    - Don't put any bootstrap peers or anchor peers, and then peers don't know each other and thus don't gossip.
    - > https://lists.hyperledger.org/g/fabric/topic/30184836

## 1 native peer, 1 native solo orderer

```bash
cd $GOPATH/src/hyperledger/fabric/_debug/first-network-simple
./byfn.sh generate
```

### 1 native solo orderer

### 1 native peer

### op

```bash
export FABRIC_CFG_PATH=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/sampleconfig
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_CERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

# 创建通道
./peer channel create -o orderer0.example.com:11050 -c mychannel -f ../_debug/first-network-simple/channel-artifacts/channel.tx --tls true --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
# 加入通道
./peer channel join -b ./mychannel.block
# 安装合约
./peer chaincode install -n mycc -v 1.0 -l golang -p github.com/hyperledger/fabric/_debug/chaincode/chaincode_example02/go
# 创建 net_byfn 网络
# docker network create net_byfn
# 初始化合约
./peer chaincode instantiate -o orderer0.example.com:11050 --tls true --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -l golang -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P 'OR ('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'

./peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'

./peer chaincode invoke -o orderer0.example.com:11050 --tls true --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'

./peer chaincode query -C mychannel -n mycc -c '{"Args":["query","b"]}'

./peer chaincode upgrade -o orderer0.example.com:11050 --tls true --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -l golang -v 3.2 -c '{"Args":["init","a","100","b","200"]}' -P 'OR ('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'


# 清理环境
./byfn.sh down
sudo rm -rf ../orderer/orderer-data/*
sudo rm -rf ../peer/peer-data/*
```

### pkcs11

```bash
# build native peer
go build -tags pkcs11
# "buildFlags": "-tags=pkcs11"


softhsm2-util --init-token --slot 0 --label "fabric" --so-pin 123456 --pin 654321
# The token has been initialized.
softhsm2-util --show-slots
# Available slots:
# Slot 336165381
#     Slot info:
#         Description:      SoftHSM slot ID 0x14097a05                                      
#         Manufacturer ID:  SoftHSM project                 
#         Hardware version: 2.2
#         Firmware version: 2.2
#         Token present:    yes
#     Token info:
#         Manufacturer ID:  SoftHSM project                 
#         Model:            SoftHSM v2      
#         Hardware version: 2.2
#         Firmware version: 2.2
#         Serial number:    ea5b2ba514097a05
#         Initialized:      yes
#         User PIN init.:   yes
#         Label:            fabric

# /usr/local/lib/softhsm/libsofthsm2.so

## 导入 peer 节点的私钥
## /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/keystore
## !!! 注意导入的时候 id 要是私钥的 ski，它才能够通过证书拿到公钥，得到公钥的 ski 后找到对应私钥（公钥和私钥的 ski 值一样）
softhsm2-util --import 5b874ddb3b8d35805ef06a2558623ab60a29e8e8cbe18f5bd8ea86dd84453fd2_sk --id 5b874ddb3b8d35805ef06a2558623ab60a29e8e8cbe18f5bd8ea86dd84453fd2 --label fabric --pin 654321 --slot 336165381
## id 长度任意，hex 格式，这样是错的！！！

## /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore
## 导入 org1 admin 用户的私钥
softhsm2-util --import fec5370c6a3e5242e442115947895c1165b7936f97c61cf24d7e8bddf568daa8_sk --id fec5370c6a3e5242e442115947895c1165b7936f97c61cf24d7e8bddf568daa8 --label fabric --pin 654321 --slot 336165381




export FABRIC_CFG_PATH=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/sampleconfig
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_CERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_BCCSP_DEFAULT=PKCS11
export CORE_PEER_BCCSP_PKCS11_LIBRARY=/usr/local/lib/softhsm/libsofthsm2.so
export CORE_PEER_BCCSP_PKCS11_PIN=654321
export CORE_PEER_BCCSP_PKCS11_LABEL=fabric
export CORE_PEER_BCCSP_PKCS11_HASH=SHA2
export CORE_PEER_BCCSP_PKCS11_SECURITY=256
./peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
./peer chaincode invoke -o orderer0.example.com:11050 --tls true --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
./peer chaincode query -C mychannel -n mycc -c '{"Args":["query","b"]}'
```

### chaincode dev mode

> https://hyperledger-fabric.readthedocs.io/en/release-1.4/peer-chaincode-devmode.html

```bash
export FABRIC_CFG_PATH=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/sampleconfig
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ENABLED=false
export CORE_PEER_TLS_CERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

./peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
./peer chaincode invoke -o orderer0.example.com:11050 --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network-simple/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
```

## Misc

https://chai2010.cn/advanced-go-programming-book/ch4-rpc/ch4-08-grpcurl.html

https://yq.aliyun.com/articles/740152?spm=a2c4e.11155435.0.0.42576ac1yT0Ffp  
https://www.jianshu.com/p/dbed4e210956?spm=a2c4e.10696291.0.0.6fd019a43RbjSU  

https://stackoverflow.com/questions/62495170/hyperledger-fabric-serverhandshake-tls-handshake-bad-certificate-server-peerser

<!-- Error: Error endorsing chaincode: rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: <nil>

2021-02-01 10:49:45.294 UTC [core.comm] ServerHandshake -> ERRO 06c Server TLS handshake failed in 174.908µs with error tls: first record does not look like a TLS handshake server=Orderer remoteaddress=127.0.0.1:59702 -->

```bash
# export CORE_PEER_TLS_ENABLED=true   # !!!! 这个参数踩过坑

export FABRIC_CFG_PATH=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/sampleconfig
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_CERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

./peer channel create -o orderer0.example.com:7050 -c mychannel -f ../_debug/first-network/channel-artifacts/channel.tx --tls true --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

./peer channel join -b ./mychannel.block

./peer chaincode install -n mycc -v 1.0 -l golang -p github.com/hyperledger/fabric/_debug/chaincode/chaincode_example02/go

./peer chaincode instantiate -o orderer0.example.com:7050 --tls true --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -l golang -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P 'OR ('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'
# Error: could not assemble transaction, err proposal response was not successful, error code 500, msg error starting container: error starting container: Failed to generate platform-specific docker build: Error executing build: API error (404): network net_byfn not found ""
# docker-compose -f docker-compose-cli-raft-native-3-orderers-1-peer.yaml up -d

# Error: could not assemble transaction, err proposal response was not successful, error code 500, msg chaincode registration failed: container exited with 0
# debug 中的 peer 的 CORE_PEER_CHAINCODEADDRESS 改成宿主机 ip
# https://www.jianshu.com/p/dbed4e210956?spm=a2c4e.10696291.0.0.6fd019a43RbjSU

./peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'

./peer chaincode invoke -o orderer0.example.com:7050 --tls true --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'

./peer chaincode query -C mychannel -n mycc -c '{"Args":["query","b"]}'
```

---

```bash
./peer chaincode invoke -o orderer0.example.com:7050 --tls true --cafile /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/_builddep/bad/tlsca.example.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /home/ubuntu/go/src/github.com/hyperledger/fabric/_debug/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
```
