# Tổng quan (Overview)

Chương trình này cho phép bạn quan sát cách quá trình chuyển đổi địa chỉ (address translation) được thực hiện trong một hệ thống sử dụng kỹ thuật phân đoạn (segmentation). Cơ chế phân đoạn mà hệ thống này sử dụng tương đối đơn giản: một không gian địa chỉ (address space) chỉ có đúng *hai* đoạn (segment); ngoài ra, bit cao nhất (top bit) của địa chỉ ảo do tiến trình tạo ra sẽ xác định xem địa chỉ đó thuộc về đoạn nào: `0` cho đoạn 0 (segment 0 - nơi chứa mã lệnh và heap) và `1` cho đoạn 1 (segment 1 - nơi chứa ngăn xếp/stack). Đoạn 0 phát triển theo chiều dương (về phía các địa chỉ cao hơn), trong khi đoạn 1 phát triển theo chiều âm (về phía các địa chỉ thấp hơn).

Về mặt trực quan, không gian địa chỉ có cấu trúc như sau:

```
 --------------- virtual address 0 (địa chỉ ảo 0)
 |    seg0     |
 |             |
 |             |
 |-------------|
 |             |
 |             |
 |             |
 |             |
 |(unallocated)| (chưa cấp phát)
 |             |
 |             |
 |             |
 |-------------|
 |             |
 |    seg1     |
 |-------------| virtual address max (địa chỉ ảo cực đại - kích thước không gian địa chỉ)
```

Như bạn đã biết đối với kỹ thuật phân đoạn, mỗi đoạn sẽ đi kèm với một cặp thanh ghi base/limit (cơ sở/giới hạn). Do đó, trong bài toán này sẽ có hai cặp base/limit. Thanh ghi base của đoạn 0 cho biết địa chỉ vật lý mà *phần đầu* (top) của đoạn 0 được đặt vào trong bộ nhớ vật lý, còn limit cho biết kích thước của đoạn đó; thanh ghi base của đoạn 1 cho biết vị trí mà *phần đáy* (bottom) của đoạn 1 được đặt trong bộ nhớ vật lý, và limit tương ứng cũng cho biết kích thước của đoạn (hoặc đoạn đó phát triển theo chiều âm bao xa).

Tương tự như trước, có hai bước chạy chương trình để kiểm tra mức độ hiểu biết của bạn về kỹ thuật phân đoạn. Đầu tiên, hãy chạy chương trình mà không có cờ `-c` để sinh ra một tập các phép chuyển đổi và thử tự mình thực hiện chuyển đổi địa chỉ xem có chính xác hay không. Sau đó, khi đã tính xong, hãy chạy với cờ `-c` để đối chiếu kết quả.

Ví dụ, để chạy với các cờ mặc định, hãy nhập:

```sh
prompt> ./segmentation.py 
```
hoặc

```sh 
prompt> python ./segmentation.py 
```

Bạn sẽ thấy kết quả như sau:
```sh
  ARG seed 0
  ARG address space size 1k
  ARG phys mem size 16k
  
  Segment register information:
  # Thông tin thanh ghi đoạn:

    Segment 0 base  (grows positive) : 0x00001aea (decimal 6890)
    Segment 0 limit                  : 472

    Segment 1 base  (grows negative) : 0x00001254 (decimal 4692)
    Segment 1 limit                  : 450

  Virtual Address Trace
  # Vết địa chỉ ảo:
    VA  0: 0x0000020b (decimal:  523) --> PA or segmentation violation?
    VA  1: 0x0000019e (decimal:  414) --> PA or segmentation violation?
    VA  2: 0x00000322 (decimal:  802) --> PA or segmentation violation?
    VA  3: 0x00000136 (decimal:  310) --> PA or segmentation violation?
    VA  4: 0x000001e8 (decimal:  488) --> PA or segmentation violation?

  For each virtual address, either write down the physical address it translates
  to OR write down that it is an out-of-bounds address (a segmentation
  violation). For this problem, you should assume a simple address space with
  two segments: the top bit of the virtual address can thus be used to check
  whether the virtual address is in segment 0 (topbit=0) or segment 1
  (topbit=1). Note that the base/limit pairs given to you grow in different
  directions, depending on the segment, i.e., segment 0 grows in the positive
  direction, whereas segment 1 in the negative.  
  # Với mỗi địa chỉ ảo, hãy viết ra địa chỉ vật lý tương ứng HOẶC ghi nhận rằng đó là địa chỉ
  # vượt giới hạn (lỗi vi phạm phân đoạn - segmentation violation). Trong bài toán này, bạn cần
  # giả định một không gian địa chỉ đơn giản với hai đoạn: do đó bit cao nhất của địa chỉ ảo có thể
  # được dùng để kiểm tra xem địa chỉ ảo thuộc đoạn 0 (topbit=0) hay đoạn 1 (topbit=1). Lưu ý rằng
  # các cặp base/limit được cấp sẽ phát triển theo các hướng khác nhau tùy thuộc vào từng đoạn,
  # nghĩa là đoạn 0 phát triển theo chiều dương, trong khi đoạn 1 phát triển theo chiều âm.
```

Sau khi bạn đã tự tính toán xong các phép chuyển đổi trong vết địa chỉ ảo, hãy chạy lại chương trình với cờ `-c`. Bạn sẽ thấy kết quả như sau (không bao gồm các thông tin lặp lại):

