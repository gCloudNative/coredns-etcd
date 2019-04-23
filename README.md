

## ETCD Cluster on 3 Nodes
```
172.31.3.128
172.31.3.129
172.31.3.130
```




## Create Corefile
```
cat << EOF > /data/Corefile
.:53 {
    etcd senyang.com {
        stubzones
        path /skydns
        endpoint http://172.31.3.128:2379,http://172.31.3.129:2379,http://172.31.3.130:2379
        upstream /etc/resolv.conf
        fallthrough
    }
    prometheus
    cache 160
    loadbalance
    forward . 8.8.8.8:53 8.8.4.4:53 /etc/resolv.conf
    log
    errors
    debug
}
EOF
```


## Launch ETCD and CoreDNS on Rancher
docker-compose.yml
rancher-compose.yml


## 测试

#### A记录
```
ETCDCTL_API=3 etcdctl put /skydns/com/senyang/www '{"host":"10.194.11.253","ttl":10}'
dig +short @172.31.3.130 A www.senyang.com 
```

#### AAAA记录
```
ETCDCTL_API=3 etcdctl put /coredns/com/senyang/www '{"host":"1002::4:2","ttl":10}' 
dig -t AAAA @172.31.3.130 +short www.senyang.com 
```

#### CNAME记录
```
ETCDCTL_API=3 etcdctl put /coredns/com/senyang/www '{"host":"www.baidu.com","ttl":10}' 
dig -t CNAME @172.31.3.130 +short www.senyang.com
```


#### SRV记录
```
ETCDCTL_API=3 etcdctl put /coredns/com/senyang/www '{"host":"10.194.11.253","port":5000,"ttl":10}'
dig -t SRV @172.31.3.130 +short www.senyang.com
```




## 参考

https://www.cnblogs.com/itzgr/p/9920910.html

https://www.cnblogs.com/mafeng/p/10291891.html

https://www.cnblogs.com/leffss/p/10148507.html

