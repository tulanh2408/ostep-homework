# Tổng quan

Chương trình này cho phép bạn quan sát cách quá trình dịch địa chỉ (address translation) được thực hiện trong một hệ thống sử dụng các thanh ghi cơ sở và thanh ghi giới hạn (base and bounds registers). Tương tự như trước, có hai bước khi chạy chương trình để kiểm tra mức độ hiểu biết của bạn về cơ chế cơ sở và giới hạn. Đầu tiên, hãy chạy chương trình mà không có cờ `-c` để tạo ra một tập hợp các phép dịch và xem bạn có thể tự mình thực hiện chính xác các phép dịch địa chỉ này hay không. Sau đó, khi đã hoàn thành, hãy chạy với cờ `-c` để kiểm tra kết quả của mình.

Trong bài tập này, chúng ta sẽ giả định một không gian địa chỉ hơi khác so với mô hình chuẩn tắc (canonical) vốn có heap và stack nằm ở hai đầu đối diện của không gian. Thay vào đó, chúng ta sẽ giả định rằng không gian địa chỉ gồm một phần mã lệnh (code section), tiếp theo là một stack kích thước cố định (nhỏ), và một vùng nhớ động (heap) có chiều phát triển xuống ngay sau đó, trông tương tự như những gì bạn thấy trong hình bên dưới. Trong cấu hình này, chỉ có một chiều phát triển duy nhất, hướng về các vùng địa chỉ cao hơn của không gian địa chỉ.

```sh
  -------------- 0KB
  |    Code    |
  -------------- 2KB
  |   Stack    |
  -------------- 4KB
  |    Heap    |
  |     |      |
  |     v      |
  -------------- 7KB
  |   (free)   |
  |     ...    |
```

Trong hình trên, thanh ghi giới hạn (bounds register) sẽ được đặt là 7 KB, vì đó là mốc đại diện cho điểm kết thúc của không gian địa chỉ. Các truy xuất đến bất kỳ địa chỉ nào nằm trong giới hạn đều được coi là hợp lệ; các truy xuất vượt quá giá trị này là nằm ngoài giới hạn (out of bounds) và do đó phần cứng sẽ phát sinh một ngoại lệ (exception).

Để chạy với các cờ mặc định, hãy gõ `relocation.py` tại dòng lệnh. Kết quả sẽ tương tự như sau:

```sh
prompt> ./relocation.py 
...
Base-and-Bounds register information:
# Thông tin thanh ghi Cơ sở và Giới hạn:

  Base   : 0x00003082 (decimal 12418)
  Limit  : 472

Virtual Address Trace
# Vết thực thi địa chỉ ảo (Virtual Address Trace)
  VA  0: 0x01ae (decimal:430) -> PA or violation?
  VA  1: 0x0109 (decimal:265) -> PA or violation?
  VA  2: 0x020b (decimal:523) -> PA or violation?
  VA  3: 0x019e (decimal:414) -> PA or violation?
  VA  4: 0x0322 (decimal:802) -> PA or violation?
```

Đối với mỗi địa chỉ ảo (VA), chương trình tạo ra địa chỉ vật lý (PA) tương ứng sau khi dịch HOẶC ghi rõ rằng đó là địa chỉ ngoài vùng giới hạn (lỗi VI PHẠM PHÂN ĐOẠN - segmentation violation). Đối với bài toán này, bạn nên giả định một không gian địa chỉ ảo đơn giản với kích thước cho trước.

Như bạn thấy, bài tập chỉ đơn giản là tạo ra các địa chỉ ảo ngẫu nhiên. Với mỗi địa chỉ, bạn cần xác định xem nó có nằm trong giới hạn hay không, và nếu có, hãy xác định xem nó được dịch sang địa chỉ vật lý nào. Chạy với `-c` (cờ "tính toán giúp cho tôi" - compute this for me) sẽ cho chúng ta biết kết quả của các phép dịch địa chỉ này, tức là liệu chúng có hợp lệ hay không, và nếu hợp lệ thì địa chỉ vật lý thu được là gì. Để thuận tiện, tất cả các con số đã đều được biểu diễn ở cả hệ thập lục phân (hex) và hệ thập phân (decimal).

```sh
prompt> ./relocation.py -c
...
Virtual Address Trace
# Vết thực thi địa chỉ ảo
  VA  0: 0x01ae (decimal:430) -> VALID: 0x00003230 (dec:12848)
  VA  1: 0x0109 (decimal:265) -> VALID: 0x0000318b (dec:12683)
  VA  2: 0x020b (decimal:523) -> SEGMENTATION VIOLATION
  VA  3: 0x019e (decimal:414) -> VALID: 0x00003220 (dec:12832)
  VA  4: 0x0322 (decimal:802) -> SEGMENTATION VIOLATION
```

Với địa chỉ cơ sở là 12418 (thập phân), địa chỉ 430 nằm trong giới hạn (tức là nó nhỏ hơn giá trị của thanh ghi giới hạn (limit register) là 472) và do đó được dịch thành 430 cộng với 12418, bằng 12848. Một vài địa chỉ hiển thị ở trên nằm ngoài giới hạn (523, 802), vì chúng vượt quá giá trị giới hạn. Khá đơn giản phải không? Thật vậy, đó chính là một trong những nét đẹp của cơ chế cơ sở và giới hạn: nó cực kỳ đơn giản!

Có một vài cờ bạn có thể sử dụng để theo dõi tốt hơn cách chương trình diễn ra:

```sh
prompt> ./relocation.py -h
Usage: relocation.py [options]

Options:
  -h, --help            show this help message and exit
                        # hiển thị thông báo trợ giúp và thoát
  -s SEED, --seed=SEED  the random seed
                        # giá trị hạt giống ngẫu nhiên (random seed)
  -a ASIZE, --asize=ASIZE address space size (e.g., 16, 64k, 32m)
                        # kích thước không gian địa chỉ (ví dụ: 16, 64k, 32m)
  -p PSIZE, --physmem=PSIZE physical memory size (e.g., 16, 64k)
                        # kích thước bộ nhớ vật lý (ví dụ: 16, 64k)
  -n NUM, --addresses=NUM # of virtual addresses to generate
                        # số lượng địa chỉ ảo cần tạo
  -b BASE, --b=BASE     value of base register
                        # giá trị thanh ghi cơ sở (base register)
  -l LIMIT, --l=LIMIT   value of limit register
                        # giá trị thanh ghi giới hạn (limit register)
  -c, --compute         compute answers for me
                        # tự động tính toán kết quả
```

Cụ thể, bạn có thể thay đổi kích thước không gian địa chỉ ảo (`-a`), kích thước bộ nhớ vật lý (`-p`), số lượng địa chỉ ảo cần tạo (`-n`), và giá trị của các thanh ghi cơ sở cũng như thanh ghi giới hạn cho tiến trình này (lần lượt là `-b` và `-l`).