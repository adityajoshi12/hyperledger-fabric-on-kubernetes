# Hyperledger Fabric On Kubernetes

### Youtube

<a href="https://youtu.be/e04TcJHUI5M" title="Hyperledger Fabric On Kubernetes">
  <p align="center">
    <img width="75%" src="https://img.youtube.com/vi/e04TcJHUI5M/maxresdefault.jpg" alt="Music Caster Video Demo Thumbnail"/>
  </p>
</a>

### Namespace

```bash
kubectl create ns fabric
```

### CA

```bash
kubectl hlf ca create --storage-class=do-block-storage --capacity=2Gi --name=org1-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric
kubectl hlf ca create --storage-class=do-block-storage --capacity=2Gi --name=org2-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric
kubectl hlf ca create --storage-class=do-block-storage --capacity=2Gi --name=ord-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric
```

```bash
# GET ALL CAs
kubectl get fabriccas.hlf.kungfusoftware.es -A
```

### Peer

```bash
kubectl hlf ca register --name=org1-ca --user=org1-peer1 --secret=peerpw --type=peer --enroll-id enroll --enroll-secret=enrollpw --mspid=Org1MSP --namespace=fabric
kubectl hlf ca register --name=org1-ca --user=org1-peer2 --secret=peerpw --type=peer --enroll-id enroll --enroll-secret=enrollpw --mspid=Org1MSP --namespace=fabric
kubectl hlf ca register --name=org2-ca --user=org2-peer1 --secret=peerpw --type=peer --enroll-id enroll --enroll-secret=enrollpw --mspid=Org2MSP --namespace=fabric
kubectl hlf ca register --name=org2-ca --user=org2-peer2 --secret=peerpw --type=peer --enroll-id enroll --enroll-secret=enrollpw --mspid=Org2MSP --namespace=fabric
```

```bash
kubectl hlf peer create --storage-class=do-block-storage --enroll-id=org1-peer1 --mspid=Org1MSP --enroll-pw=peerpw --capacity=5Gi --name=org1-peer1 --ca-name=org1-ca.fabric --namespace=fabric --statedb=couchdb
kubectl-hlf peer create --storage-class=do-block-storage --enroll-id=org1-peer2 --mspid=Org1MSP --enroll-pw=peerpw --capacity=5Gi --name=org1-peer2 --ca-name=org1-ca.fabric --namespace=fabric --statedb=couchdb
kubectl-hlf peer create --storage-class=do-block-storage --enroll-id=org2-peer1 --mspid=Org2MSP --enroll-pw=peerpw --capacity=5Gi --name=org2-peer1 --ca-name=org2-ca.fabric --namespace=fabric --statedb=couchdb
kubectl-hlf peer create --storage-class=do-block-storage --enroll-id=org2-peer2 --mspid=Org2MSP --enroll-pw=peerpw --capacity=5Gi --name=org2-peer2 --ca-name=org2-ca.fabric --namespace=fabric --statedb=couchdb
```

```bash
# GET ALL PEERs
kubectl get fabricpeers.hlf.kungfusoftware.es -A
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
kubectl hlf utils adduser --userPath=org1-peer.yaml --config=networkConfig.yaml --username=admin --mspid=Org1MSP
kubectl hlf utils adduser --userPath=org2-peer.yaml --config=networkConfig.yaml --username=admin --mspid=Org2MSP
```

### Orderer

```bash
kubectl hlf ca register --name=ord-ca --user=orderer --secret=ordererpw  --type=orderer --enroll-id enroll --enroll-secret=enrollpw --mspid=OrdererMSP --namespace=fabric
```

```bash
kubectl hlf ordnode create  --storage-class=do-block-storage --enroll-id=orderer --mspid=OrdererMSP --enroll-pw=ordererpw --capacity=2Gi --name=ord-node1 --ca-name=ord-ca.fabric --namespace=fabric
```

```bash
kubectl hlf ca register --name=ord-ca --user=admin --secret=adminpw --type=admin --enroll-id enroll --enroll-secret=enrollpw --mspid=OrdererMSP --namespace=fabric
kubectl-hlf ca enroll --name=ord-ca --user=admin --secret=adminpw --mspid=OrdererMSP --ca-name ca  --output admin-ordservice.yaml --namespace=fabric
kubectl-hlf ca enroll --name=ord-ca --user=admin --secret=adminpw --mspid=OrdererMSP --ca-name tlsca  --output admin-tls-ordservice.yaml --namespace=fabric
```

```bash
kubectl-hlf inspect --output ordservice.yaml -o OrdererMSP
```