```sh
  Virtual Address Trace
  # Vết địa chỉ ảo:
    VA  0: 0x0000020b (decimal:  523) --> SEGMENTATION VIOLATION (SEG1)
    VA  1: 0x0000019e (decimal:  414) --> VALID in SEG0: 0x00001c88 (decimal: 7304)
    VA  2: 0x00000322 (decimal:  802) --> VALID in SEG1: 0x00001176 (decimal: 4470)
    VA  3: 0x00000136 (decimal:  310) --> VALID in SEG0: 0x00001c20 (decimal: 7200)
    VA  4: 0x000001e8 (decimal:  488) --> SEGMENTATION VIOLATION (SEG0)
```

Như bạn có thể thấy, khi dùng cờ `-c`, chương trình sẽ thực hiện chuyển đổi các địa chỉ cho bạn, qua đó bạn có thể kiểm tra xem mình đã hiểu cách một hệ thống sử dụng phân đoạn thực hiện chuyển đổi địa chỉ hay chưa.

Tất nhiên, có một số tham số bạn có thể dùng để tạo ra các đề bài khác nhau. Một tham số đặc biệt quan trọng là `-s` hoặc `-seed`, cho phép bạn sinh ra các bài toán khác nhau bằng cách truyền vào một giá trị hạt giống ngẫu nhiên (random seed) khác. Hãy chắc chắn rằng bạn dùng cùng một giá trị seed khi tạo đề bài và khi kiểm tra đáp án.

Ngoài ra còn có các tham số giúp bạn thử nghiệm với các kích thước khác nhau của không gian địa chỉ và bộ nhớ vật lý. Ví dụ, để thử nghiệm kỹ thuật phân đoạn trên một hệ thống siêu nhỏ, bạn có thể nhập:

```sh
prompt> ./segmentation.py -s 100 -a 16 -p 32
ARG seed 0
ARG address space size 16
ARG phys mem size 32
 
Segment register information:
# Thông tin thanh ghi đoạn:

  Segment 0 base  (grows positive) : 0x00000018 (decimal 24)
  Segment 0 limit                  : 4

  Segment 1 base  (grows negative) : 0x00000012 (decimal 18)
  Segment 1 limit                  : 5

Virtual Address Trace
# Vết địa chỉ ảo:
  VA  0: 0x0000000c (decimal:   12) --> PA or segmentation violation?
  VA  1: 0x00000008 (decimal:    8) --> PA or segmentation violation?
  VA  2: 0x00000001 (decimal:    1) --> PA or segmentation violation?
  VA  3: 0x00000007 (decimal:    7) --> PA or segmentation violation?
  VA  4: 0x00000000 (decimal:    0) --> PA or segmentation violation?
```

Lệnh trên yêu cầu chương trình sinh ra các địa chỉ ảo cho một không gian địa chỉ có kích thước 16 byte nằm ở một vị trí nào đó trong bộ nhớ vật lý 32 byte. Như bạn thấy, các địa chỉ ảo sinh ra đều rất nhỏ (12, 8, 1, 7 và 0). Đồng thời, chương trình cũng chọn các giá trị thanh ghi base và limit rất nhỏ tương ứng. Hãy chạy cùng cờ `-c` để xem đáp án.

Ví dụ này cũng sẽ cho bạn thấy chính xác ý nghĩa của từng cặp base/limit. Cụ thể, base của đoạn 0 được đặt tại địa chỉ vật lý 24 (hệ thập phân) và có kích thước 4 byte. Do đó, các địa chỉ *ảo* 0, 1, 2 và 3 thuộc về đoạn 0, hợp lệ, và lần lượt ánh xạ tới các địa chỉ vật lý 24, 25, 26 và 27.

Phức tạp hơn một chút là đoạn 1 phát triển theo chiều âm. Trong ví dụ siêu nhỏ ở trên, thanh ghi base của đoạn 1 được đặt tại địa chỉ vật lý 18, với kích thước 5 byte. Điều này có nghĩa là 5 byte *cuối cùng* của không gian địa chỉ ảo, trong trường hợp này là 11, 12, 13, 14 và 15, là các địa chỉ ảo hợp lệ, và chúng lần lượt ánh xạ tới các địa chỉ vật lý 13, 14, 15, 16 và 17.

Nếu bạn thấy chưa hiểu rõ, hãy đọc lại phần này một lần nữa -- bạn bắt buộc phải hiểu rõ cơ chế hoạt động này thì mới có thể giải được các bài tập dạng này.

Lưu ý rằng bạn có thể chỉ định các giá trị lớn hơn bằng cách thêm hậu tố "k", "m", hoặc thậm chí "g" vào các giá trị truyền vào cùng cờ `-a` hoặc `-p`, tương ứng với "kilobyte", "megabyte", và "gigabyte". Ví dụ, nếu bạn muốn thực hiện một số phép chuyển đổi với không gian địa chỉ 1-MB đặt trong bộ nhớ vật lý 32-MB, bạn có thể nhập:

```sh
prompt> ./segmentation.py -a 1m -p 32m
```

Nếu muốn thiết lập chi tiết hơn nữa, bạn có thể tự mình đặt các giá trị thanh ghi base và limit thông qua các tùy chọn `--b0`, `--l0`, `--b1`, và `--l1`. Hãy tự mình thử nghiệm xem sao.

Cuối cùng, bạn luôn có thể chạy:

```sh
prompt> ./segmentation.py -h 
```

để xem danh sách đầy đủ các cờ và tùy chọn.

Chúc bạn học tốt!