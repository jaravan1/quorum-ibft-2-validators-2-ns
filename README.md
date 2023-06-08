
# Quorum blockchain network simulation repository w/ 2 validators in different namespaces

### Generate the validators

```
npx quorum-genesis-tool --consensus ibft --chainID 10 --blockperiod 5 --requestTimeout 10 --epochLength 30000 --difficulty 1 --gasLimit '0xe0000000' --coinbase '0x0000000000000000000000000000000000000000' --validators 2 --members 0 --bootnodes 0
```

### Start minikube with calico for network policies:

```
minikube start --network-plugin=cni --cni=calico
```

### Deploy/Remove installation

```bash
./deploy.sh
```

```bash
./remove.sh
```

### Ephemeral container with `tcpdump` 

Start ephemeral container on target pod:

```bash
kubectl debug -it -n quorum1 validator1-0 --image=dockersec/tcpdump -- sh
```

In the command line inside the container start tcpdump:

```bash
tcpdump -n port 30303
```

```
                     TCPDUMP FLAGS
Unskilled =  URG  =  (Not Displayed in Flag Field, Displayed elsewhere) 
Attackers =  ACK  =  (Not Displayed in Flag Field, Displayed elsewhere)
Pester    =  PSH  =  [P] (Push Data)
Real      =  RST  =  [R] (Reset Connection)
Security  =  SYN  =  [S] (Start Connection)
Folks     =  FIN  =  [F] (Finish Connection)
          SYN-ACK =  [S.] (SynAcK Packet)
                     [.] (No Flag Set)
```

#### Monitor establishment of connection between quorum peers

0. Deployment of network policy in namespace `quorum2` will block all inbound traffic to this namespace. Thus, connection is forced to be outbound from validator2-0 and inbound to validator1-0. Connectivity can be checked using a curl pod
1. Start ephemeral container inside pod of validator1-0
2. Delete pod of validator2-0 so as to force a restart
3. While the validator2-0 pod is starting we execute the command `tcpdump -n port 30303` inside the ephemeral container inside validator1-0. In this way we will be able to monitor the establishment of the connection from the start
4. When validator2-0 has started properly we are able to see tcp traffic between validator1-0 and validator2-0 quorum peers
5. We may check the status of the tcp connection inside quorum pods (will be outbound for validator2-0 and inbound for validator1-0):

```bash
netstat -atn
```

### Geth client inside quorum container:

Attach client:

```
geth attach --datadir /data /data/geth.ipc
```

and check peers:

```
admin.peers
```

also blocks increasing:

```
eth.blockNumber
```

### Test connectivity:

```
kubectl run -it --rm --restart=Never mycurlpod --image=curlimages/curl -n quorum1 sh
```

```
curl -w '%{http_code}\n' quorum-validator2.quorum4:8545 --connect-timeout 2
```

#### Useful Links:

- [CURL ERROR: Recv failure: Connection reset by peer](https://stackoverflow.com/questions/10285700/curl-error-recv-failure-connection-reset-by-peer-php-curl)
- [Why no data flow after TCP 3 way handshake?](https://ask.wireshark.org/question/9178/why-no-data-flow-after-tcp-3-way-handshake/)
- [How to TCPdump in Kubernetes !](https://cloudyuga.guru/hands_on_lab/tcpdump_kubernetes)
- [TCP flags](https://gist.github.com/tuxfight3r/9ac030cb0d707bb446c7)
