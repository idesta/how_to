
# IPSec Configuration on Mikrotic Router


> **/ip ipsec profile**
```
add dh-group=modp2048 enc-algorithm=aes-256 name=ET-kekros-prof1 nat-traversal=no
```

> **/ip ipsec peer**
```
add address=10.208.71.236/32 local-address=10.131.113.116 name=ET-peer1-kekros01 profile=ET-kekros-prof1
```

> **/ip ipsec identity**
```
add peer=ET-peer1-kekros01 [secret="preshared key"]
```

> **/ip ipsec proposal**
```
add enc-algorithms=aes-256-cbc lifetime=1h name=ET-peer1-kekros01-proposal01
```

> **/ip ipsec policy**

```
add dst-address=10.200.200.226/32 peer=ET-peer1-kekros01 proposal=ET-peer1-kekros01-proposal01 src-address=172.16.150.234/32 tunnel=yes
```








