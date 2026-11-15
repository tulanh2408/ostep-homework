# Tổng quan (Overview)

Sử dụng công cụ `vsfs.py` để nghiên cứu cách trạng thái của hệ thống tệp thay đổi khi các thao tác khác nhau diễn ra. Hệ thống tệp khởi đầu ở trạng thái trống, chỉ có duy nhất một thư mục gốc (root directory). Khi quá trình mô phỏng diễn ra, nhiều thao tác khác nhau sẽ được thực hiện, từ đó dần dần làm thay đổi trạng thái trên đĩa (on-disk state) của hệ thống tệp.

Các thao tác có thể xảy ra bao gồm:

- `mkdir()` - tạo một thư mục mới
- `creat()` - tạo một tệp (trống) mới
- `open()`, `write()`, `close()` - ghi nối thêm một khối (block) vào tệp
- `link()`   - tạo một liên kết cứng (hard link) đến một tệp
- `unlink()` - hủy liên kết một tệp (xóa tệp nếu `linkcnt==0`)

Để hiểu cách hoạt động của bài tập này, trước tiên bạn phải nắm được cách trạng thái trên đĩa của hệ thống tệp này được biểu diễn. Trạng thái của hệ thống tệp được hiển thị bằng cách in ra nội dung của bốn cấu trúc dữ liệu khác nhau:

- bitmap inode (inode bitmap): cho biết những inode nào đã được cấp phát
- inodes: bảng chứa các inode và nội dung của chúng
- bitmap dữ liệu (data bitmap): cho biết những khối dữ liệu (data block) nào đã được cấp phát
- data: cho biết nội dung của các khối dữ liệu

Các bitmap khá dễ hiểu: giá trị 1 biểu thị inode hoặc khối dữ liệu (data block) tương ứng đã được cấp phát, và giá trị 0 biểu thị inode hoặc khối dữ liệu đó còn trống.

Mỗi inode có ba trường:
  Trường đầu tiên chỉ định loại tệp (ví dụ: `f` đại diện cho một tệp thông thường - regular file, `d` đại diện cho một thư mục - directory); 
  Trường thứ hai chỉ định khối dữ liệu nào thuộc về tệp (ở đây, các tệp chỉ có thể ở trạng thái trống - địa chỉ khối dữ liệu đặt là `-1`, hoặc có kích thước đúng một khối với địa chỉ là một số không âm); 
  Trường thứ ba biểu thị số lượng tham chiếu (reference count) của tệp hoặc thư mục đó. 
  
Ví dụ, inode dưới đây là một tệp thông thường, đang trống (trường địa chỉ (address field) là -1), và chỉ có đúng một liên kết trong hệ thống tệp:

```sh
  [f a:-1 r:1]
```

Nếu tệp đó được cấp phát một khối dữ liệu (data block) (giả sử là khối 10 (block 10)), nó sẽ được hiển thị như sau:

```sh
  [f a:10 r:1]
```

Nếu sau đó có ai đó tạo một liên kết cứng (hard link) đến inode này, nó sẽ trở thành:

```sh
  [f a:10 r:2]
```

Cuối cùng, các khối dữ liệu có thể chứa dữ liệu người dùng (user data) hoặc dữ liệu thư mục (directory data). Nếu chứa dữ liệu thư mục, mỗi mục mục lục (entry) bên trong khối sẽ có dạng `(tên, inumber)`, trong đó "tên" (name) là tên của tệp hoặc thư mục, và "inumber" là số hiệu inode của tệp đó. 

Do đó, một thư mục gốc trống sẽ trông như thế này (giả sử inode gốc là 0):

```sh
  [(.,0) (..,0)]
```

Nếu chúng ta thêm một tệp duy nhất "f" vào thư mục gốc và tệp này được cấp phát inode số 1, nội dung thư mục gốc khi đó sẽ trở thành:

```sh
  [(.,0) (..,0) (f,1)]
```

Nếu một khối dữ liệu chứa dữ liệu người dùng, nó sẽ chỉ được hiển thị dưới dạng một ký tự đơn bên trong khối, ví dụ: "h". Nếu nó trống và chưa được cấp phát, chương trình sẽ chỉ hiển thị một cặp dấu ngoặc vuông rỗng (`[]`).

Toàn bộ hệ thống tệp do đó được mô tả như sau:

```sh
inode bitmap 11110000
inodes       [d a:0 r:3] [f a:1 r:1] [f a:-1 r:1] [d a:2 r:2] [] ...
data bitmap  11100000
data         [(.,0) (..,0) (y,1) (z,2) (f,3)] [u] [(.,3) (..,0)] [] ...
```

