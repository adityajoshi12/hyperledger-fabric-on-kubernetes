# HLF Operator With Istio

## Adding Istio and HLF Opertor

```bash
helm repo add kfs https://kfsoftware.github.io/hlf-helm-charts --force-update
helm install hlf-operator --version=1.8.0-beta9 --set image.tag=v1.8.0 kfs/hlf-operator
OR
helm install hlf-operator ./chart/hlf-operator

istioctl install --set profile=default -y

export PEER_IMAGE=hyperledger/fabric-peer
export PEER_VERSION=2.4.6

export ORDERER_IMAGE=hyperledger/fabric-orderer
export ORDERER_VERSION=2.4.6

export CA_IMAGE=hyperledger/fabric-ca
export CA_VERSION=1.5
export SC=$(kubectl get sc -o=jsonpath='{.items[0].metadata.name}')
export DOMAIN=
```

### CA

```bash
kubectl hlf ca create --storage-class=$SC --capacity=2Gi --name=org1-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric --hosts=org1-ca.$DOMAIN --istio-ingressgateway=ingressgateway --istio-port=443 --image=$CA_IMAGE --version=$CA_VERSION
kubectl hlf ca create --storage-class=$SC --capacity=2Gi --name=org2-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric --hosts=org2-ca.$DOMAIN --istio-ingressgateway=ingressgateway --istio-port=443 --image=$CA_IMAGE --version=$CA_VERSION
kubectl hlf ca create --storage-class=$SC --capacity=2Gi --name=ord-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric --hosts=ord-ca.$DOMAIN --istio-ingressgateway=ingressgateway --istio-port=443 --image=$CA_IMAGE --version=$CA_VERSION
```

### Register Peer Identities

```bash
kubectl hlf ca register --name=org1-ca --user=org1-peer1 --secret=peerpw --type=peer --enroll-id enroll --enroll-secret=enrollpw --mspid=Org1MSP --namespace=fabric
kubectl hlf ca register --name=org1-ca --user=org1-peer2 --secret=peerpw --type=peer --enroll-id enroll --enroll-secret=enrollpw --mspid=Org1MSP --namespace=fabric
kubectl hlf ca register --name=org2-ca --user=org2-peer1 --secret=peerpw --type=peer --enroll-id enroll --enroll-secret=enrollpw --mspid=Org2MSP --namespace=fabric
kubectl hlf ca register --name=org2-ca --user=org2-peer2 --secret=peerpw --type=peer --enroll-id enroll --enroll-secret=enrollpw --mspid=Org2MSP --namespace=fabric
```

### Peer

```bash
kubectl hlf peer create --storage-class=$SC --enroll-id=org1-peer1 --mspid=Org1MSP --enroll-pw=peerpw --capacity=5Gi --name=org1-peer1 --ca-name=org1-ca.fabric --namespace=fabric --statedb=couchdb --hosts=org1-peer1."$DOMAIN" --istio-ingressgateway=ingressgateway --istio-port=443 --image=$PEER_IMAGE --version=$PEER_VERSION 
kubectl-hlf peer create --storage-class=$SC --enroll-id=org1-peer2 --mspid=Org1MSP --enroll-pw=peerpw --capacity=5Gi --name=org1-peer2 --ca-name=org1-ca.fabric --namespace=fabric --statedb=couchdb --hosts=org1-peer2."$DOMAIN" --istio-ingressgateway=ingressgateway --istio-port=443 --image=$PEER_IMAGE --version=$PEER_VERSION 
kubectl-hlf peer create --storage-class=$SC --enroll-id=org2-peer1 --mspid=Org2MSP --enroll-pw=peerpw --capacity=5Gi --name=org2-peer1 --ca-name=org2-ca.fabric --namespace=fabric --statedb=couchdb --hosts=org2-peer1."$DOMAIN" --istio-ingressgateway=ingressgateway --istio-port=443 --image=$PEER_IMAGE --version=$PEER_VERSION 
kubectl-hlf peer create --storage-class=$SC --enroll-id=org2-peer2 --mspid=Org2MSP --enroll-pw=peerpw --capacity=5Gi --name=org2-peer2 --ca-name=org2-ca.fabric --namespace=fabric --statedb=couchdb --hosts=org2-peer2."$DOMAIN" --istio-ingressgateway=ingressgateway --istio-port=443 --image=$PEER_IMAGE --version=$PEER_VERSION 
```

### Admin Certs

```bash
kubectl hlf ca register --name=org1-ca --user=admin --secret=adminpw --type=admin --enroll-id enroll --enroll-secret=enrollpw --mspid=Org1MSP --namespace=fabric
kubectl hlf ca enroll --name=org1-ca --user=admin --secret=adminpw --ca-name ca  --output org1-peer.yaml --mspid=Org1MSP --namespace=fabric
```

