# ARM-Tutorial

## ARM Registers

Số lượng thanh ghi phụ thuộc vào phiên bản ARM. Theo Sổ tay Tham khảo ARM, có 30 thanh ghi 32-bit mục đích chung , ngoại trừ các bộ xử lý dựa trên ARMv6-M và ARMv7-M. 16 thanh ghi đầu tiên có thể truy cập ở chế độ người dùng, các thanh ghi bổ sung có sẵn trong chế độ thực thi phần mềm đặc quyền (ngoại trừ ARMv6-M và ARMv7-M). Trong loạt bài hướng dẫn này, chúng ta sẽ làm việc với các thanh ghi có thể truy cập ở bất kỳ chế độ đặc quyền nào: r0-15. 16 thanh ghi này có thể được chia thành hai nhóm: thanh ghi mục đích chung và thanh ghi mục đích đặc biệt.

<img width="591" height="831" alt="ảnh" src="https://github.com/user-attachments/assets/54fdbf69-7223-42f8-b5d3-2068cf1ef044" />

Bảng sau đây chỉ là cái nhìn tổng quan về mối liên hệ giữa các thanh ghi ARM và  các thanh ghi trong bộ xử lý Intel.

<img width="584" height="446" alt="ảnh" src="https://github.com/user-attachments/assets/31d66079-e975-4865-8944-2185cdf21af1" />

> R0-R12 : có thể được sử dụng trong các thao tác chung để lưu trữ giá trị tạm thời, con trỏ (vị trí bộ nhớ), v.v. Ví dụ, R0 có thể được gọi là thanh ghi tích lũy trong các phép toán số học hoặc để lưu trữ kết quả của một hàm đã được gọi trước đó. R7 trở nên hữu ích khi làm việc với các lệnh gọi hệ thống vì nó lưu trữ số hiệu lệnh gọi hệ thống và R11 giúp chúng ta theo dõi các ranh giới trên ngăn xếp, đóng vai trò là con trỏ khung (sẽ được đề cập sau). Hơn nữa, quy ước gọi hàm trên ARM quy định rằng bốn đối số đầu tiên của một hàm được lưu trữ trong các thanh ghi r0-r3.

> R13: SP (Con trỏ Ngăn xếp). Con trỏ Ngăn xếp trỏ đến đỉnh của ngăn xếp. Ngăn xếp là một vùng bộ nhớ được sử dụng để lưu trữ dữ liệu cụ thể cho hàm, vùng này sẽ được thu hồi khi hàm trả về. Do đó, con trỏ ngăn xếp được sử dụng để cấp phát kHông gian trên ngăn xếp, bằng cách trừ giá trị (tính bằng byte) mà chúng ta muốn cấp phát khỏi con trỏ ngăn xếp. Nói cách khác, nếu chúng ta muốn cấp phát một giá trị 32 bit, chúng ta trừ 4 khỏi con trỏ ngăn xếp.

> R14: LR (Thanh ghi Liên kết). Khi một lệnh gọi hàm được thực hiện, Thanh ghi Liên kết sẽ được cập nhật với một địa chỉ bộ nhớ tham chiếu đến lệnh tiếp theo mà hàm được khởi tạo. Thao tác này cho phép chương trình quay lại hàm "cha" đã khởi tạo lệnh gọi hàm "con" sau khi hàm "con" kết thúc.

> R15: PC (Bộ đếm Chương trình). Bộ đếm Chương trình được tự động tăng theo kích thước của lệnh được thực thi. Kích thước này luôn là 4 byte ở trạng thái ARM và 2 byte ở chế độ THUMB. Khi một lệnh rẽ nhánh đang được thực thi, PC giữ địa chỉ đích. Trong quá trình thực thi, PC lưu trữ địa chỉ của lệnh hiện tại cộng với 8 (hai lệnh ARM) ở trạng thái ARM, và lệnh hiện tại cộng với 4 (hai lệnh Thumb) ở trạng thái Thumb(v1). Điều này khác với x86, nơi PC luôn trỏ đến lệnh tiếp theo cần thực thi.

## Current Program Status Register

Khi bạn gỡ lỗi nhị phân ARM bằng gdb, bạn sẽ thấy thứ gọi là Cờ:

<img width="836" height="123" alt="ảnh" src="https://github.com/user-attachments/assets/bed6e6b1-aa30-4167-80e8-3efebc46f158" />

