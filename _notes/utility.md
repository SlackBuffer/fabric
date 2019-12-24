# `chaintool`
- A Fabric **chaincode compiler**.
    - Compile chaincode inside fabric-ccenv container.
- https://fabric-chaintool.readthedocs.io/en/latest/
- https://github.com/hyperledger/fabric-chaintool
# `configtxlator`
- `./common/tools/configtxlator`

    ```bash
    # inside CLI
    peer channel fetch config config_block.pb -o orderer.example.com:7050 -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

    apt update && apt install -y jq
    # https://github.com/stedolan/jq

    configtxlator start &

    curl -X POST --data-binary @config_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > config_block.json

    # jq . config_block.json
    jq .data.
    ```

- https://hyperledger-fabric.readthedocs.io/en/latest/commands/configtxlator.html
# `cryptogen`
- Generates the x509 certificates used to identify and authenticate the various components in the network.
- `./common/tools/cryptogen`
- `cryptogen` consumes a file - ``crypto-config.yaml`` - that contains the network topology and allows us to generate a library of certificates for both the organizations and the components that belong to those organizations. 
    - Each organization is provisioned a unique root certificate (``ca-cert``), that binds specific components (peers and orderers) to that org.
    - Transactions and communications within Fabric are signed by an entity's private key (``keystore``), and then verified by means of a public key (``signcerts``). 
    - You will notice a `count` variable within this file. We use this to specify the number of peers per Organization.
        - In our case it's two peers per Org. The rest of this template is extremely self-explanatory.

- `$CRYPTOGEN generate --config=./crypto-config.yaml`
# `configtxgen`
- Generates the requisite configuration artifacts for orderer bootstrap and channel creation.
- `./fabric/common/configtx/tool/configtxgen`
- `$CONFIGTXGEN -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block`
- `$CONFIGTXGEN -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME`

- `../../release/darwin-amd64/bin/configtxgen --inspectBlock ./channel-artifacts/genesis.block -profile TwoOrgsOrdererGenesis -channelID mychannel > inspectBlock.json`
- `../../release/darwin-amd64/bin/configtxgen --inspectChannelCreateTx ./channel-artifacts/channel.tx -profile TwoOrgsChannel > inspectChannelCreateTx.json`
# fabric-baseimage
# fabric-baseos
# fabric-ccenv
- The `fabric-ccenv` image is used to **build chaincode**.
    - The `fabric-ccenv` image currently includes the github.com/hyperledger/fabric/core/chaincode/shim (“shim”) package. This is convenient, as it provides the ability to package chaincode without the need to include the “shim”. However, this may cause issues in future releases (and/or when trying to use packages which are included by the “shim”).
    - In order to avoid any issues, users are advised to manually vendor the “shim” package with their chaincode prior to using the peer CLI for packaging and/or for installing chaincode.
    - Please refer to https://jira.hyperledger.org/browse/FAB-5177 for more details, and kindly be aware that given the above, we may end up changing the `fabric-ccenv` in the future.
    - > https://hyperledger-fabric.readthedocs.io/en/release-1.1/releases.html#known-issues-workarounds