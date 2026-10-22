
# Tổng quan

Chương trình `process-run.py` này cho phép bạn quan sát trạng thái của một tiến trình (process state) thay đổi như thế nào khi nó đang chạy trên CPU. Như đã mô tả ở chương này, các tiến trình có thể ở một trong những trạng thái sau đây:

```sh
RUNNING (Đang chạy) - Tiến trình đang trực tiếp thực thi trên CPU
READY (Sẵn sàng)    - Tiến trình đã sẵn sàng để sử dụng CPU ngay lúc này
                      (nhưng một tiến trình khác vẫn đang chiếm giữ CPU)
BLOCKED (Bị chặn)   - Tiến trình đang chờ một thao tác vào/ra (I/O)
                      (ví dụ: nó vừa phát ra yêu cầu truy xuất tới đĩa)
DONE (Hoàn tất)     - Tiến trình đã được thực thi xong toàn bộ.
```

Trong bài tập này, chúng ta sẽ xem cách các trạng thái tiến trình thay đổi khi một chương trình chạy, và từ đó hiểu rõ hơn một chút về cách thức mà chúng hoạt động.

Để chạy chương trình và xem các tùy chọn (options) của nó, hãy thực hiện lệnh sau:

```sh
prompt> ./process-run.py -h
```

Nếu nó không hoạt động, nhập `python` ở phía trước câu lệnh như sau:

```sh
prompt> python process-run.py -h
```

Đây là kết quả bạn sẽ nhìn thấy:

```sh
Usage: process-run.py [options]

Options:
  -h, --help            Hiển thị tóm tắt hướng dẫn lệnh và thoát
  -s SEED, --seed=SEED  Giá trị hạt giống ngẫu nhiên (random seed)
  -l PROCESS_LIST, --processlist=PROCESS_LIST
                        Danh sách các tiến trình cần chạy, có dạng X1:Y1,X2:Y2,... trong đó X là số lượng lệnh mà tiến trình đó cần chạy, và Y là xác suất (từ 0 đến 100) để một lệnh sẽ sử dụng CPU hoặc phát ra một yêu cầu I/O

  -L IO_LENGTH, --iolength=IO_LENGTH
                        Thời gian để một thao tác I/O hoàn thành
  -S PROCESS_SWITCH_BEHAVIOR, --switch=PROCESS_SWITCH_BEHAVIOR
                        Hành vi chuyển đổi giữa các tiến trình: SWITCH_ON_IO,
                        SWITCH_ON_END

  -I IO_DONE_BEHAVIOR, --iodone=IO_DONE_BEHAVIOR
                        Hành vi xử lý khi I/O kết thúc: IO_RUN_LATER,
                        IO_RUN_IMMEDIATE
  -c                    Chương trình tự thực thi mô phỏng và tính toán các chỉ số
  -p, --printstats      In thông tin thống kê chỉ số chi tiết ở cuối phiên chạy; chỉ hữu ích khi dùng kèm cờ -c
                        (Nếu không có cờ -c thì kết quả thống kê sẽ không được in ra)
```

Tùy chọn quan trọng nhất cần nắm vững là PROCESS_LIST (được chỉ định qua cờ `-l` hoặc `--processlist`), giúp xác định chính xác những gì mỗi chương trình đang chạy (hay 'tiến trình') sẽ thực hiện. Một tiến trình bao gồm các lệnh (instructions), và mỗi lệnh chỉ thực hiện một trong hai việc: 
- Sử dụng CPU
- Phát ra một yêu cầu I/O (và chờ cho đến khi nó hoàn tất)

Khi một tiến trình sử dụng CPU (và không thực hiện I/O), nó sẽ chỉ đơn thuần chuyển đổi luân phiên giữa hai trạng thái RUNNING trên CPU hoặc ở trạng thái READY chờ đến lượt chạy. Ví dụ, dưới đây là một lần chạy đơn giản chỉ có duy nhất một tiến trình đang chạy, và tiến trình đó chỉ sử dụng CPU (không thực hiện bất kỳ thao tác I/O nào).

```sh
prompt> ./process-run.py -l 5:100 
Produce a trace of what would happen when you run these processes: 
# Tạo vết thực thi cho những gì đã xảy ra khi bạn chạy tiến trình này
Process 0
  cpu
  cpu
  cpu
  cpu
  cpu

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)

# Hành vi quan trọng:
  # Hệ thống sẽ chuyển đổi khi tiến trình hiện tại HOÀN TẤT hoặc PHÁT RA YÊU CẦU I/O
  # Sau khi I/O hoàn tất, tiến trình phát yêu cầu I/O sẽ chạy SAU (khi đến lượt của nó)

prompt> 
```