Thanh ghi $cpsr hiển thị giá trị của Thanh ghi Trạng thái Chương trình Hiện tại (CPSR) và bên dưới đó, bạn có thể thấy các Cờ Thumb, Fast, Interrupt, Overflow, Carry, Zero và Negative. Các cờ này đại diện cho một số bit nhất định trong thanh ghi CPSR và được thiết lập theo giá trị của CPSR, đồng thời được in đậm khi được kích hoạt. Các bit N, Z, C và V giống hệt với các bit SF, ZF, CF và OF trong thanh ghi EFLAG trên x86. Các bit này được sử dụng để hỗ trợ thực thi có điều kiện trong các câu lệnh điều kiện và vòng lặp ở cấp độ hợp ngữ. 

<img width="778" height="283" alt="ảnh" src="https://github.com/user-attachments/assets/f1418365-2a4a-4ae7-892c-314a8045ec18" />

Hình trên cho thấy sơ đồ bố trí của một thanh ghi 32 bit (CPSR), trong đó phía bên trái (<-) chứa các bit có trọng số cao nhất và phía bên phải (->) chứa các bit có trọng số thấp nhất. Mỗi ô (trừ phần GE và M cùng với các ô trống) có kích thước một bit. Các phần một bit này định nghĩa các thuộc tính khác nhau của trạng thái hiện tại của chương trình.

<img width="833" height="653" alt="ảnh" src="https://github.com/user-attachments/assets/200111d3-e1bd-441c-98a3-c8eea0f11d11" />

## ARM & Thumb

Bộ xử lý ARM có hai trạng thái chính mà chúng có thể hoạt động, ARM và Thumb. Sự khác biệt chính giữa hai trạng thái này là tập lệnh, trong đó các lệnh ở trạng thái ARM luôn là 32-bit, và các lệnh ở trạng thái Thumb là 16-bit (nhưng có thể là 32-bit). Việc biết khi nào và cách sử dụng Thumb đặc biệt quan trọng cho mục đích phát triển khai thác ARM. Khi viết shellcode ARM, chúng ta cần loại bỏ các byte NULL và việc sử dụng các lệnh Thumb 16-bit thay vì các lệnh ARM 32-bit sẽ giảm khả năng xảy ra lỗi.

Các lệnh ARM thường được theo sau bởi một hoặc hai toán hạng và thường sử dụng mẫu sau:

```asm
MNEMONIC{S}{condition} {Rd}, Operand1, Operand2
```

> MNEMONIC     - Tên viết tắt
> 
> {S}          - Một hậu tố tùy chọn. Nếu S được chỉ định, các cờ điều kiện sẽ được cập nhật dựa trên kết quả của phép toán.
>
> {condition}  - Điều kiện cần phải đáp ứng để lệnh được thực thi
> 
> {Rd}         - Thanh ghi (đích) để lưu trữ kết quả của lệnh
> 
> Operand1     - Toán hạng đầu tiên. Có thể là một thanh ghi hoặc một giá trị tức thời
> 
> Operand2     - Toán hạng thứ hai (linh hoạt). Có thể là một giá trị tức thời (số) hoặc một thanh ghi có tùy chọn dịch chuyển.

```asm
ADD   R0, R1, R2         - Thêm nội dung của R1 (Toán hạng 1) và R2 (Toán hạng 2 dưới dạng thanh ghi) và lưu trữ kết quả vào R0 (Rd)
ADD   R0, R1, #2         - Thêm nội dung của R1 (Toán hạng 1) và giá trị 2 (Toán hạng 2 dưới dạng giá trị tức thời) và lưu trữ kết quả vào R0 (Rd)
MOVLE R0, #5             - Di chuyển số 5 (Toán hạng 2, vì trình biên dịch coi nó là MOVLE R0, R0, #5) đến R0 (Rd) CHỈ khi điều kiện LE (Nhỏ hơn hoặc Bằng) được đáp ứng
MOV   R0, R1, LSL #1     - Di chuyển nội dung của R1 (Toán hạng 2 dưới dạng thanh ghi với phép dịch logic sang trái) sang trái một bit đến R0 (Rd). Vì vậy, nếu R1 có giá trị 2, nó sẽ được dịch trái một bit và trở thành 4. 4 sau đó được chuyển đến R0.
```

<img width="827" height="578" alt="ảnh" src="https://github.com/user-attachments/assets/4005213d-558e-42f8-9a28-84b4f6e351b7" />

## Memory Instructions: Load and Store

ARM sử dụng mô hình LOAD-STORE trữ để truy cập bộ nhớ, nghĩa là chỉ các lệnh LOAD/STORE (LDR và ​​STR) mới có thể truy cập bộ nhớ.

