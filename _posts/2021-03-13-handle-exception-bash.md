---
title: 'Handle exception in Bash'
date: 2021-03-13
permalink: /posts/2021/03/handle-exception-in-bash/
tags:
- bash
- linux
---

## Giới thiệu

Trong các ngôn ngữ lập trình bậc cao phổ biến như Java khi có exception xảy ra chúng ta thường có hai cách giải quyết đó là dừng chương trình bằng việc ném ra ngoại lệ sử dụng `throw/throws` keyword hoặc bắt ngoại lệ để chương trình có thể tiếp tục thực thi bằng cách sử dụng `try/catch` block.

## Môi trường

- Bash shell

## Kiến thức cơ bản 

- `Terminal` là môi trường cho phép nhận `input` (text) từ bàn phím..., gửi `input` (command) đến `shell` và nhận `output` trả về từ `shell`.
- `Shell` vừa là một trình thông dịch (`command line interpreter`) giúp bạn thực thi câu lệnh mà người dùng gửi đến thông qua `terminal/terminal-emulator`, vừa là một ngôn ngữ lập trình. Nếu như với Java, chúng ta sử dụng file với đuôi `.java` thì với shell, mà trong phạm vi bài viết này là `bash` (`Bourne-Again SHell`), chúng ta sử dụng file với đuôi `.sh` (stand for shell) hay còn được gọi là `bash script`.