Ở đây, tiến trình chúng ta chỉ định là "5:100", nghĩa là nó bao gồm 5 chỉ thị, và xác suất để mỗi lệnh là một lệnh dùng CPU là 100%.

Bạn có thể xem chuyện gì xảy ra với tiến trình này bằng cách thêm cờ -c để chương trình tự tính toán kết quả cho bạn:

```sh
prompt> ./process-run.py -l 5:100 -c
Time     PID: 0        CPU        IOs
  1     RUN:cpu          1
  2     RUN:cpu          1
  3     RUN:cpu          1
  4     RUN:cpu          1
  5     RUN:cpu          1
```

Kết quả này thực sự không quá thú vị: tiến trình chỉ đơn giản ở trạng thái RUN rồi kết thúc, sử dụng CPU trong toàn bộ thời gian (giữ cho CPU luôn bận rộn suốt thời gian chạy) và không thực hiện bất kỳ thao tác I/O nào.

Hãy làm cho bài toán phức tạp hơn bằng cách cho chạy hai tiến trình cùng lúc:

```sh
prompt> ./process-run.py -l 5:100,5:100
Produce a trace of what would happen when you run these processes: 
# Tạo vết thực thi cho những gì sẽ xảy ra khi chạy các tiến trình này
Process 0
  cpu
  cpu
  cpu
  cpu
  cpu

Process 1
  cpu
  cpu
  cpu
  cpu
  cpu

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)

# Hành vi quan trọng:
  # Hệ thống sẽ chuyển đổi khi tiến trình hiện tại HOÀN TẤT hoặc PHÁT YÊU CẦU I/O
  # Sau khi I/O hoàn tất, tiến trình phát yêu cầu I/O sẽ chạy SAU (khi đến lượt của nó)
```

Trong trường hợp này, hai tiến trình khác nhau cùng chạy và mỗi tiến trình cũng chỉ sử dụng CPU. Điều gì sẽ xảy ra khi hệ điều hành thực thi chúng? Hãy cùng tìm hiểu:

```sh
prompt> ./process-run.py -l 5:100,5:100 -c
Time     PID: 0     PID: 1        CPU        IOs
  1     RUN:cpu      READY          1
  2     RUN:cpu      READY          1
  3     RUN:cpu      READY          1
  4     RUN:cpu      READY          1
  5     RUN:cpu      READY          1
  6        DONE    RUN:cpu          1
  7        DONE    RUN:cpu          1
  8        DONE    RUN:cpu          1
  9        DONE    RUN:cpu          1
 10        DONE    RUN:cpu          1
```

Như bạn thấy ở trên, đầu tiên tiến trình có "mã định danh tiến trình" (hay "PID") bằng 0 sẽ chạy, trong khi tiến trình 1 ở trạng thái READY (sẵn sàng chạy) nhưng phải đợi cho đến khi tiến trình 0 hoàn tất. Khi tiến trình 0 thực thi xong, nó chuyển trạng thái sang DONE, đồng thời tiến trình 1 sẽ bắt đầu được chạy. Khi tiến trình 1 kết thúc, chuỗi thực thi (trace) hoàn tất.

Hãy xem thêm một ví dụ nữa trước khi bước vào trả lời các câu hỏi. Trong ví dụ này, tiến trình chỉ phát ra các yêu cầu I/O. Tại đây, chúng ta quy định rằng mỗi thao tác I/O cần 5 đơn vị thời gian để hoàn thành thông qua cờ -L.

```sh
prompt> ./process-run.py -l 3:0 -L 5
Produce a trace of what would happen when you run these processes: 
# Tạo vết thực thi cho những gì sẽ xảy ra khi chạy các tiến trình này:
Tiến trình 0
  io
  io_done
  io
  io_done
  io
  io_done

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)
# Hành vi quan trọng:
  # Hệ thống sẽ chuyển đổi khi tiến trình hiện tại HOÀN TẤT hoặc PHÁT YÊU CẦU I/O
  # Sau khi I/O hoàn tất, tiến trình phát yêu cầu I/O sẽ chạy SAU (khi đến lượt của nó)
```

Bạn nghĩ chuỗi thực thi (trace) sẽ trông như thế nào? Hãy cùng nhau kiểm tra:

