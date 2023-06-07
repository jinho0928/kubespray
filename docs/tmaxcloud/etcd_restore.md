# ETCD 백업 및 복구
## 주의사항
#### ETCD 복구 시 지정한 --data-dir 옵션과 etcd.yaml의 --data-dir이 동일한지 체크한다.
#### 동일한 스냅샷 하나를 가지고 모든 마스터(ETCD) 노드 복구를 진행한다.
#### kube-system이 정상화 되지 않을 시, kubelet 재시작 및 etcd&k8s-api-server log 체크한다.

## 1. ETCD 백업
#### etcdctl 세팅 및 멤버조회
마스터 노드에서 아래 명령어를 수행한다.
```yml
alias etcdctl='ETCDCTL_API=3 etcdctl --endpoints=https://{{ endpoint }}:2379 --cacert={{ ca_cert_path }} --cert={{ etcd_cert_path }} --key={{ etcd_key_path }}'

etcdctl member list  --write-out=table
```
#### etcd snapshot (다중 마스터 구성시 마스터 중 1개만 snapshot, 동일한 snapshot으로 모든 마스터(ETCD) 노드 복구)
```yml
etcdctl snapshot save {{ snapshot_save_path }}
```

#### 예시
etcdctl을 사용하기 위해 아래와 같이 설정한다.
```yml
alias etcdctl='ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key'

etcdctl member list  --write-out=table

etcdctl snapshot save /etc/etcd_backup/etcd-`date +%Y%m%d_%H%M%S`
```

## 2. ETCD 복구
#### 기존 data backup 및 삭제
기존 data를 백업시켜 놓은 후 삭제한다.
```yml
mv /var/lib/etcd {{ backup_path }}
rm -rf /var/lib/etcd
```

#### etcdctl restore
snapshot으로 생성된 .db 파일을 모든 etcd노드에 배치 후, restore을 진행한다. (default: --data-dir: /var/lib/etcd)
```yml
[master1에서 진행]
etcdctl snapshot restore {{ snapshot_file_path }} --name {{ master1_name }} --initial-cluster m1=http://{{ master1_ip }}:2380,m2=http://{{ master2_ip }}:2380,m3=http://{{ master3_ip }}:2380 --initial-advertise-peer-urls http://{{ master1_ip }}:2380

[master2에서 진행]
etcdctl snapshot restore {{ snapshot_file_path }} --name {{ master2_name }} --initial-cluster m1=http://{{ master1_ip }}:2380,m2=http://{{ master2_ip }}:2380,m3=http://{{ master3_ip }}:2380 --initial-advertise-peer-urls http://{{ master2_ip }}:2380

[master3에서 진행]
etcdctl snapshot restore {{ snapshot_file_path }} --name {{ master3_name }} --initial-cluster m1=http://{{ master1_ip }}:2380,m2=http://{{ master2_ip }}:2380,m3=http://{{ master3_ip }}:2380 --initial-advertise-peer-urls http://{{ master3_ip }}:2380
...
```

#### 예시
```yml
```
