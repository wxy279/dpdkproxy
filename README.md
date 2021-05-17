# README
  This project aim at provide a high performance DPDK based proxy, and now support TLS proxy at present, later will support HTTP proxy and HTTPS proxy 
# network topology
test

# install
  * get the executable 
  * get the dpdk source code and compile it and setup physic interface and memory for DPDK 
      * git clone
      * compilie
          * export RTE_TARGET=x86_64-native-linuxapp-gcc
          * make install T=x86_64-native-linuxapp-gcc
      * setup hugepage
      ```
      mkdir -p /mnt/huge
      mount -t hugetlbfs nodev /mnt/huge
      echo 768 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
      ```
      * install kni and igb_uio kernel module
      ```
      modprobe uio
      insmod x86_64-native-linuxapp-gcc/kmod/igb_uio.ko
      insmod x86_64-native-linuxapp-gcc/kmod/rte_kni.ko carrier=on
      ```
      * bind interface to DPDK
      ```
      # ens35 for example
      ifconfig ens35 down
      python  usertools/dpdk-devbind.py --bind=igb_uio ens35
      ```
  
  * start dpdkproxy with default configuration
# performance

|                                  |      1 thread   |2 thread    |4 thread   |
| :------------------------------: | :-------------: | :--------: | :--------:|
| ECDHE-ECDSA-AES128-SHA256 CPS    | 2570            |  5511      |    10224  |
| ECDHE-ECDSA-AES128-SHA256 QPS    | 43788           |  92764     |    151630 |
| ECDHE-RSA-AES128-SHA256 CPS(2k)  | 921             |  1881      |    3703   |
| ECDHE-RSA-AES128-SHA256 QPS(2k)  | 33911           |  69301     |  127809   |
| ECDHE-RSA-AES128-SHA256 CPS(1k)  | 2201            |  4253      |    8344   |
| ECDHE-RSA-AES128-SHA256 QPS(1k)  | 45802           |  87140     |   150164  |
| AES128-SHA256(2k) CPS            | 1087            |  2199      |    4358   |
| AES128-SHA256(2k) QPS            | 35722           |  73896     |   132915  |
| AES128-SHA256(1k) CPS            | 3381            |  6689      |    12490  |
| AES128-SHA256(1k) QPS            | 49338           |  95448     |   154565  |
| SM2-WITH-SMS4-SM3 CPS            | 444             |  930       |   1880    |
| SM2-WITH-SMS4-SM3 QPS            | 15607           |  31520     |   50727   |