```bash
kubectl-hlf utils adduser --userPath=admin-ordservice.yaml --config=ordservice.yaml --username=admin --mspid=OrdererMSP
```

### Connection Profile

```bash
kubectl hlf inspect --output networkConfig.yaml -o Org1MSP -o OrdererMSP -o Org2MSP
```

### Channel

```bash
kubectl hlf channel generate --output=mychannel.block --name=mychannel --organizations Org1MSP --organizations Org2MSP --ordererOrganizations OrdererMSP
```

```bash
kubectl hlf ordnode join --block=mychannel.block --name=ord-node1 --namespace=fabric --identity=admin-tls-ordservice.yaml --namespace=fabric
```

```bash
kubectl hlf channel join --name=mychannel --config=networkConfig.yaml --user=admin -p=org1-peer1.fabric
kubectl hlf channel join --name=mychannel --config=networkConfig.yaml --user=admin -p=org1-peer2.fabric
kubectl hlf channel join --name=mychannel --config=networkConfig.yaml --user=admin -p=org2-peer1.fabric
kubectl hlf channel join --name=mychannel --config=networkConfig.yaml --user=admin -p=org2-peer2.fabric
```

### Anchor Peers

```bash
kubectl hlf channel addanchorpeer --channel=mychannel --config=networkConfig.yaml --user=admin --peer=org1-peer1.fabric
kubectl hlf channel addanchorpeer --channel=mychannel --config=networkConfig.yaml --user=admin --peer=org2-peer1.fabric
```

### Chaincode

```bash
CC_NAME=mycc
cat <<METADATA-EOF >"metadata.json"
    {
        "type": "ccaas",
        "label": "${CC_NAME}"
     }
METADATA-EOF

cat <<CONN_EOF >"connection.json"
    {
    "address": "${CC_NAME}:7052",
    "dial_timeout": "10s",
    "tls_required": false
    }
CONN_EOF

tar cfz code.tar.gz connection.json
tar cfz ${CC_NAME}-external.tgz metadata.json code.tar.gz
PACKAGE_ID=$(kubectl-hlf chaincode calculatepackageid --path=$CC_NAME-external.tgz --language=node --label=$CC_NAME)
echo "PACKAGE_ID=$PACKAGE_ID"
```

### Installing Chaincode

```bash
kubectl hlf chaincode install --path=./${CC_NAME}-external.tgz --config=networkConfig.yaml --language=node --label=$CC_NAME --user=admin --peer=org1-peer1.fabric
kubectl hlf chaincode install --path=./${CC_NAME}-external.tgz --config=networkConfig.yaml --language=node --label=$CC_NAME --user=admin --peer=org2-peer1.fabric
```

### Chaincode Containerizing

```bash
chaincode structure
package.json file
dockerfile
docker build and push
```

### Deploying Chaincode

```bash
kubectl hlf externalchaincode sync --image=adityajoshi12/hlf-nodejs-external-cc:latest --name=$CC_NAME --namespace=fabric --package-id=$PACKAGE_ID --tls-required=false --replicas=1
```

### Approve Chaincode

```bash
kubectl hlf chaincode approveformyorg --config=networkConfig.yaml --user=admin --peer=org1-peer1.fabric --package-id=$PACKAGE_ID --version 1.0 --sequence 1 --name=$CC_NAME --policy="OR('Org1MSP.member','Org2MSP.member')" --channel=mychannel
kubectl hlf chaincode approveformyorg --config=networkConfig.yaml --user=admin --peer=org2-peer1.fabric --package-id=$PACKAGE_ID --version 1.0 --sequence 1 --name=$CC_NAME --policy="OR('Org1MSP.member','Org2MSP.member')" --channel=mychannel
```

### Commit Chaincode

```bash
kubectl hlf chaincode commit --config=networkConfig.yaml --mspid=Org1MSP --user=admin --version 1.0 --sequence 1 --name=$CC_NAME --policy="OR('Org1MSP.member','Org2MSP.member')" --channel=mychannel
```

### Invoke/Query

```bash
kubectl hlf chaincode invoke --config=networkConfig.yaml --user=admin --peer=org1-peer1.fabric --chaincode=$CC_NAME --channel=mychannel --fcn=createCar -a "1000" -a "honda" -a "civic" -a "red" -a "aditya"
```

```bash
kubectl hlf chaincode query --config=networkConfig.yaml --user=admin --peer=org1-peer1.fabric --chaincode=$CC_NAME --channel=mychannel --fcn=queryAllCars -a ''
```

### Also read
1. [Istio Enabled blockchain network](ISTIO.md)
2. [Adding new org running in different kubernetes cluster](ADD-ORG.md)
