# THỰC HÀNH LAB OPEN VSWITCH SỬ DỤNG MININET

## Setup
Ở bài lab này, thực hành tạo topo gồm 4 host và 1 switch, và không sử dụng controller.

Mô hình sẽ trông giống như sau:
```
    +                                                       +
    |                                                       |
    |                       +-----+                         |
    |                       |     |                         |
    |                       |     |                         |
    |     +----------+      | OVS |     +----------+        |
    |     |   h1     |      |     |     |   h2     |        |
    |     |10.0.0.1  |    1 |     |2    |10.0.0.2  |        |
    |     +----------+------+     +-----+-----------+       |
    |                       |     |                         |
    |     +----------+------+     +-----+-----------+       |
    |     |  h3      |    3 |     |4    |  h4      |        |
    |     | 10.0.0.3 |      |     |     |10.0.0.4  |        |
    |     +----------+      |     |     +----------+        |
    |                       |     |                         |
    |                       |     |                         |
    |                       |     |                         |
    |                       +-----+                         |
    |                                                       |
    |                                                       |
    +                                                       +
```

Sử dụng câu lệnh sau:
```
mn --topo=single,4 --mac --controller=none
```

Kiểm tra OVS Switch và các Port, Host
```
mininet> dump
<Host h1: h1-eth0:10.0.0.1 pid=152939> 
<Host h2: h2-eth0:10.0.0.2 pid=152941> 
<Host h3: h3-eth0:10.0.0.3 pid=152943> 
<Host h4: h4-eth0:10.0.0.4 pid=152945> 
<OVSSwitch s1: lo:127.0.0.1,s1-eth1:None,s1-eth2:None,s1-eth3:None,s1-eth4:None pid=152950>

mininet> net
h1 h1-eth0:s1-eth1
h2 h2-eth0:s1-eth2
h3 h3-eth0:s1-eth3
h4 h4-eth0:s1-eth4
s1 lo:  s1-eth1:h1-eth0 s1-eth2:h2-eth0 s1-eth3:h3-eth0 s1-eth4:h4-eth0

mininet> links
h1-eth0<->s1-eth1 (OK OK) 
h2-eth0<->s1-eth2 (OK OK) 
h3-eth0<->s1-eth3 (OK OK) 
h4-eth0<->s1-eth4 (OK OK) 

mininet> sh ovs-vsctl show
dec56eff-775f-4057-a2e3-35b506adae71
    Bridge s1
        Controller "ptcp:6654"
        fail_mode: secure
        Port s1-eth2
            Interface s1-eth2
        Port s1-eth1
            Interface s1-eth1
        Port s1-eth3
            Interface s1-eth3
        Port s1-eth4
            Interface s1-eth4
        Port s1
            Interface s1
                type: internal
    ovs_version: "2.17.9"

```

Như vậy việc tạo OVS Switch và kết nối tới các Port của Switch với các Host đã thành công. Tiếp theo, ta kiểm tra xem có flow nào được thêm vào chưa
```
mininet> sh ovs-ofctl dump-flows s1
```

Như vậy chưa có flow nào được thêm. Thực hiện kiểm tra kết nối:
```
mininet> pingall
*** Ping: testing ping reachability
h1 -> X X X 
h2 -> X X X 
h3 -> X X X 
h4 -> X X X 
*** Results: 100% dropped (0/12 received)
```

## Thêm các flow để ping được từ h1 tới h3 với nhau:

- Ở đây ta sẽ thêm các flow để điều hướng các gói tin ARP request cũng như ARP reply tới đích đến cần thiết

- Đầu tiên, h1 sẽ gửi 1 bản tin ARP request để biết được địa chỉ của h3. Vì vậy, ta cần thêm flow để forward gói tin này ra port nối với h3
```
sh sudo ovs-ofctl add-flow s1 dl_type=0x0806,nw_dst=10.0.0.3,action=output:3
```

- Tương tự với gói tin ARP request gửi từ h3 tới h1
```
mininet> sh sudo ovs-ofctl add-flow s1 dl_type=0x0806,nw_dst=10.0.0.1,action=output:1
```

- Sau khi h1 nhận được gói tin ARP reply, khi ping nó sẽ tạo các gói tin ICMP request và reply. Ta sẽ add tương ứng các lệnh với gói tin ICMP như với gói tin ARP
```
mininet> sh sudo ovs-ofctl add-flow s1 dl_type=0x0800,nw_dst=10.0.0.3,action=output:3


mininet> sh sudo ovs-ofctl add-flow s1 dl_type=0x0800,nw_dst=10.0.0.1,action=output:1
```

- Thực hiện dump flow ta được kết quả:
```
mininet> sh sudo ovs-ofctl dump-flows s1
 cookie=0x0, duration=64.381s, table=0, n_packets=0, n_bytes=0, arp,arp_tpa=10.0.0.3 actions=output:"s1-eth3"
 cookie=0x0, duration=30.325s, table=0, n_packets=0, n_bytes=0, arp,arp_tpa=10.0.0.1 actions=output:"s1-eth1"
 cookie=0x0, duration=11.848s, table=0, n_packets=0, n_bytes=0, ip,nw_dst=10.0.0.3 actions=output:"s1-eth3"
 cookie=0x0, duration=7.227s, table=0, n_packets=0, n_bytes=0, ip,nw_dst=10.0.0.1 actions=output:"s1-eth1"
```

- Thực hiện ping từ h1 tới h3
```
mininet> h1 ping h3
PING 10.0.0.3 (10.0.0.3) 56(84) bytes of data.
64 bytes from 10.0.0.3: icmp_seq=1 ttl=64 time=0.404 ms
64 bytes from 10.0.0.3: icmp_seq=2 ttl=64 time=0.078 ms
^C
--- 10.0.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1017ms
rtt min/avg/max/mdev = 0.078/0.241/0.404/0.163 ms
```
