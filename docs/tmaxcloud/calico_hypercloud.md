# Calico

kubespray로 Calico 설치를 위해 inventory/tmaxcloud/group_vars/k8s_cluster/k8s-net-calico.yml에서 설정해야하는 값은 다음 하나의 값이다.

```yml
calico_ip_auto_method: "cidr=HOST-POD-NETWORK-SUBNET/CIDR"
```

HOST-POD-NETWORK-SUBNET/CIDR 값은 설치하는 클러스터에서 파드 통신을 위해 사용하는 네트워크의 주소를 CIDR 형식으로 나타낸 것이다.

### 예시

예를 들어 다음과 같이 확인 가능하다.

```bash
$ ip addr show dev enp3s0
3: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 94:de:80:0e:2f:ea brd ff:ff:ff:ff:ff:ff
    inet 192.168.7.93/24 brd 192.168.7.255 scope global enp3s0
       valid_lft forever preferred_lft forever
    inet6 fe80::96de:80ff:fe0e:2fea/64 scope link
       valid_lft forever preferred_lft forever
```

파드 네트워크에 해당하는 인터페이스의 ip 주소로부터 네트워크의 네트워크주소 확인 → 192.168.7.0/24

```yml
calico_ip_auto_method: "cidr=192.168.7.0/24"
```

## AWS 환경에서 IP-in-IP 모드 사용을 위한 설정
파일: inventory/tmaxcloud/group_vars/k8s_cluster/k8s-net-calico.yml
```yml
calico_ipip_mode: 'Always'
```

## IPIP mode 및 vxlan mode 설정
### IPIP mode enable
* ipip mode와 vxlan_mode는 동시에 enable할 수 없으므로 ipip 설정시 vxlan_mode를 disable할 것
* calico_ipip_mode : calico 에서 IP-in-IP를 활성화 할 것 인지 여부 
* calico_ipv4pool_ipip: 기본 ippool에서 IP-in-IP를 활성화 할 것 인지 여부 
* default 설정은 calico-node ipip 설정은 disable, default ip pool ipip는 enable, vxlan mode disable임

```yml
calico_ipv4pool_ipip: "Always"
calico_ipip_mode: "Never"
calico_vxlan_mode: "Never"
```


### Vxlan_mode enable 설정
* ipip mode와 vxlan_mode는 동시에 enable할 수 없으므로 vxlan 설정시 ipip mode를 disable할 것

```yml
calico_ipv4pool_ipip: "Never"
calico_ipip_mode: "Never"
calico_vxlan_mode: "Always"
```


## 구축 이후에 위 설정을 변경하는 방법
### IPIP mode enable 설정
1. ipip mode나 vxlan mode는 클러스터 구축시에 설정함이 바람직하나 불가피하게 구축후에 설정을 변경해야 할 경우 ipip mode enable 전에 vxlan_mode를 disable할 것

calicoctl patch felixConfiguration default --patch '{"spec": {"vxlanEnabled": false}}’
calicoctl patch ipPool default-ipv4-ippool --patch '{"spec":{"vxlanMode":"Never"}}’

2. 이후 ippool의 ipip 설정 enable

kubectl edit ippool default-ipv4-ippool -n kube-system 에서 
ipipMode: "Always"



### vxlan mode enable 설정 
1. ipip 설정 disable

kubectl edit ippool default-ipv4-ippool -n kube-system 에서 
ipipMode: "Never"

2. vxlan mode enable

calicoctl patch felixConfiguration default --patch '{"spec": {"vxlanEnabled": true}}’
calicoctl patch ipPool default-ipv4-ippool --patch '{"spec":{"vxlanMode":"Always"}}’
