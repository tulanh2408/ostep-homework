
# Tổng quan

Chương trình `scheduler.py` cho phép bạn quan sát cách các bộ lập lịch (scheduler) khác nhau hoạt động dựa trên các chỉ số lập lịch như thời gian phản hồi (response time), thời gian hoàn thành (turnaround time), và tổng thời gian chờ (wait time). Ba bộ lập lịch được "cài đặt" sẵn gồm: FIFO, SJF, và RR.

Có hai bước để chạy chương trình.

Đầu tiên, chạy mà không có cờ `-c`: thao tác này sẽ hiển thị bài toán bạn cần giải mà không hiển thị kết quả. Ví dụ, nếu bạn muốn tính toán thời gian phản hồi, thời gian hoàn thành và thời gian chờ cho ba tác vụ (job) bằng chính sách FIFO, hãy chạy lệnh sau:

```sh
prompt> ./scheduler.py -p FIFO -j 3 -s 100
```

Nếu lệnh trên không hoạt động, hãy thử lệnh này:
```sh
prompt> python ./scheduler.py -p FIFO -j 3 -s 100
```

Lệnh này chỉ định chính sách FIFO với ba tác vụ, và quan trọng là với một giá trị hạt giống ngẫu nhiên (random seed) cụ thể là 100. Nếu muốn xem lời giải chính xác cho bài toán này, bạn phải chỉ định lại đúng giá trị seed ngẫu nhiên đó. Hãy chạy thử và xem điều gì sẽ xảy ra. Đây là những gì bạn sẽ thấy:

```sh
prompt> ./scheduler.py -p FIFO -j 3 -s 100
ARG policy FIFO
ARG jobs 3
ARG maxlen 10
ARG seed 100

Here is the job list, with the run time of each job: 
# [Dưới đây là danh sách các tác vụ, cùng thời gian chạy của từng tác vụ:]
  Job 0 (length = 1)
  Job 1 (length = 4)
  Job 2 (length = 7)
```

Nó tính toán thời gian hoàn thành (turnaround time), thời gian phản hồi (response time) và thời gian chờ (wait time) cho từng tác vụ. Khi hoàn tất, hãy chạy lại chương trình này với các đối số tương tự, nhưng kèm thêm cờ `-c` để chương trình cung cấp đáp án cho bạn. Bạn có thể sử dụng cờ `-s <một_số_nào_đó>` hoặc danh sách tác vụ tự chỉ định (ví dụ: `-l 10,15,20`) để tự tạo ra các bài toán khác nhau.

Như bạn có thể thấy từ ví dụ này, ba tác vụ được tạo ra gồm: Job 0 có thời lượng bằng 1, Job 1 có thời lượng bằng 4, và Job 2 có thời lượng bằng 7. Theo như chương trình thông báo, giờ bạn có thể dựa vào thông tin này để tính toán một vài số liệu thống kê và kiểm tra xem mình đã nắm vững các khái niệm cơ bản hay chưa.

Sau khi tính xong, bạn có thể sử dụng chính chương trình này để "giải" bài toán và kiểm tra xem kết quả của mình có chính xác không. Để làm điều đó, hãy dùng cờ `-c`. Đầu ra thu được như sau:

```sh
prompt> ./scheduler.py -p FIFO -j 3 -s 100 -c
ARG policy FIFO
ARG jobs 3
ARG maxlen 10
ARG seed 100

Here is the job list, with the run time of each job: 
# [Dưới đây là danh sách các tác vụ, cùng thời gian chạy của từng tác vụ:]
  Job 0 (length = 1)
  Job 1 (length = 4)
  Job 2 (length = 7)

** Solutions **
# [** Lời giải **]

Execution trace:
# [Vết thực thi:]
  [time   0] Run job 0 for 1.00 secs (DONE)
  [time   1] Run job 1 for 4.00 secs (DONE)
  [time   5] Run job 2 for 7.00 secs (DONE)

Final statistics:
# [Thống kê cuối cùng:]
  Job   0 -- Response: 0.00  Turnaround 1.00  Wait 0.00
  Job   1 -- Response: 1.00  Turnaround 5.00  Wait 1.00
  Job   2 -- Response: 5.00  Turnaround 12.00  Wait 5.00

  Average -- Response: 2.00  Turnaround 6.00  Wait 2.00
# [Trung bình -- Phản hồi: 2.00  Hoàn thành: 6.00  Chờ: 2.00]
```