```sh
prompt> ./process-run.py -l 3:0 -L 5 -c
Time    PID: 0       CPU       IOs
  1         RUN:io             1
  2        BLOCKED                           1
  3        BLOCKED                           1
  4        BLOCKED                           1
  5        BLOCKED                           1
  6        BLOCKED                           1
  7*   RUN:io_done             1
  8         RUN:io             1
  9        BLOCKED                           1
 10        BLOCKED                           1
 11        BLOCKED                           1
 12        BLOCKED                           1
 13        BLOCKED                           1
 14*   RUN:io_done             1
 15         RUN:io             1
 16        BLOCKED                           1
 17        BLOCKED                           1
 18        BLOCKED                           1
 19        BLOCKED                           1
 20        BLOCKED                           1
 21*   RUN:io_done             1
```

Như bạn thấy, chương trình đã phát ra ba yêu cầu I/O. Mỗi khi phát ra một yêu cầu I/O, tiến trình sẽ chuyển trạng thái BLOCKED (bị chặn), và trong lúc thiết bị đang bận xử lý yêu cầu I/O đó thì CPU rơi vào trạng thái nhàn rỗi (idle).

Để xử lý ngắt hoàn tất I/O, một thao tác CPU nữa sẽ diễn ra (io_done). Lưu ý rằng việc dùng một lệnh CPU duy nhất để xử lý việc khởi tạo và hoàn tất I/O không hoàn toàn giống với thực tế, mà nó chỉ được mô hình hóa tại đây nhằm mục đích đơn giản hóa bài toán.

Hãy in ra một vài số liệu thống kê (chạy lại cùng câu lệnh như trên nhưng thêm cờ `-p`) để quan sát bức tranh tổng thể: 

```sh
Stats: Total Time 21
Stats: CPU Busy 6 (28.57%)
Stats: IO Busy  15 (71.43%)

# Thống kê: Tổng thời gian 21
# Thống kê: CPU bận 6 (28.57%)
# Thống kê: I/O bận 15 (71.43%)
```

Như bạn thấy, chuỗi thực thi mất tổng cộng 21 nhịp đồng hồ (clock ticks) để hoàn thành, nhưng CPU chỉ bận chưa tới 30% tổng thời gian. Ngược lại, thiết bị I/O lại bận rộn khá nhiều (71,43%). Nhìn chung, chúng ta luôn mong muốn giữ cho tất cả thiết bị đều luôn bận rộn, vì điều đó đồng nghĩa với việc tài nguyên hệ thống luôn được tận dụng hiệu quả nhất.

Dưới đây là một số cờ quan trọng khác:
```sh
  -s SEED, --seed=SEED  Giá trị hạt giống ngẫu nhiên (random seed)
    Cờ này cung cấp cách tạo ngẫu nhiên nhiều bài toán (jobs) khác nhau

  -L IO_LENGTH, --iolength=IO_LENGTH
    Cờ này quyết định thời gian để một thao tác I/O hoàn thành (mặc định là 5 ticks)

  -S PROCESS_SWITCH_BEHAVIOR, --switch=PROCESS_SWITCH_BEHAVIOR
                        Khi chuyển đổi giữa hai tiến trình: SWITCH_ON_IO, SWITCH_ON_END
    Cờ này quyết định thời điểm hệ thống chuyển sang chạy một tiến trình khác:
    - SWITCH_ON_IO, hệ thống sẽ chuyển ngữ cảnh sang tiến trình khác ngay khi tiến trình hiện tại phát yêu cầu I/O
    - SWITCH_ON_END, hệ thống chỉ chuyển sang tiến trình khác khi tiến trình hiện tại đã thực thi xong (done)

  -I IO_DONE_BEHAVIOR, --iodone=IO_DONE_BEHAVIOR
                        Hành vi sẽ xử lý khi I/O kết thúc: IO_RUN_LATER, IO_RUN_IMMEDIATE
    Cờ này quyết định thời điểm một tiến trình được chạy lại sau khi hoàn tất I/O:
    - IO_RUN_IMMEDIATE: Chuyển sang chạy lại tiến trình này ngay lập tức
    - IO_RUN_LATER: Chuyển sang chạy tiến trình này theo thứ tự tự nhiên
      (Ví dụ: Phụ thuộc vào hành vi chuyển đổi tiến trình đang thiết lập)
```

Bây giờ, hãy bắt đầu trả lời các câu hỏi ở cuối chương sách để tìm hiểu sâu hơn nhé.