```bash
kubectl hlf ca register --name=org2-ca --user=admin --secret=adminpw --type=admin --enroll-id enroll --enroll-secret=enrollpw --mspid=Org2MSP --namespace=fabric
kubectl hlf ca enroll --name=org2-ca --user=admin --secret=adminpw --ca-name ca  --output org2-peer.yaml --mspid=Org2MSP --namespace=fabric
```

### Connection Profile

```bash
kubectl hlf inspect --output networkConfig.yaml -o Org1MSP -o OrdererMSP -o Org2MSP
```

```bash
# adding users to connection profile
kubectl hlf utils adduser --userPath=org1-peer.yaml --config=networkConfig.yaml --username=admin --mspid=Org1MSP
kubectl hlf utils adduser --userPath=org2-peer.yaml --config=networkConfig.yaml --username=admin --mspid=Org2MSP
```

### Orderer

```bash
kubectl hlf ca register --name=ord-ca --user=orderer --secret=ordererpw  --type=orderer --enroll-id enroll --enroll-secret=enrollpw --mspid=OrdererMSP --namespace=fabric
```

```bash
kubectl hlf ordnode create  --storage-class=do-block-storage --enroll-id=orderer --mspid=OrdererMSP --enroll-pw=ordererpw --capacity=2Gi --name=ord-node1 --ca-name=ord-ca.fabric --namespace=fabric --hosts=ord-node1.$DOMAIN --istio-ingressgateway=ingressgateway --istio-port=443 --image=$ORDERER_IMAGE --version=$ORDERER_VERSION

kubectl hlf ordnode create  --storage-class=do-block-storage --enroll-id=orderer --mspid=OrdererMSP --enroll-pw=ordererpw --capacity=2Gi --name=ord-node2 --ca-name=ord-ca.fabric --namespace=fabric --hosts=ord-node2.$DOMAIN --istio-ingressgateway=ingressgateway --istio-port=443 --image=$ORDERER_IMAGE --version=$ORDERER_VERSION

kubectl hlf ordnode create  --storage-class=do-block-storage --enroll-id=orderer --mspid=OrdererMSP --enroll-pw=ordererpw --capacity=2Gi --name=ord-node3 --ca-name=ord-ca.fabric --namespace=fabric --hosts=ord-node3.$DOMAIN --istio-ingressgateway=ingressgateway --istio-port=443 --image=$ORDERER_IMAGE --version=$ORDERER_VERSION
```

```bash
kubectl hlf ca register --name=ord-ca --user=admin --secret=adminpw --type=admin --enroll-id enroll --enroll-secret=enrollpw --mspid=OrdererMSP --namespace=fabric
kubectl-hlf ca enroll --name=ord-ca --user=admin --secret=adminpw --mspid=OrdererMSP --ca-name ca  --output admin-ordservice.yaml --namespace=fabric
kubectl-hlf ca enroll --name=ord-ca --user=admin --secret=adminpw --mspid=OrdererMSP --ca-name tlsca  --output admin-tls-ordservice.yaml --namespace=fabric
```

```bash

kubectl-hlf inspect --output ordservice.yaml -o OrdererMSP
kubectl-hlf utils adduser --userPath=admin-ordservice.yaml --config=ordservice.yaml --username=admin --mspid=OrdererMSP
```

### Connection Profile

```bash
kubectl hlf inspect --output networkConfig.yaml -o Org1MSP -o OrdererMSP -o Org2MSP
# adding users to connection profile
kubectl hlf utils adduser --userPath=org1-peer.yaml --config=networkConfig.yaml --username=admin --mspid=Org1MSP
kubectl hlf utils adduser --userPath=org2-peer.yaml --config=networkConfig.yaml --username=admin --mspid=Org2MSP
```

### Channel

```bash
# create channel - mychannel
kubectl hlf channel generate --output=mychannel.block --name=mychannel --organizations Org1MSP --organizations Org2MSP --ordererOrganizations OrdererMSP
```

### Fix the Orderer Admin service
Add the admin hosts inside the `spec.adminIstio`
Eg:
```
  adminIstio:
    hosts:
      - ord-node1-admin.$DOMAIN
    ingressGateway: ingressgateway
    port: 443
```

```bash
# Orderer join channel
kubectl hlf ordnode join --block=mychannel.block --name=ord-node1 --namespace=fabric --identity=admin-tls-ordservice.yaml --namespace=fabric
kubectl hlf ordnode join --block=mychannel.block --name=ord-node2 --namespace=fabric --identity=admin-tls-ordservice.yaml --namespace=fabric
kubectl hlf ordnode join --block=mychannel.block --name=ord-node3 --namespace=fabric --identity=admin-tls-ordservice.yaml --namespace=fabric

```

