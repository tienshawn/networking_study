# Tìm hiểu về tables trong iptables

Mục lục:
- 1. Mangle table
- 2. Nat table
- 3. Filter table
- 4. Raw table



## 1. Tổng quan về Mangle table trong iptables

- Nhìn chung, mangle tables được sử dụng để thay đổi thông tin của gói tin. Nói một cách khác, chúng ta có thể sử dụng bảng này thể thay đổi các giạ trị của gói tin, như  trường Kiểu dịch vụ (TOS - Type of Service), TTL - Time to live và những giá trị tương tự.

> Khuyến cáo: không sử dụng bảng này để thực hiện packet filtering, hoặc NAT (SNAT, DNAT, Masquerading)

> Những trường trong gói tin như TOS - Type of Service, TTL - Time to live, và MARK chỉ hợp lệ trong mange table. Những giá trị này không thể được sử dụng ngoài phạm vi của bảng mangle.

### TOS - Type of Service Target

- Target TOS được sử dụng để đặt hoặc thay đổi trường Type of Service trong gói tin. 

- Mục đích có thể là để thiết lập các chính sách (policy), có thể là liên quan đến cách routing gói tin như thế nào,... 

- Tuy nhiên. trường này thì không thường được sử dụng và các routers thường có xu hướng không quan tâm đến giá trị của trường này.


### TTL - Time to Live Target

- Target TTL được sử dụng để thay đổi trường Time to Live của gói tin.

### MARK Target

- Trường MARK dùng để đặt một dấu lên giá trị cụ thể của gói tin. 

- Các gói tin được đánh dấu có thể được áp dụng các chính sách nào đó (giới hạn băng thông, hoặc thực hiện Class Based Queuing)


## 2. Tổng quan về Nat Table

- Bảng này được sử dụng cho việc NAT (Network Address Translation) lên các gói tin khác nhau, hay chính là việc thay đổi trường Source hoặc Destination address. Các targets được sử dụng trong bảng này là:
  - DNAT

  - SNAT

  - MASQUERADE

  - REDIRECT

### Giải thích các Target trong bảng NAT:

- Target DNAT 
   - được sử dụng trong để thay đổi Destination Address của gói tin. 

   ![](../images/dmz-dnat.png)
  
  - Ví dụ như trong mô hình DMZ, ta có các web server host được bảo vệ bởi Firewall. Khi các IP public từ bên ngoài tiếp cận đến Firewall, chúng ta muốn điều hướng những luồng traffic này tới web server của chúng ta. Hay nói một cách khác, thay đổi địa chỉ đích đến của gói tin và định tuyến tới web server host.

  - Điều này có thể là do Web server của chúng ta chỉ có thể được tiếp cận trong dải mạng nội bộ của công ty (không có IP public), hoặc là chúng ta muốn kiểm soát cũng như bảo mật Web server khỏi các mối nguy cơ từ bên ngoài, hoặc LoadBalancing,... Khi đó, các kết nối bên ngoài chỉ được phép kết nối tới Firewall - nơi các Rules được đặt, và Firewall lúc này sẽ đảm nhận nhiệm vụ routing traffic tới nơi cần thiết.

- Target SNAT 
  - được sử dụng để thay đổi Source Address của gói tin. 

  ![](../images/dmz-snat.png)

  - Ví dụ như trong mô hình DMZ, thông thường ta muốn giấu mạng nội bộ khỏi Internet. Để kết nối ra ngoài mạng Internet ta cần IP public và ta có một Firewall có IP public như mô hình trên. Từ Internal Desktop và Public Server ta muốn kết nối ra ngoài Internet.
  Vì sử dụng IP nội bộ nên ta không thể kết nối ra ngoài Internet như cách thông thường. 
  
  - Lúc đó, ta có thể sử dụng Firewall - nơi có IP Public, và thực hiện Source NAT trên đây. Các kết nối LAN sẽ đi qua Router và được thực hiện Source NAT. Từ đó ta có thể kết nối tới Internet từ trong chính Local LAN.


- Target MASQUERADE
  - Mục đích sử dụng tương tự như SNAT. Tuy nhiên, thay vì sử dụng một IP public duy nhất thì ta có thể sử dụng pool IP để thực hiện SNAT cho gói tin.

## 3. Tổng quan về Filter table

- Filter table được sử dụng để lọc gói tin (packet filtering).

- Tuỳ thuộc vào các Rules mà gói tin khi đến đây sẽ được DROP hoặc ACCEPT hay REJECT, tuỳ thuộc vào các trường giá trị trong gói tin.

## 4. Tổng quan về Raw table

- Bảng Raw được sử dụng để đặt dấu lên gói tin để xem gói tin đó có nên được xử lý bởi connection tracking system hay không, bằng cách sử udnjg NOTRACK target.

- Gói tin với NOTRACK target sẽ không được track bởi connection tracking system và ngược lại.

- Về connection tracking system, hệ thống này sẽ liên quan với State machine được giới thiệu ở phần sau.

- Bảng Raw chỉ có PREROUTING và OUTPUT chain, bởi đây là nơi có thể xử lý được gói tin trước khi gói tin đến connection tracking system.

> Lưu ý: Để sử dụng Raw table, ta cần phải load iptable_raw module trước.