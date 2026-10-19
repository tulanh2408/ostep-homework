[English](README.md) | Tiếng Việt

---
# Homework (Bài tập)

Mỗi chương đều có một số câu hỏi ở phần cuối; Chúng ta gọi nó là "bài tập về nhà" (homeworks), vì bạn nên làm "việc" (work) ở "nhà" (home). Hợp lý chứ? Nó là một trong những điểm đổi mới của cuốn sách này.

Các bài tập này nhằm củng cố kiến thức mà bạn đã học trong mỗi chương. Rất nhiều bài tập xoay quanh việc vận hành một trình mô phỏng (simulator) tái hiện lại một khía cạnh cụ thể của hệ điều hành. Ví dụ, một trình mô phỏng lập lịch đĩa (disk scheduling) sẽ giúp bạn hiểu rõ cách thức hoạt động của các thuật toán lập lịch đĩa khác nhau. Một số bài tập khác chỉ là các bài code ngắn, giúp các bạn trực tiếp khám phá cách thức vận hành của các hệ thống thực tế.

Với các trình mô phỏng, ý tưởng cốt lõi rất đơn giản: Mỗi công cụ bên dưới cho phép bạn tự tạo đề bài và nhận lời giải cho vô số bài toán ngẫu nhiên. Bạn có thể đổi các giá trị seed ngẫu nhiên (random seeds) khác nhau để sinh ra các bài toán mới; gắn thêm cờ `-c` để hiển thị luôn đáp án cho bạn (dĩ nhiên là bạn đã thử tự mình tìm ra cách giải trước đó!).

Mỗi bài tập liệt kê dưới đây bao gồm một file README giải thích những việc cần làm. Trước đây, phần nội dung này từng được đưa trực tiếp vào trong các chương của sách, nhưng điều đó khiến cho cuốn sách trở nên quá dài và dày. Hiện tại, tất cả những gì còn lại trong sách chỉ còn các câu hỏi bạn muốn trả lời bằng trình mô phỏng; chi tiết về cách chạy mã nguồn và cách mã nguồn hoạt động đều nằm tất cả trong file README tương ứng theo từng chương.

Vì vậy, quy trình học của bạn sẽ là: đọc hết một chương, xem các câu hỏi ở cuối chương đó và tự tìm lời giải bằng cách làm bài tập về nhà. Một số câu hỏi yêu cầu chạy trình mô phỏng viết bằng Python (được cung cấp sẵn bên dưới). Một số câu khác yêu cầu bạn tự tay viết code (ví dụ như viết mã C để hoàn thành một tác vụ). Khi làm tới đây, đọc kỹ file README của bài tập đó là điều rất cần thiết.

Để sử dụng chúng, cách tốt nhất là clone kho bài tập này về máy. Ví dụ:
```sh
prompt> git clone https://github.com/remzi-arpacidusseau/ostep-homework/
prompt> cd file-disks
prompt> ./disk.py -h
```

# Introduction (Giới thiệu)

