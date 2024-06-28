# LAB với tính năng Open vSwitch conntrack

## Mô hình bài Lab:

```
    +                                                       +
    |                                                       |
    |                       +-----+                         |
    |                       |     |                         |
    |                       |     |                         |
    |     +----------+      | OVS |     +----------+        |
    |     |   VM1    |      |     |     |   VM2    |        |
    |     |10.10.0.10|      |     |     |10.10.0.21|        |
    +-----+          +------+     +-----+          +--------+
    |     |          |    A |     | B   |          |        |
    |     |          |      |     |     |          |        |
    |     +----------+      |     |     +----------+        |
    |                       |     |                         |
    |                       |     |                         |
    |                       |     |                         |
    |                       +-----+                         |
    |                                                       |
    |                                                       |
    +                                                       +

```
A, B tương ứng là các port gắn từ OVS vào trong VM (vnet1, vnet2)

## Thực hành:

- Tạo OVS bridge:

```ovs-vsctl add-br ovs-conntrack```

- Thêm các flow để thực hiện connetion tracking
```
#flow-1
ovs-ofctl add-flow ovs-conntrack "table=0, priority=50, ct_state=-trk, tcp, in_port=vnet1, actions=ct(table=0)"

#flow-2
ovs-ofctl add-flow ovs-conntrack "table=0, priority=50, ct_state=+trk+new, tcp, in_port=vnet1, actions=ct(commit),vnet2"
```

- Kiểm tra trạng thái của connection tracking:
  - Giả lập luồng tcp trao đổi giữa 2 VM sử dụng netcat:
  
  ```
  Ở VM2: nc -l 10.0.0.2 5000

  Ở VM1: nc -v 10.0.0.2 5000
  ```

  - Sử dụng câu lệnh sau để kiểm tra:
   
  ```
  watch "ovs-appctl dpctl/dump-conntrack | grep 10.0.10"
  ```

  Kết quả: ta có thể thấy được trạng thái SYN_SENT của connection
  ```
  tcp,orig=(src=10.0.10.1,dst=10.0.10.2,sport=55838,dport=5000),reply=(src=10.0.10.2,dst=10.0.10.1,sport=5000,dport=55838),protoinfo=(state=SYN_SENT)
  ```

  - Tiếp tục, ở phía còn lại, ta cũng thêm các flow để conntrack tương tự:
  ```
  #flow-3
  ovs-ofctl add-flow ovs-conntrack "table=0, priority=50, ct_state=-trk, tcp, in_port=vnet2, actions=ct(table=0)"

  #flow-4
  ovs-ofctl add-flow ovs-conntrack "table=0, priority=50, ct_state=+trk+est, tcp, in_port=vnet2, actions=vnet1"
  ```

 - Kết quả: ta có thể thấy trạng thái đã chuyển sang thành ESTABLISHED
    ```
    tcp,orig=(src=10.0.10.1,dst=10.0.10.2,sport=58650,dport=5000),reply=(src=10.0.10.2,dst=10.0.10.1,sport=5000,dport=58650),protoinfo=(state=ESTABLISHED)
    ```

 - Khi ngắt kết nối, trạng thái chuyển sang TIME_WAIT:
    ```
    tcp,orig=(src=10.0.10.1,dst=10.0.10.2,sport=58650,dport=5000),reply=(src=10.0.10.2,dst=10.0.10.1,sport=5000,dport=58650),protoinfo=(state=TIME_WAIT)
    ```
    > Tuy nhiên, theo đúng lý thuyết thì sẽ phải có gói tin ACK từ phía VM1 nữa (VM1 gửi yêu cầu SYN, VM2 gửi SYN_ACK, VM1 gửi ACK). Nhưng ở trên ta mới track 2 gói tin đầu, tức còn thiếu gói tin ACK từ phía VM1. Trạng thái của kết nối sẽ sớm bị đóng nếu không có dữ liệu nào được truyển đi.

    > Để sửa, ta có thể thêm flow sau:
    ```
    #flow-5
    ovs-ofctl add-flow ovs-conntrack "table=0, priority=50, ct_state=+trk+est, tcp, in_port=vnet1, actions=vnet2"
    ```
### Flow matching tương ứng với các trạng thái của TCP state

