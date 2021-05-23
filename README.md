# README
  This project aim at provide a high performance DPDK based proxy, and now support TLS proxy at present, later will support HTTP proxy and HTTPS proxy 
# network topology
* client <----> DUT (dpdkproxy) <----> server 
# packet follow
  * request from client

    CIP -------->  VIP && LIP ---------> RIP
  * response from server

    CIP <--------  VIP && LIP <--------- RIP
# install dpdk and dpdkproxy
  * get the dpdk source code and compile it and setup physic interface and memory for DPDK (if you have igb_uio and rte_kni module installed and bind nic and set memory, ignor this step)
      * step 1: get the dpdk 18.11 LTS version http://fast.dpdk.org/rel/dpdk-18.11.7.tar.xz and assume we get the tar package in director /home/tmp(this will used in step 6)
      ```
       wget http://fast.dpdk.org/rel/dpdk-18.11.7.tar.xz
       tar Jxf dpdk-18.11.7.tar.xz
       cd dpdk-stable-18.11.7
      ```
      * step 2: compile dpdk (aim to get kni and igb_uio kernel module)
      ```
       export RTE_TARGET=x86_64-native-linuxapp-gcc
       make install -j 4 T=x86_64-native-linuxapp-gcc  (numactl-devel and libnuma-dev maybe needed)
      ```
      * step 3: setup hugepage
      ```
       mkdir -p /mnt/huge
       mount -t hugetlbfs nodev /mnt/huge
       echo 768 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
      ```
      * step 4: install kni and igb_uio kernel module
      ```
       modprobe uio
       insmod x86_64-native-linuxapp-gcc/kmod/igb_uio.ko
       insmod x86_64-native-linuxapp-gcc/kmod/rte_kni.ko carrier=on
      ```
      * step 5: bind interface to DPDK
      ```
       #ens35 for example
       ifconfig ens35 down
       python  usertools/dpdk-devbind.py --bind=igb_uio ens35
      ```
      * step 6: write a shell script (we named it with dpdkproxy_if_mem_init.sh) to make nic and memory setup every time system bootup (option if you don't want to run dpdkproxy automatically after system bootup) 
      ```
       mkdir -p /mnt/huge
       mount -t hugetlbfs nodev /mnt/huge
       echo 768 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
       # ens35 for example
       ifconfig ens35 down
       
       echo "Unloading any existing DPDK UIO module"
       /sbin/lsmod | grep -s igb_uio > /dev/null
       if [ $? -eq 0 ] ; then
          sudo /sbin/rmmod igb_uio
       fi
       rmmod igb_uio.ko
       rmmod rte_kni.ko 
       modprobe uio 
       # assume we get the dpdk 
       insmod /home/tmp/dpdk-stable-18.11.7/x86_64-native-linuxapp-gcc/kmod/igb_uio.ko
       insmod /home/tmp/dpdk-stable-18.11.7/x86_64-native-linuxapp-gcc/kmod/rte_kni.ko carrier=on
       python /home/tmp/dpdk-stable-18.11.7/usertools/dpdk-devbind.py --bind=igb_uio ens35
      ```
      * step 7: append following line to /etc/rc.local (needless if you don't execute step 6)
      ```
       echo "/home/tmp/dpdk-stable-18.11.7/dpdkproxy_if_mem_init.sh" >> /etc/rc.local
       chmod +x /etc/rc.local
      ```
  * get the dpdkproxy and install it 
    ```
    git clone https://github.com/wxy279/dpdkproxy.git
    cd dpdkproxy
    mkdir tmp
    tar zxf dproxy.tar.gz -C tmp/
    cd tmp
    pkg-dproxy-install.sh
    /usr/share/dproxy/script/dpdkproxy_start.sh
    ```
# performance
following results were tested on huaweicloud ECS
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