- Mỗi lần bạn gõ câu lệnh trên terminal và nhấn phím `Enter`, `bash` sẽ thực thi câu lệnh đó và trả về một `exit status` hay còn được gọi là `return code`, là một số nguyên nằm trong khoảng [0,255]. Theo quy ước thì khi một câu lệnh thực thi thành công sẽ trả về `0 exit status`, còn nếu có lỗi xảy ra thì [`non-zero exit status`](https://tldp.org/LDP/abs/html/exitcodes.html) sẽ được trả về tỉ như `command not found` là `127`. Chúng ta có một biến đặc biệt lưu trữ `exit status` của câu lệnh đã được thực thi trước đó, và để kiểm tra bạn có thể làm như sau: 

  ```shell
  echo $?
  ```
  
  - Ex:
      ```shell
      ls logbasex
      echo $?
      ```
      
  - Output
      ```shell
      ls: cannot access 'logbasex': No such file or directory
      2  
      ```
- `File Descriptor` là một số nguyên dương đại diện cho một tập tin đang được mở trong một process. Một `terminal emulator` có 3 file descriptors mặc định đó là `stdin` ,`stdout`, `stderr`. 
    ```shell
        ls -la /proc/$$/fd
    ```
  Output
    ```shell
    total 0
    dr-x------ 2 logbasex logbasex  0 Thg 3  14 07:06 .
    dr-xr-xr-x 9 logbasex logbasex  0 Thg 3  14 07:06 ..
    lrwx------ 1 logbasex logbasex 64 Thg 3  14 07:06 0 -> /dev/pts/5
    lrwx------ 1 logbasex logbasex 64 Thg 3  14 07:06 1 -> /dev/pts/5
    lrwx------ 1 logbasex logbasex 64 Thg 3  14 07:06 2 -> /dev/pts/5
    lrwx------ 1 logbasex logbasex 64 Thg 3  14 08:35 255 -> /dev/pts/5
    ```
    - [stdin](https://en.wikipedia.org/wiki/File_descriptor): Standard Input (File descriptor 0)
    - [stdout](https://en.wikipedia.org/wiki/File_descriptor): Standard Output (File descriptor 1)
    - [stderr](https://en.wikipedia.org/wiki/File_descriptor): Standard Error (File descriptor 2)   
- `Pipeline` là một chuỗi của 2 hay nhiều câu lệnh được phân cách bằng kí tự `|` hoặc `|&` cho phép chúng ta truyền dữ liệu từ process này sang process kia mà [không cần lưu dữ liệu trực tiếp lên   ổ cứng](https://stackoverflow.com/questions/18053647/does-writing-data-to-stdout-in-linux-occupy-disk-space). Trong đó`|` pipes redirect `STDOUT` của process này sang `STDIN` của process kia còn `|&` redirect cả `STDOUT` và`STDERR` Ví dụ:
    ```shell
     ls $PWD | grep logbasex.sh
    ```
  Đây là một `pipeline` cơ bản có thể được hiểu như sau: Liệt kê các tập tin trong thư mục hiện tại rồi tìm kiếm những tập tin có tên là `logbasex.sh` và hiển thị kết quả ra màn hình, tức là `output` (`STDOUT`) của câu lệnh thứ nhất `ls $PWD` được sử dụng làm `input`(`STDIN`) của câu lệnh thứ hai `grep logbasex.sh`.

<iframe width="560" height="315"
src="http://www.youtube.com/embed/bKzonnwoR2I" 
frameborder="0" 
allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" 
allowfullscreen></iframe>

- **Ở đây mình chỉ giới thiệu qua các khái niệm cơ bản đủ để hiểu được nội dung bài viết. Mọi người chủ động tìm hiểu thêm nếu có hứng thú nhé.**
<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/logbasex.github.io/1024-file-descriptors-ought-to-be-enough-for-anybody.jpg"  alt=""/>
</p>

## Tạo bug

1. Để xử lý ngoại lệ thì đầu tiên phải có chương trình ném ra ngoại lệ trước đã. Mình tạo một file có tên `error.sh` với nội dung như sau:
    ```shell
    #!/bin/bash
     
    ls logbasex
    good morning
    love
    ```
2. Execute file  
    ```shell
    bash error.sh
    ```
 3. Output 
    ```shell
    ls: cannot access 'logbasex': No such file or directory
    error.sh: line 4: good: command not found
    error.sh: line 5: love: command not found
    ```

## Xử lý ngoại lệ    

### I. Trường hợp cơ bản
 
 Như đã thấy ở trên, cả 3 câu lệnh đều thực thi không thành công, chương trình kết thúc và chúng ta có 3 dòng báo lỗi hiển thị trên màn hình. Ok. Nhưng bây giờ nếu bạn muốn chương trình kết thúc ngay tại dòng lệnh đầu tiên mà thực thi không thành công thì làm thế nào? 

Thật may mắn là `bash` có hỗ trợ một `built-in command` gọi là [`set`](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html) với option `-e`.

> Exit immediately if a pipeline (see Pipelines), which may consist of a single simple command (see Simple Commands), a list (see Lists), or a compound command (see Compound Commands) returns a non-zero status. **The shell does not exit if** the command that fails is part of the command list immediately following a while or until keyword, part of the test in an if statement, part of any command executed in a `&&` or `||` list except the command following the final `&&` or `||`, **any command in a pipeline but the last**, or if the command’s return status is being inverted with !. If a compound command other than a subshell returns a non-zero status because a command failed while -e was being ignored, the shell does not exit. A trap on ERR, if set, is executed before the shell exits.
>
> This option applies to the shell environment and each subshell environment separately (see Command Execution Environment), and may cause subshells to exit before executing all the commands in the subshell. 
>
> If a compound command or shell function executes in a context where -e is being ignored, none of the commands executed within the compound command or function body will be affected by the -e setting, even if -e is set and a command returns a failure status. If a compound command or shell function sets -e while executing in a context where -e is ignored, that setting will not have any effect until the compound command or the command containing the function call completes.

1. Chúng ta tạo một script mới có tên `error-e.sh` với nội dung như sau
    ```shell
    #!/bin/bash
    
    set -e
    
    ls logbasex
    good morning
    love
    ```
2. Thực thi
    ```shell
    bash -e error.sh 
    
    OR
    
    bash error-e.sh
    ```
3. Output
    ```shell
    ls: cannot access 'logbasex': No such file or directory
    ```

### II. Ngoại lệ của `set -e` option (any command in a pipeline but the last)

1. Chúng ta tạo một script mới có tên `error-pipeline.sh` với nội dung như sau
    ```shell
    #!/bin/bash
    set -e
    
    logbasex | echo "hello"
    echo "good morning"
    ```
2. Thực thi
    ```shell
    bash error-pipeline.sh
    ```
3. Output
    ```shell
    hello
    error-pipeline.sh: line 4: logbasex: command not found
    good morning
    ```

Như bạn có thể thấy, ngoại lệ ném ra ở dòng thứ 4 nhưng câu lệnh ở dòng thứ 5 vẫn được thực thi là bởi vì câu lệnh `logbasex` không phải là câu lệnh cuối cùng trong một `pipelines`.

Và thật may mắn là câu lệnh `set` cung cấp cho chúng ta option `-o pipefail`

> -o pipefail
>
>If set, the return value of a pipeline is the value of the last (rightmost) command to exit with a non-zero status, or zero if all commands in the pipeline exit successfully. This option is **disabled by default**.
>
>Nếu sử dụng thì exit status trả về của một pipeline sẽ là exit status của câu lệnh ngoài cùng bên phải mà có exist status khác `0` hoặc là bằng `0` nếu tất cả câu lệnh trong một pipeline đều được thực thi thành công. 

Ví dụ với script dưới đây
```shell
set -o pipefail
logbasex | grep some-string /non/existent/file | sort
echo "${PIPESTATUS[0]} ${PIPESTATUS[1]} ${PIPESTATUS[2]}" $?
```
thì output sẽ là 
```shell
grep: /non/existent/file: No such file or directory
logbasex: command not found
127 2 0 2
```
Như bạn có thể thấy, exit status của pipeline trên chính là bằng `2`.

Và...

Việc bây giờ là sửa lại script và thực thi để thấy ngay điều kì diệu
```shell
#!/bin/bash

set -eo pipefail
logbasex | echo "hello"
echo "good morning"
```

Oops :(( Output nhìn có vẻ không đẹp lắm, sẽ thật tuyệt nếu bỏ được error log ra khỏi output. Và thật may mắn, `bash` hỗ trợ chúng ta redirect error ouput (`STDERR`) vào `hố đen vô cực` như sau
```shell
bash error-pipefail.sh 2> /dev/null
```
> Mình sẽ giải thích câu lệnh trên ở một bài viết khác. 
>

<p align="center">
    <img src="https://s3-ap-southeast-1.amazonaws.com/logbasex.github.io/cat-your-opinion.jpg" alt="">
</p>
### II. Bỏ qua ngoại lệ
Trước giờ chúng ta chỉ nói đến vấn đề ném ra ngoại lệ, nhưng bây giờ chúng ta muốn chương trình vẫn tiếp tục thực thi khi bắt gặp ngoại lệ thì làm thế nào? 

Thú thật thì chả biết làm thế nào ngoài sử dụng toán tử `OR` cả. Toán tử này trong `bash script` được biểu diễn dưới kí tự `||` (double pipe) và cách dùng cũng tương tự như trong Java, có hỗ trợ [short-circuit](https://www.geeksforgeeks.org/short-circuiting-in-java-with-examples).

1. Việc bây giờ là tạo một script mới với tên `error-suppression.sh` có nội dung như sau: 
    ```shell
    #!/bin/bash
    
    set -eo pipefail
    logbasex | echo "hello" || true
    echo "good morning"
    ```
2. Thực thi  
    ```shell
    bash error-suppression.sh 2> /dev/null
    ```
3. Output
    ```shell
    hello
    good morning
    ```

### IV. Xử lí hậu kì
Giả sử trong quá trình thực thi một script bất kì, bạn tạo ra vô số tập tin tạm thời (temporary files) trên hệ thống và bạn định sẽ xóa hết chúng ở cuối script. Nhưng điều gì sẽ xảy ra nếu có một ngoại lệ được ném ra và script của bạn thực thi không thành công? Nhiều khả năng là không có gì cho đến một ngày đẹp trời nào đó :)

<p align="center">
  <img src="https://i.stack.imgur.com/CG8Uz.png"  alt=""/>
</p>

Đến đây không hẳn là hết cách, tuy nhiên thì bây giờ nếu bạn quyết định dọn dẹp hệ thống thì dọn dẹp cái gì bây giờ, nói chung về cơ bản là khá mất thời gian và cách tốt nhất là khi ngoại lệ xảy ra thì chúng ta cũng đồng thời xóa hết những tập tin đã tạo ra trong quá trình chạy script.

Vẫn may mắn như thường lệ là `bash` có hỗ trợ câu lệnh `trap` giúp chúng ta làm việc đó.

1. Tạo một script mới với tên `error-clean-up-file.sh` có nội dung như sau
    ```shell
    #!/bin/bash
    
    TMP=$(mktemp /tmp/logbasex.XXXXXX)
    
    trap 'rm -f $TMP' ERR
    trap 'echo $TMP' ERR
    
    /usr/bin/false
    ```
   
2. Output
    ```shell
    /tmp/logbasex.5toDp0
    ```

Khi bắt gặp một câu lệnh trong script trả về `non-zero exit status` trong quá trình thực thi mà ở đây là `/usr/bin/false` thì `ERR trap` sẽ được kích hoạt để xóa tập tin đã tạo ra và hiển thị tên tập tin đó ra màn hình. Tuy nhiên, chú ý rằng nếu ngoại lệ xảy ra trong `function` thì câu lệnh `trap` sẽ không được thực thi. 

```shell
#!/bin/bash

function tempfile {
    mktemp /tmp/logbasex.XXXXXX
    false
}

TMP=$(tempfile)

trap 'rm -f $TMP' ERR
trap 'echo $TMP' ERR

```

Và đó là lý do mà chúng ta lại một lần nữa dùng đến câu lệnh `set`

```shell
set -o errtrace
```

Việc bây giờ là sửa lại script để thấy ngay điều kì diệu

```shell
#!/bin/bash

set -o errtrace

function tempfile {
    mktemp /tmp/logbasex.XXXXXX
    false
}

TMP=$(tempfile)

trap 'rm -f $TMP' ERR
trap 'echo $TMP' ERR
```


Thanks for the following precious resources

1. https://stackoverflow.com/questions/25378845/what-does-set-o-errtrace-do-in-a-shell-script
2. https://www.baeldung.com/linux/creating-temp-file
3. https://www.shell-tips.com/bash/functions/
4. https://linuxhint.com/bash_error_handling/
5. https://linuxhint.com/bash_pipe_tutorial/
6. https://gist.github.com/mohanpedala/1e2ff5661761d3abd0385e8223e16425 
7. https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/




-----------------------------------

Thank for reading :blush:.