# Tổng quan (Overview)

Trong bài tập này, bạn sẽ sử dụng một chương trình đơn giản có tên là `paging-linear-translate.py` để kiểm tra mức độ hiểu biết của mình về cách thức hoạt động của cơ chế chuyển đổi địa chỉ ảo sang vật lý cơ bản với các bảng trang tuyến tính (linear page tables). Để chạy chương trình, hãy nhớ gõ trực tiếp tên chương trình (`./paging-linear-translate.py`) hoặc sử dụng lệnh (`python paging-linear-translate.py`). Khi chạy với cờ `-h` (trợ giúp), bạn sẽ thấy:

```sh
prompt> ./paging-linear-translate.py -h
Usage: paging-linear-translate.py [options]

Options:
-h, --help              show this help message and exit
-s SEED, --seed=SEED    the random seed
-a ASIZE, --asize=ASIZE 
                        address space size (e.g., 16, 64k, ...)
-p PSIZE, --physmem=PSIZE
                        physical memory size (e.g., 16, 64k, ...)
-P PAGESIZE, --pagesize=PAGESIZE
                        page size (e.g., 4k, 8k, ...)
-n NUM, --addresses=NUM number of virtual addresses to generate
-u USED, --used=USED    percent of address space that is used
-v                      verbose mode
-c                      compute answers for me
```

Trước tiên, hãy chạy chương trình mà không truyền vào bất kỳ đối số nào:

```sh
prompt> ./paging-linear-translate.py 
ARG seed 0
ARG address space size 16k
ARG phys mem size 64k
ARG page size 4k
ARG verbose False

The format of the page table is simple:
The high-order (left-most) bit is the VALID bit.
  If the bit is 1, the rest of the entry is the PFN.
  If the bit is 0, the page is not valid.
Use verbose mode (-v) if you want to print the VPN # by
each entry of the page table.
# Định dạng của bảng trang rất đơn giản:
# Bit bậc cao (ngoài cùng bên trái) là bit VALID (hợp lệ).
#   Nếu bit này là 1, phần còn lại của mục nhập là PFN.
#   Nếu bit này là 0, trang đó không hợp lệ (not valid).
# Sử dụng chế độ chi tiết (-v) nếu bạn muốn in số VPN # bên cạnh mỗi mục nhập của bảng trang.

Page Table (from entry 0 down to the max size)
   0x8000000c
   0x00000000
   0x00000000
   0x80000006

Virtual Address Trace
  VA  0: 0x00003229 (decimal:    12841) --> PA or invalid?
  VA  1: 0x00001369 (decimal:     4969) --> PA or invalid?
  VA  2: 0x00001e80 (decimal:     7808) --> PA or invalid?
  VA  3: 0x00002556 (decimal:     9558) --> PA or invalid?
  VA  4: 0x00003a1e (decimal:    14878) --> PA or invalid?
```

Đối với mỗi địa chỉ ảo, hãy viết ra địa chỉ vật lý mà nó được chuyển đổi sang HOẶC ghi rõ rằng đó là một địa chỉ nằm ngoài phạm vi (ví dụ: lỗi phân đoạn - segmentation fault).

Như bạn có thể thấy, những gì chương trình cung cấp cho bạn là một bảng trang dành cho một tiến trình cụ thể (hãy nhớ rằng, trong một hệ thống thực tế dùng bảng trang tuyến tính, mỗi tiến trình sẽ có một bảng trang riêng; ở đây chúng ta chỉ tập trung vào một tiến trình duy nhất, không gian địa chỉ của nó, và do đó là một bảng trang đơn lẻ). Bảng trang cho bạn biết, đối với mỗi số trang ảo (VPN - Virtual Page Number) của không gian địa chỉ, trang ảo đó được ánh xạ tới một số khung trang vật lý (PFN - Physical Frame Number) cụ thể và do đó hợp lệ (valid), hoặc không hợp lệ (not valid).

Định dạng của một mục nhập bảng trang (PTE - page-table entry) rất đơn giản: bit ngoài cùng bên trái (bit bậc cao nhất) là bit hợp lệ (valid bit); các bit còn lại, nếu bit hợp lệ bằng 1, sẽ là PFN.

Trong ví dụ trên, bảng trang ánh xạ VPN 0 tới PFN 0xc (hệ thập phân là 12), VPN 3 tới PFN 0x6 (hệ thập phân là 6), và để lại hai trang ảo còn lại là 1 và 2 ở trạng thái không hợp lệ.