Chương | Việc cần làm
-------|-------------
[Introduction](http://www.cs.wisc.edu/~remzi/OSTEP/intro.pdf) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Chưa có bài tập

# Virtualization (Ảo hóa)

Chương | Việc cần làm
-------|-------------
[Abstraction: Processes](http://www.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf) | Chạy [process-run.py](cpu-intro)
[Process API](http://www.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf) | Chạy [fork.py](cpu-api) và viết thêm code
[Direct Execution](http://www.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf) | Tự viết code
[Scheduling Basics](http://www.cs.wisc.edu/~remzi/OSTEP/cpu-sched.pdf) | Chạy [scheduler.py](cpu-sched)
[MLFQ Scheduling](http://www.cs.wisc.edu/~remzi/OSTEP/cpu-sched-mlfq.pdf)	| Chạy [mlfq.py](cpu-sched-mlfq)
[Lottery Scheduling](http://www.cs.wisc.edu/~remzi/OSTEP/cpu-sched-lottery.pdf) | Chạy [lottery.py](cpu-sched-lottery)
[Multiprocessor Scheduling](http://www.cs.wisc.edu/~remzi/OSTEP/cpu-sched-multi.pdf) | Chạy [multi.py](cpu-sched-multi)
[Abstraction: Address Spaces](http://www.cs.wisc.edu/~remzi/OSTEP/vm-intro.pdf) | Tự viết code
[VM API](http://www.cs.wisc.edu/~remzi/OSTEP/vm-api.pdf) | Tự viết code
[Relocation](http://www.cs.wisc.edu/~remzi/OSTEP/vm-mechanism.pdf) | Chạy [relocation.py](vm-mechanism)
[Segmentation](http://www.cs.wisc.edu/~remzi/OSTEP/vm-segmentation.pdf) | Chạy [segmentation.py](vm-segmentation)
[Free Space](http://www.cs.wisc.edu/~remzi/OSTEP/vm-freespace.pdf) | Chạy [malloc.py](vm-freespace)
[Paging](http://www.cs.wisc.edu/~remzi/OSTEP/vm-paging.pdf) | Chạy [paging-linear-translate.py](vm-paging)
[TLBs](http://www.cs.wisc.edu/~remzi/OSTEP/vm-tlbs.pdf) | Tự viết code
[Multi-level Paging](http://www.cs.wisc.edu/~remzi/OSTEP/vm-smalltables.pdf) | Chạy [paging-multilevel-translate.py](vm-smalltables)
[Paging Mechanism](http://www.cs.wisc.edu/~remzi/OSTEP/vm-beyondphys.pdf) | Chạy [mem.c](vm-beyondphys)
[Paging Policy](http://www.cs.wisc.edu/~remzi/OSTEP/vm-beyondphys-policy.pdf) | Chạy [paging-policy.py](vm-beyondphys-policy)
[Complete VM](http://www.cs.wisc.edu/~remzi/OSTEP/vm-complete.pdf) | Chưa có bài tập

# Concurrency (Đồng thời)

Chương | Việc cần làm
-------|-------------
[Threads Intro](http://www.cs.wisc.edu/~remzi/OSTEP/threads-intro.pdf) | Chạy [x86.py](threads-intro)
[Thread API](http://www.cs.wisc.edu/~remzi/OSTEP/threads-api.pdf)	| Chạy [Mã nguồn C](threads-api)
[Locks](http://www.cs.wisc.edu/~remzi/OSTEP/threads-locks.pdf)	| Chạy [x86.py](threads-locks)
[Lock Usage](http://www.cs.wisc.edu/~remzi/OSTEP/threads-locks-usage.pdf) | Tự viết code
[Condition Variables](http://www.cs.wisc.edu/~remzi/OSTEP/threads-cv.pdf) | Chạy [Mã nguồn C](threads-cv)
[Semaphores](http://www.cs.wisc.edu/~remzi/OSTEP/threads-sema.pdf) | Đọc và viết [some code](threads-sema)
[Concurrency Bugs](http://www.cs.wisc.edu/~remzi/OSTEP/threads-bugs.pdf) | Chạy [Mã nguồn C](threads-bugs)
[Event-based Concurrency](http://www.cs.wisc.edu/~remzi/OSTEP/threads-events.pdf) | Tự viết code

# Persistence (Tính bền bỉ)

Chương | Việc cần làm
-------|-------------
[I/O Devices](http://www.cs.wisc.edu/~remzi/OSTEP/file-devices.pdf) | Chưa có bài tập
[Hard Disk Drives](http://www.cs.wisc.edu/~remzi/OSTEP/file-disks.pdf) | Chạy [disk.py](file-disks)
[RAID](http://www.cs.wisc.edu/~remzi/OSTEP/file-raid.pdf) | Chạy [raid.py](file-raid)
[FS Intro](http://www.cs.wisc.edu/~remzi/OSTEP/file-intro.pdf) | Tự viết code
[FS Implementation](http://www.cs.wisc.edu/~remzi/OSTEP/file-implementation.pdf) | Chạy [vsfs.py](file-implementation)
[Fast File System](http://www.cs.wisc.edu/~remzi/OSTEP/file-ffs.pdf) | Chạy [ffs.py](file-ffs)
[Crash Consistency and Journaling](http://www.cs.wisc.edu/~remzi/OSTEP/file-journaling.pdf) | Chạy [fsck.py](file-journaling)
[Log-Structured File Systems](http://www.cs.wisc.edu/~remzi/OSTEP/file-lfs.pdf) | Chạy [lfs.py](file-lfs)
[Solid-State Disk Drives](http://www.cs.wisc.edu/~remzi/OSTEP/file-ssd.pdf) | Chạy [ssd.py](file-ssd)
[Data Integrity](http://www.cs.wisc.edu/~remzi/OSTEP/file-integrity.pdf) | Chạy [checksum.py](file-integrity) và tự viết code
[Distributed Intro](http://www.cs.wisc.edu/~remzi/OSTEP/dist-intro.pdf) | Tự viết code
[NFS](http://www.cs.wisc.edu/~remzi/OSTEP/dist-nfs.pdf) | Viết mã phân tích
[AFS](http://www.cs.wisc.edu/~remzi/OSTEP/dist-afs.pdf) | Chạy [afs.py](dist-afs)
