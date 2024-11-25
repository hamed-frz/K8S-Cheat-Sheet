# Useful Commands: 

- ceph -s
- ceph health detail 
- ceph orch status
- ceph orch host ls
- ceph orch ps
- ceph orch ls
- ceph osd tree
- ceph pg ls
- ceph osd pool ls detail
- ceph config dump
---
**volums of pool**
- rbd ls --pool replicapool -l
---
**Create one file that have 200 blocks of 1MB size**
- dd if=/dev/random  of=./bigfile bs=1M count=200
---
**Create  multiple files -> for performance test**
- for number in {1..100}; do  dd if=/dev/random  of=./bigfile bs=1M count=200 done;