Hệ thống tệp này có 8 inode và 8 khối dữ liệu. Thư mục gốc chứa ba mục (ngoài "." và "..") trỏ tới "y", "z", và "f". Bằng cách tra cứu inode 1, chúng ta có thể thấy "y" là một tệp thông thường (loại f), có một khối dữ liệu duy nhất được cấp phát cho nó (địa chỉ 1). Trong khối dữ liệu 1 đó chứa nội dung của tệp "y": cụ thể là ký tự "u". Chúng ta cũng có thể thấy "z" là một tệp thông thường trống (trường địa chỉ đặt là -1), và "f" (inode số 3) là một thư mục, cũng đang trống. Bạn cũng có thể thấy từ các bitmap rằng 4 mục đầu tiên trong bitmap inode đã được đánh dấu là cấp phát, cũng như 3 mục đầu tiên trong bitmap dữ liệu.

Trình mô phỏng có thể được chạy với các cờ sau:

```sh
prompt> vsfs.py -h
Usage: vsfs.py [options]

Options:
  -h, --help            show this help message and exit
  -s SEED, --seed=SEED  the random seed
                        # [giá trị hạt giống ngẫu nhiên]
  -i NUMINODES, --numInodes=NUMINODES 
                        number of inodes in file system
                        # [số lượng inode trong hệ thống tệp]
  -d NUMDATA, --numData=NUMDATA 
                        number of data blocks in file system
                        # [số lượng khối dữ liệu trong hệ thống tệp]
  -n NUMREQUESTS, --numRequests=NUMREQUESTS 
                        number of requests to simulate
                        # [số lượng yêu cầu cần mô phỏng]
  -r, --reverse         instead of printing state, print ops
                        # [in ra các thao tác thay vì in ra trạng thái]
  -p, --printFinal      print the final set of files/dirs
                        # [in ra danh sách tệp/thư mục cuối cùng]
  -c, --compute         compute answers for me
                        # [tự động tính toán đáp án]
```

Cách dùng điển hình là chỉ cần chỉ định một giá trị hạt giống ngẫu nhiên (seed) (để tạo ra một bài toán khác nhau), và số lượng yêu cầu cần mô phỏng. Ở chế độ mặc định này, trình mô phỏng in ra trạng thái của hệ thống tệp tại mỗi bước, và hỏi bạn xem thao tác nào đã diễn ra để chuyển hệ thống tệp từ trạng thái này sang trạng thái khác. Ví dụ:

```sh
prompt> ./vsfs.py -n 6 -s 16
...
Initial state
# [Trạng thái ban đầu]

inode bitmap  10000000
inodes        [d a:0 r:2] [] [] [] [] [] [] []
data bitmap   10000000
data          [(.,0) (..,0)] [] [] [] [] [] [] []

Which operation took place?
# [Thao tác nào đã diễn ra?]

inode bitmap  11000000
inodes        [d a:0 r:2] [f a:-1 r:1] [] [] [] [] [] []
data bitmap   10000000
data          [(.,0) (..,0) (y,1)] [] [] [] [] [] [] []

Which operation took place?
# [Thao tác nào đã diễn ra?]

inode bitmap  11000000
inodes        [d a:0 r:2] [f a:1 r:1] [] [] [] [] [] []
data bitmap   11000000
data          [(.,0) (..,0) (y,1)] [u] [] [] [] [] [] []

Which operation took place?
# [Thao tác nào đã diễn ra?]

inode bitmap  11000000
inodes        [d a:0 r:2] [f a:1 r:2] [] [] [] [] [] []
data bitmap   11000000
data          [(.,0) (..,0) (y,1) (m,1)] [u] [] [] [] [] [] []

Which operation took place?
# [Thao tác nào đã diễn ra?]

inode bitmap  11000000
inodes        [d a:0 r:2] [f a:1 r:1] [] [] [] [] [] []
data bitmap   11000000
data          [(.,0) (..,0) (y,1)] [u] [] [] [] [] [] []

Which operation took place?
# [Thao tác nào đã diễn ra?]

inode bitmap  11100000
inodes        [d a:0 r:2] [f a:1 r:1] [f a:-1 r:1] [] [] [] [] []
data bitmap   11000000
data          [(.,0) (..,0) (y,1) (z,2)] [u] [] [] [] [] [] []

Which operation took place?
# [Thao tác nào đã diễn ra?]

inode bitmap  11110000
inodes        [d a:0 r:3] [f a:1 r:1] [f a:-1 r:1] [d a:2 r:2] [] [] [] []
data bitmap   11100000
data          [(.,0) (..,0) (y,1) (z,2) (f,3)] [u] [(.,3) (..,0)] [] [] [] [] []
```

