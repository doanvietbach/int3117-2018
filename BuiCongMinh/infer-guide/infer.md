# Infer

Mục lục:
- [Installation](#installation)
- [Infer Workflow](#infer-workflow)
- [Phân tích các ứng dụng hoặc dự án](#phân-tích-các-ứng-dụng-hoặc-dự-án)

## Installation

- Với MacOS, có thể cài đặt thông qua [Homebrew](https://brew.sh/)
```
brew install infer
```
- Với Linux có thể cài đặt sử dụng file [binary](https://github.com/facebook/infer/releases/tag/v0.15.0)

## Infer Workflow

Việc sử dụng infer được chia làm 2 giai đoạn:
- Giai đoạn "capture"
- Giai đoạn phân tích

### Giai đoạn captue
Các câu lệnh đã được dịch được Infer sử dụng để dịch (translate) file để phân tích bằng ngôn ngữ trung gian của Infer

Việc dịch này (translation) gần giống với dịch chương trình (compilation), Infer sử dụng thông tin của quá trình dịch(compilation) để sử dụng cho quá trình dịch(translation) của chính nó. Vậy nên khi compile file, ta sử dụng lệnh dịch của Infer: `infer run -- javac File.java` hay `infer run -- clang -c file.c`. Khi đó, file vẫn được dịch(compile) và cũng được dịch(translate) bởi Infer để  có thể phân tích ở giai đoạn thứ 2.

Infer lưu những file trung gian mặc định trong thử mục mà lệnh infer được gọi, thư mục này được gọi là `infer-out/`, ta có thể thay đổi thử mục tùy ý sử dụng `-o`, ví dụ như:
```
infer run -o /tmp/out -- javac Test.java
```

Ta có thể chỉ thực hiện giai đoạn "capture" với việc chỉ sử dụng lệnh `capture` thay vì `run`:
```
infer capture -- javac Test.java
```

### Giai đoạn phân tích
Trong giai đoạn này, các file trong `infer-out/` được phân tích bởi Infer. Infer phân thích từng hàm(function and method) riêng biệt. Nếu như Infer gặp pải lỗi khi phân tích một hàm (function or method), nó sẽ dừng quá trình kiểm tra với hàm đó, nhưng vẫn tiếp tục phân tích các hàm khác. Vậy nên, một cách sử dụng Infer đó là chạy Infer trên mã nguồn, sửa các lỗi được sinh ra, và chạy lặp lại để tìm ra lỗi có thể xuất hiện thêm hoặc để kiểm tra các lỗi đã được sửa đầy đủ.

Lỗi sẽ được hiển thị ở trên màn hình output của terminal và cũng được ghi ra file `infer-out/bugs.txt`. Các bugs được lọc và chỉ các bugs có khả năng cao nhất mới được báo cáo. Ở trong thư mục kết quả (`infer-out/`), file `report.csv` lưu tất cả các lỗi, cảnh bảo và thông tin được báo bởi Infer dưới định dạng csv.

### Các cách làm việc với Infer
Mặc định, khi được chạy, Infer sẽ xóa hết các kết quả trong `infer-out/` nếu như chúng tồn tại. Đây được gọi là phương pháp(workflow) cơ bản của Infer, khi mà toàn bộ dự án (project) được phân tích lại từ đầu. Truyền vào tùy chọn `--reactive` (hay `-r`) có tách dụng làm Infer không xóa đi `infer-out/`, đây là phương pháp sử dụng riêng biệt (differential workflow) của Infer.

Có vài trường hợp ngoại lệ với Infer. Ví dụ như ta có thể chỉ thực hiện một trong các giai đoạn ở trên, sử dụng `infer run -- javac Hello.java` sẽ ngang với thực hiện 2 lệnh:
```
infer capture -- javac Hello.java
infer analyze
```
Lệnh thứ 2 sẽ không xóa đi các tệp trong `infer-out/` vì các tệp cần cho giai đoạn phân tích nằm ở đây.

#### Phương pháp mặc định(Global workflow or default workflow)
Phương pháp mặc định hay toàn cục phù hợp nhất với việc sử dụng Infer trên tất cả các tệp của một dự án(project), ví dụ như với dự án sử dựng Gradle(Gradle based project) mà cần sử dụng lệnh `gradle build`:
```
infer run -- gradle build
```

Thông thường, lệnh `infer run <your build command here>` được sử dụng khi lệnh build được sử dụng để dịch mã nguồn (compile the source code).

#### Phương pháp riêng biệt(Differential workflow)
Trong các dự án phần mền ví dự như ứng dụng di động, hệ thống xây dựng tăng dần(incremental build system) được áp dụng, mã nguồn sẽ trải qua hàng loạt các thay đổi. Các dự án loại này sẽ phù hợp hơn với việc phân tích các thay đổi gần nhất của dự án chứ không phân tích toàn bộ dự án mỗi lần có thay đổi. Sử dụng *reactive mode* giúp ta thực hiện việc này.

Đầu tiên Infer cần được chạy trên một bản "sạch"(clean version) của dự án, để có thể thu được các lệnh dịch(compilation commands) tại giai đoạn capture.

Ví dụ như với một dự án sử dụng gradle:
```
gradle clean
infer capture -- gradle build
```

Sau đó, nếu như ta cần thay đổi các tệp trong dự án, ví dự như cần thay đổi theo một report của Infer, ta có thể làm sạch(clean) và phân tích lại toàn bộ dự án(như phương pháp nêu ở phần trên), hoặc ta có thể sử dụng Infer để phân tích tách dụng của các thay đổi gần nhất. Lựa chọn thứ 2 là tối ưu hơn vì chỉ một phần của dự án được phân tích.
```
edit some/File.java
# make some changes to some/File.java
infer run --reactive -- gradle build
```

Ta có thể sự dụng `--continue` để kết hợp nhiều thay đổi để phân tích. Ví dụ như:
```
edit some/File1.java
# make some changes to some/File1.java
infer run --reactive -- gradle build
edit some/File2.java
# make some changes to some/File2.java
infer run --reactive --continue -- gradle build
```

Sau lần gọi đầu tiên, Infer sẽ phân tích kết quả của lần thay đổi đầu tiên. Sau lần gọi thứ 2, nếu không sử dụng `--continue`, thì chỉ kết quả của lần thay đổi thứ 2 được phân tích.

Cuối cùng, ta luôn có thể phân tích các thay đổi gần nhất một cách riêng biệt:
```
infer run --reactive --continue -- analyze
```

### Infer reports
Ta có thể lấy được nhiều thông tin về báo cáo mà Infer sinh ra bằng cách sử dụng `inferTraceBugs` trong cùng một thư mục. Ví dụ như:
```
infer run -- gradle build
inferTraceBugs
```

Công cụ này cho phép ta quan sát được các nguyên nhân(error traces) dẫn đến lỗi(bug) được báo cáo bởi Infer, công cụ này rất hữu dụng trong trường hợp cần tìm chính xác các nguyên nhân của từng lỗi. Sử dụng `inferTraceBugs --help` để có thêm thông tin.

## Phân tích các ứng dụng hoặc dự án
Để phân tích các tệp với Infer, ta cần sử dụng chương trình dịch `javac` và `clang`. Ta cũng có thể sử dụng `gcc`, tuy nhiên, thực chất Infer sẽ sử dụng `clang` để dịch mã. Vậy nên Infer sẽ có thể không làm việc tốt với các mã không dịch được bằng `clang`

Thêm nữa, ta có thể sử dụng Infer với nhiều hệ thống build(build system). Cần chú ý rằng ta có thể chạy Infer nhanh hơn bằng cách chạy các lệnh dịch(compilation command) song song, ví dụ như `infer run -- make -j8`. Cần nhớ rằng nếu ta muốn phân tích một dự án, ta cần làm sạch(`clean`) trước khi tất cả các tệp được dịch và Infer phân tích các tệp.

Dưới đây sẽ là các ví dụ của các hệ thống(build system) được hỗ trợ bởi Infer.

### ant
```
infer run -- ant
```

### Buck
```
infer run -- buck <buck target>
```

Khi chạy lệnh trên, các cảnh bảo bởi Infer 