<img width="252" height="172" alt="ảnh" src="https://github.com/user-attachments/assets/a09b35ce-b149-4c31-9722-7a6e1c8ca49f" />

<img width="548" height="191" alt="ảnh" src="https://github.com/user-attachments/assets/3d0dcae1-1895-4aa6-86ec-0c81e0c6aac5" />

<img width="833" height="451" alt="ảnh" src="https://github.com/user-attachments/assets/6de4186c-409e-44ce-8824-c685eba4ac74" />

<img width="792" height="680" alt="ảnh" src="https://github.com/user-attachments/assets/edc60e6d-35f5-41d3-9f89-fe1a6cf71f98" />

## Load/Store Multiple

LDM và STM có các biến thể. Loại biến thể được xác định bởi hậu tố của lệnh. Các hậu tố được sử dụng trong ví dụ là: -IA (tăng sau), -IB (tăng trước), -DA (giảm sau), -DB (giảm trước)

LDM giống với LDMIA, nghĩa là địa chỉ của phần tử tiếp theo được tải sẽ được tăng lên sau mỗi lần tải. Theo cách này, chúng ta có được một lần tải dữ liệu tuần tự (chuyển tiếp) từ địa chỉ bộ nhớ được chỉ định bởi toán hạng đầu tiên (thanh ghi lưu trữ địa chỉ nguồn).

```asm
ldmia r0, {r4-r6} /* words[3] -> r4 = 0x03, words[4] -> r5 = 0x04; words[5] -> r6 = 0x05; */ 
stmia r1, {r4-r6} /* r4 -> array_buff[0] = 0x03; r5 -> array_buff[1] = 0x04; r6 -> array_buff[2] = 0x05 */
```

Sau khi thực hiện hai lệnh trên, các thanh ghi R4-R6 và các địa chỉ bộ nhớ 0x000100D0, 0x000100D4 và 0x000100D8 chứa các giá trị 0x3, 0x4 và 0x5.

Lệnh LDMIB trước tiên tăng địa chỉ nguồn thêm 4 byte (giá trị một từ) rồi thực hiện lần tải đầu tiên. Theo cách này, chúng ta vẫn có một lần tải dữ liệu tuần tự (chuyển tiếp), nhưng phần tử đầu tiên cách địa chỉ nguồn 4 byte.

```asm
ldmib r0, {r4-r6}            /* words[4] -> r4 = 0x04; words[5] -> r5 = 0x05; words[6] -> r6 = 0x06 */
stmib r1, {r4-r6}            /* r4 -> array_buff[1] = 0x04; r5 -> array_buff[2] = 0x05; r6 -> array_buff[3] = 0x06 */
```
Sau khi thực hiện hai lệnh trên, các thanh ghi R4-R6 và các địa chỉ bộ nhớ 0x100D4, 0x100D8 và 0x100DC chứa các giá trị 0x4, 0x5 và 0x6.

Khi chúng ta sử dụng lệnh LDMDA, mọi thứ bắt đầu hoạt động ngược lại. R0 trỏ tới words[3]. Khi quá trình tải bắt đầu, chúng ta di chuyển ngược lại và tải các words[3], words[2] và words[1] vào R6, R5, R4.

Tải nhiều, giảm dần sau:

```asm
ldmda r0, {r4-r6} /* words[3] -> r6 = 0x03; words[2] -> r5 = 0x02; words[1] -> r4 = 0x01 */
```

Thanh ghi R4, R5, R6:
```asm
gef> info register r4 r5 r6
r4     0x1    1
r5     0x2    2
r6     0x3    3
```

Tải nhiều, giảm dần trước:

```asm
ldmdb r0, {r4-r6} /* words[2] -> r6 = 0x02; words[1] -> r5 = 0x01; words[0] -> r4 = 0x00 */
```

Thanh ghi R4, R5, R6:
```asm
gef> info register r4 r5 r6
r4 0x0 0
r5 0x1 1
r6 0x2 2
```

## PUSH and POP

Khi PUSH vào STACK:
* Đầu tiên, địa chỉ trong SP sẽ GIẢM đi 4.
* Thứ hai, thông tin được lưu trữ tại địa chỉ mới được SP trỏ tới.

Khi POP ra khỏi STACK:
* Giá trị tại địa chỉ SP hiện tại được tải vào một thanh ghi nhất định,
* Địa chỉ trong SP được TĂNG thêm 4.

``` asm
PUSH = STMDB sp!, reglist
POP = LDMIA sp! reglist
```

## Conditional Execution
