
# Tổng quan

Có hai trình mô phỏng được đề cập trong chương này. Trình mô phỏng 
đầu tiên, `fork.py`, là một công cụ đơn giản để minh họa cấu trúc 
của cây tiến trình (process tree) khi các tiến trình được tạo ra và bị hủy bỏ. 
Đọc thêm về công cụ này 
[tại đây](README-fork.md).

Trình mô phỏng thứ hai là chương trình `generator.py`, có chức năng 
tạo ra các chương trình C thực tế sử dụng `fork()`, `wait()`, và `exit()` 
nhằm minh họa cách thức `fork` hoạt động trong các chương trình đang chạy. 
Đọc thêm về chương trình này [tại đây](README-generator.md).