Bởi vì bảng trang là một mảng tuyến tính, những gì được in ra ở trên là bản sao chính xác của những gì bạn sẽ thấy trong bộ nhớ nếu bạn tự mình quan sát các bit. Tuy nhiên, đôi khi sẽ dễ dàng hơn khi sử dụng trình mô phỏng này nếu bạn chạy với cờ chi tiết (`-v`); cờ này cũng sẽ in ra VPN (chỉ số index) trong bảng trang. Từ ví dụ trên, khi chạy với cờ `-v`:

```sh
Page Table (from entry 0 down to the max size)
  [       0]   0x8000000c
  [       1]   0x00000000
  [       2]   0x00000000
  [       3]   0x80000006
```

Nhiệm vụ của bạn là sử dụng bảng trang này để chuyển đổi các địa chỉ ảo được cung cấp trong chuỗi vết (trace) thành các địa chỉ vật lý. Hãy xem xét địa chỉ đầu tiên: VA 0x3229. Để chuyển đổi địa chỉ ảo này thành địa chỉ vật lý, trước tiên chúng ta phải tách nó thành các thành phần cấu thành: số trang ảo (VPN) và độ lệch (offset). Chúng ta làm điều này bằng cách ghi nhận kích thước của không gian địa chỉ và kích thước trang. Trong ví dụ này, không gian địa chỉ được đặt là 16KB (một không gian địa chỉ rất nhỏ) và kích thước trang là 4KB. Do đó, chúng ta biết rằng có 14 bit trong địa chỉ ảo, và độ lệch chiếm 12 bit, còn lại 2 bit dành cho VPN. Như vậy, với địa chỉ 0x3229 của chúng ta, ở dạng nhị phân là 11 0010 0010 1001, chúng ta biết hai bit cao nhất chỉ định VPN. Vì vậy, 0x3229 nằm trên trang ảo 3 với độ lệch là 0x229.

Tiếp theo, chúng ta tra cứu trong bảng trang để xem VPN 3 là hợp lệ và được ánh xạ tới một khung trang vật lý nào đó hay là không hợp lệ, và chúng ta thấy rằng nó thực sự hợp lệ (bit bậc cao là 1) và được ánh xạ tới trang vật lý 6. Do đó, chúng ta có thể tạo địa chỉ vật lý cuối cùng bằng cách lấy trang vật lý 6 ghép với độ lệch, như sau: 0x6000 (trang vật lý đã được dịch bit vào đúng vị trí) OR với 0x0229 (độ lệch), cho ra địa chỉ vật lý cuối cùng: 0x6229. Như vậy, chúng ta có thể thấy rằng địa chỉ ảo 0x3229 chuyển đổi thành địa chỉ vật lý 0x6229 trong ví dụ này.

Để xem phần còn lại của lời giải (sau khi bạn đã tự mình tính toán xong!), chỉ cần chạy với cờ `-c` (như thường lệ):

```sh
...
VA  0: 00003229 (decimal: 12841) --> 00006229 (25129) [VPN 3]
VA  1: 00001369 (decimal:  4969) --> Invalid (VPN 1 not valid)
VA  2: 00001e80 (decimal:  7808) --> Invalid (VPN 1 not valid)
VA  3: 00002556 (decimal:  9558) --> Invalid (VPN 2 not valid)
VA  4: 00003a1e (decimal: 14878) --> 00006a1e (27166) [VPN 3]
```

Dĩ nhiên, bạn có thể thay đổi nhiều tham số trong số này để tạo ra các bài toán thú vị hơn. Hãy chạy chương trình với cờ `-h` để xem có những tùy chọn nào:

* Cờ `-s` thay đổi giá trị hạt giống ngẫu nhiên (random seed) và do đó tạo ra các giá trị bảng trang khác nhau cũng như các địa chỉ ảo khác nhau để chuyển đổi.
* Cờ `-a` thay đổi kích thước của không gian địa chỉ.
* Cờ `-p` thay đổi kích thước của bộ nhớ vật lý.
* Cờ `-P` thay đổi kích thước của một trang.
* Cờ `-n` có thể được sử dụng để tạo ra nhiều địa chỉ cần chuyển đổi hơn (thay vì mặc định là 5).
* Cờ `-u` thay đổi tỷ lệ các ánh xạ hợp lệ, từ 0% (`-u 0`) cho đến 100% (`-u 100`). Giá trị mặc định là 50, nghĩa là khoảng một nửa số trang trong không gian địa chỉ ảo sẽ hợp lệ.
* Cờ `-v` in ra các số VPN để giúp bạn quan sát dễ dàng hơn.