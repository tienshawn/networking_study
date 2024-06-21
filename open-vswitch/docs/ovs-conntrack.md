# Conntrack trong Open-vSwitch

## Các khái niệm cơ bản

- Conntrack: là một connection tracking module cho biết trạng thái của kết nối (ví dụ như với TCP thì có các state là SYN_SENT, SYN_RECEIVED,...)

- pipeline: là đường đi packet mà ở đó packet match một flow nào đó và thực hiện action nào đó

- network namespace: là cách ảo hóa routing domain bên trong một instance của linux kernel. 

- flow: là OpenFlow flow, có thể được lập trình bằng OpenFlow controller hoặc OVS command line tool như ovs-ofctl.
  - Flow sẽ có match fields và actions.


## Các fields sử dụng trong OVS Conntrack 

### 1. cs_state: là trạng thái của kết nối match với packet. Các giá trị có thể là:
- new 
- est (ESTABLISHED)
- rel (RELATED)
- rpl (REPLY)
- inv (INVOLVED)
- trk (TRACKED)
- snat 
- dnat

> Ta có thể thêm dấu + hoặc - tương ứng với flag để set hoặc unset flag. VD: ct_state=+trk-new 

### 2. ct_zone: là 

### 3. ct_mark

### 4. ct_label

### 5. ct_nw_src / ct_ipv6_src

### 6. ct_nw_dst / ct_ipv6_dst

### 7. ct_nw_proto

### 8. ct_tp_src

### 9. ct_tp_dst




