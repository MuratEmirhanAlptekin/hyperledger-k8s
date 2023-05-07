# Hyperledger Fabric On Kubernetes


You first need to clone hlf-operator locally from https://github.com/hyperledger/bevel-operator-fabric.git 
and then go helm install hlf-operator ./chart/hlf-operator inside the repo
kubectl krew install hlf to install hlf plugin 

### Namespace

```bash
kubectl create ns fabric
```

### You need to create CA for each org and ord choose your own storage class (you can find it with kubectl get sc)
```bash
kubectl hlf ca create --storage-class=do-block-storage --capacity=2Gi --name=org1-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric
``` 


```bash
# GET ALL CAs
kubectl get fabriccas.hlf.kungfusoftware.es -A
```

### Peer

register your peers to a CA
```bash
kubectl hlf ca register --name=org1-ca --user=org1-peer1 --secret=peerpw --type=peer --enroll-id enroll --enroll-secret=enrollpw --mspid=Org1MSP --namespace=fabric
```

```bash
kubectl hlf peer create --storage-class=do-block-storage --enroll-id=org2-peer2 --mspid=Org2MSP --enroll-pw=peerpw --capacity=5Gi --name=org2-peer2 --ca-name=org2-ca.fabric --namespace=fabric --statedb=couchdb --version=2.4.1-v0.0.4
```

```bash
# GET ALL PEERs
kubectl get fabricpeers.hlf.kungfusoftware.es -A
```

### Admin Certs

Go to a folder and save the outputs of these
```bash
kubectl hlf ca register --name=org2-ca --user=admin --secret=adminpw --type=admin --enroll-id enroll --enroll-secret=enrollpw --mspid=Org2MSP --namespace=fabric
kubectl hlf ca enroll --name=org2-ca --user=admin --secret=adminpw --ca-name ca  --output org2-peer.yaml --mspid=Org2MSP --namespace=fabric
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
## Add these orgs to your networkConfig.yaml  
```bash
kubectl hlf utils adduser --userPath=org1-peer.yaml --config=networkConfig.yaml --username=admin --mspid=Org1MSP
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
```

### Anchor Peers

```bash
kubectl hlf channel addanchorpeer --channel=mychannel --config=networkConfig.yaml --user=admin --peer=org1-peer1.fabric
```