```bash
# peer channel join
kubectl hlf channel join --name=mychannel --config=networkConfig.yaml --user=admin -p=org1-peer1.fabric
kubectl hlf channel join --name=mychannel --config=networkConfig.yaml --user=admin -p=org1-peer2.fabric
kubectl hlf channel join --name=mychannel --config=networkConfig.yaml --user=admin -p=org2-peer1.fabric
kubectl hlf channel join --name=mychannel --config=networkConfig.yaml --user=admin -p=org2-peer2.fabric
```

### Anchor Peer

```bash
kubectl hlf channel addanchorpeer --channel=mychannel --config=networkConfig.yaml --user=admin --peer=org1-peer1.fabric
kubectl hlf channel addanchorpeer --channel=mychannel --config=networkConfig.yaml --user=admin --peer=org2-peer1.fabric
```

### Installing Chaincode

#### Create metadata file

```bash
# remove the code.tar.gz chaincode.tgz if they exist
rm code.tar.gz chaincode.tgz
export CHAINCODE_NAME=asset
export CHAINCODE_LABEL=asset
cat << METADATA-EOF > "metadata.json"
{
    "type": "ccaas",
    "label": "${CHAINCODE_LABEL}"
}
METADATA-EOF
```

#### Prepare connection file

```bash
cat > "connection.json" <<CONN_EOF
{
  "address": "${CHAINCODE_NAME}:7052",
  "dial_timeout": "10s",
  "tls_required": false
}
CONN_EOF

tar cfz code.tar.gz connection.json
tar cfz chaincode.tgz metadata.json code.tar.gz
export PACKAGE_ID=$(kubectl hlf chaincode calculatepackageid --path=chaincode.tgz --language=node --label=$CHAINCODE_LABEL)
echo "PACKAGE_ID=$PACKAGE_ID"

kubectl hlf chaincode install --path=./chaincode.tgz --config=networkConfig.yaml --language=golang --label=$CHAINCODE_LABEL --user=admin --peer=org1-peer1.fabric
kubectl hlf chaincode install --path=./chaincode.tgz --config=networkConfig.yaml --language=golang --label=$CHAINCODE_LABEL --user=admin --peer=org1-peer2.fabric


kubectl hlf chaincode install --path=./chaincode.tgz --config=networkConfig.yaml --language=golang --label=$CHAINCODE_LABEL --user=admin --peer=org2-peer1.fabric
kubectl hlf chaincode install --path=./chaincode.tgz --config=networkConfig.yaml --language=golang --label=$CHAINCODE_LABEL --user=admin --peer=org2-peer2.fabric

```

#### Deploy chaincode container on cluster
The following command will create or update the CRD based on the packageID, chaincode name, and docker image.

```bash
kubectl hlf externalchaincode sync --image=kfsoftware/chaincode-external:latest \
    --name=$CHAINCODE_NAME \
    --namespace=fabric \
    --package-id=$PACKAGE_ID \
    --tls-required=false \
    --replicas=1
```

#### Check installed chaincodes
```bash
kubectl hlf chaincode queryinstalled --config=networkConfig.yaml --user=admin --peer=org1-peer1.fabric
```

### Approve Chaincode

```bash
export SEQUENCE=1
export VERSION="1.0"
kubectl hlf chaincode approveformyorg --config=networkConfig.yaml --user=admin --peer=org1-peer1.fabric \
    --package-id=$PACKAGE_ID \
    --version "$VERSION" --sequence "$SEQUENCE" --name=asset \
    --policy="OR('Org1MSP.member','Org2.member')" --channel=mychannel

kubectl hlf chaincode approveformyorg --config=networkConfig.yaml --user=admin --peer=org2-peer1.fabric \
    --package-id=$PACKAGE_ID \
    --version "$VERSION" --sequence "$SEQUENCE" --name=asset \
    --policy="OR('Org1MSP.member','Org2.member')" --channel=mychannel
```

### Commit Chaincode

```bash
kubectl hlf chaincode commit --config=networkConfig.yaml --user=admin --mspid=Org1MSP \
    --version "$VERSION" --sequence "$SEQUENCE" --name=asset \
    --policy="OR('Org1MSP.member','Org2.member')" --channel=mychannel
```

### Invoke/Query Chaincode

```bash
kubectl hlf chaincode invoke --config=networkConfig.yaml \
    --user=admin --peer=org1-peer1.fabric \
    --chaincode=asset --channel=mychannel \
    --fcn=initLedger -a '[]'
    
kubectl hlf chaincode invoke --config=networkConfig.yaml \
    --user=admin --peer=org1-peer1.fabric \
    --chaincode=asset --channel=mychannel \
    --fcn=CreateAsset -a "100" -a "honda" -a "5" -a "red" -a "2000"

kubectl hlf chaincode query --config=networkConfig.yaml --user=admin --peer=org1-peer1.fabric --chaincode=asset --channel=mychannel --fcn=GetAllAssets
```