Khi chạy ở chế độ này, trình mô phỏng chỉ hiển thị một chuỗi các trạng thái, và yêu cầu xác định thao tác nào đã dẫn đến những lần chuyển đổi trạng thái này. Chạy với cờ `-c` sẽ cho chúng ta đáp án. Cụ thể: tệp "/y" đã được tạo, một khối đơn được ghi nối tiếp vào tệp, một liên kết cứng từ "/m" tới "/y" được tạo, "/m" bị xóa thông qua lời gọi `unlink`, tệp "/z" được tạo, và thư mục "/f" được tạo:

```sh
prompt> vsfs.py -n 6 -s 16 -c
...
Initial state
# [Trạng thái ban đầu]

inode bitmap  10000000
inodes        [d a:0 r:2] [] [] [] [] [] [] []
data bitmap   10000000
data          [(.,0) (..,0)] [] [] [] [] [] [] []

creat("/y");

inode bitmap  11000000
inodes        [d a:0 r:2] [f a:-1 r:1] [] [] [] [] [] []
data bitmap   10000000
data          [(.,0) (..,0) (y,1)] [] [] [] [] [] [] []

fd=open("/y", O_WRONLY|O_APPEND); write(fd, buf, BLOCKSIZE); close(fd);

inode bitmap  11000000
inodes        [d a:0 r:2] [f a:1 r:1] [] [] [] [] [] []
data bitmap   11000000
data          [(.,0) (..,0) (y,1)] [u] [] [] [] [] [] []

link("/y", "/m");

inode bitmap  11000000
inodes        [d a:0 r:2] [f a:1 r:2] [] [] [] [] [] []
data bitmap   11000000
data          [(.,0) (..,0) (y,1) (m,1)] [u] [] [] [] [] [] []

unlink("/m");

inode bitmap  11000000
inodes        [d a:0 r:2] [f a:1 r:1] [] [] [] [] [] []
data bitmap   11000000
data          [(.,0) (..,0) (y,1)] [u] [] [] [] [] [] []

creat("/z");

inode bitmap  11100000
inodes        [d a:0 r:2] [f a:1 r:1] [f a:-1 r:1] [] [] [] [] []
data bitmap   11000000
data          [(.,0) (..,0) (y,1) (z,2)] [u] [] [] [] [] [] []

mkdir("/f");

inode bitmap  11110000
inodes        [d a:0 r:3] [f a:1 r:1] [f a:-1 r:1] [d a:2 r:2] [] [] [] []
data bitmap   11100000
data          [(.,0) (..,0) (y,1) (z,2) (f,3)] [u] [(.,3) (..,0)] [] [] [] [] []
```

Bạn cũng có thể chạy trình mô phỏng ở chế độ "đảo ngược" (với cờ `-r`), in ra các thao tác thay vì trạng thái để xem liệu bạn có thể dự đoán được các thay đổi trạng thái từ các thao tác đã cho hay không:

```sh
prompt> ./vsfs.py -n 6 -s 16 -r
Initial state
# [Trạng thái ban đầu]

inode bitmap  10000000
inodes        [d a:0 r:2] [] [] [] [] [] [] [] 
data bitmap   10000000
data          [(.,0) (..,0)] [] [] [] [] [] [] [] 

creat("/y");

  State of file system (inode bitmap, inodes, data bitmap, data)?
  # [Trạng thái của hệ thống tệp (inode bitmap, inodes, data bitmap, data)?]

fd=open("/y", O_WRONLY|O_APPEND); write(fd, buf, BLOCKSIZE); close(fd);

  State of file system (inode bitmap, inodes, data bitmap, data)?
  # [Trạng thái của hệ thống tệp (inode bitmap, inodes, data bitmap, data)?]

link("/y", "/m");

  State of file system (inode bitmap, inodes, data bitmap, data)?
  # [Trạng thái của hệ thống tệp (inode bitmap, inodes, data bitmap, data)?]

unlink("/m")

  State of file system (inode bitmap, inodes, data bitmap, data)?
  # [Trạng thái của hệ thống tệp (inode bitmap, inodes, data bitmap, data)?]

creat("/z");

  State of file system (inode bitmap, inodes, data bitmap, data)?
  # [Trạng thái của hệ thống tệp (inode bitmap, inodes, data bitmap, data)?]

mkdir("/f");

  State of file system (inode bitmap, inodes, data bitmap, data)?
  # [Trạng thái của hệ thống tệp (inode bitmap, inodes, data bitmap, data)?]
```

Một vài cờ khác dùng để điều khiển các khía cạnh khác nhau của mô phỏng, bao gồm số lượng inode (`-i`), số lượng khối dữ liệu (`-d`), và liệu có in ra danh sách cuối cùng của toàn bộ các thư mục và tệp trong hệ thống tệp hay không (`-p`).