Như bạn thấy từ kết quả, cờ `-c` cho thấy quá trình thực thi. Job 0 chạy đầu tiên trong 1 giây, Job 1 chạy tiếp theo trong 4 giây, và sau đó Job 2 chạy trong 7 giây. Không quá phức tạp; dù sao thì đây cũng là chính sách FIFO! Vết thực thi (execution trace) đã phản ánh rõ các kết quả này.

Các số liệu thống kê cuối cùng cũng rất hữu ích: chúng tính toán "thời gian phản hồi" (response time - thời gian một tác vụ phải chờ kể từ lúc đến cho đến khi được chạy lần đầu tiên), "thời gian hoàn thành" (turnaround time - khoảng thời gian từ lúc tác vụ đến cho tới khi hoàn tất), và tổng "thời gian chờ" (wait time - toàn bộ thời gian tác vụ ở trạng thái sẵn sàng (READY) nhưng không được chạy). Các chỉ số thống kê được hiển thị chi tiết cho từng tác vụ và sau đó là giá trị trung bình trên tất cả các tác vụ. Đương nhiên, bạn nên tự tính toán tất cả những con số này trước khi chạy chương trình với cờ `-c`!

Nếu bạn muốn thử cùng dạng bài toán này nhưng với các dữ liệu đầu vào khác, hãy thử thay đổi số lượng tác vụ hoặc giá trị hạt giống ngẫu nhiên (seed), hoặc cả hai. Việc thay đổi seed ngẫu nhiên về cơ bản cho phép bạn tự tạo ra vô số bài toán khác nhau, và cờ `-c` sẽ giúp bạn đối chiếu kết quả của mình. Hãy tiếp tục luyện tập cho đến khi bạn cảm thấy mình thực sự nắm vững các khái niệm.

Một cờ hữu ích khác là `-l` (chữ L viết thường), cho phép bạn chỉ định chính xác các tác vụ mà bạn muốn quan sát quá trình bộ lập lịch hoạt động. Ví dụ, nếu bạn muốn biết thuật toán SJF sẽ hoạt động như thế nào với ba tác vụ có thời lượng lần lượt là 5, 10 và 15, bạn có thể chạy:

```sh
prompt> ./scheduler.py -p SJF -l 5,10,15
ARG policy SJF
ARG jlist 5,10,15

Here is the job list, with the run time of each job: 
# [Dưới đây là danh sách các tác vụ, cùng thời gian chạy của từng tác vụ:]
  Job 0 (length = 5.0)
  Job 1 (length = 10.0)
  Job 2 (length = 15.0)
...
```

Sau đó, bạn có thể sử dụng cờ `-c` để xem lời giải. Lưu ý rằng khi bạn chỉ định cụ thể các tác vụ, bạn không cần phải xác định seed ngẫu nhiên hay số lượng tác vụ: thời lượng của các tác vụ được lấy trực tiếp từ danh sách phân tách bằng dấu phẩy mà bạn đã cung cấp.

Tất nhiên, nhiều tình huống thú vị hơn sẽ xuất hiện khi bạn sử dụng các bộ điều phối SJF (Shortest-Job First - tác vụ ngắn nhất trước) hay RR (Round Robin - quay vòng). Hãy thử nghiệm và tự mình quan sát!

Bạn luôn có thể chạy lệnh:

```sh
prompt> ./scheduler.py -h
```

để xem danh sách đầy đủ các cờ (flags) và tùy chọn (bao gồm cả các tùy chọn như thiết lập lượng tử thời gian - time quantum cho bộ lập lịch RR).