---
title: 'Handle exception in Bash'
date: 2021-03-13
permalink: /posts/2021/03/handle-exception-in-bash/
tags:
- bash
- linux
---

## Introduction

Trong các ngôn ngữ lập trình bậc cao phổ biến như Java khi có exception xảy ra chúng ta thường có hai cách giải quyết đó là dừng chương trình bằng việc ném ra ngoại lệ sử dụng `throw/throws` keyword hoặc bắt ngoại lệ để chương trình bỏ qua ngoại lệ đó và chạy tiếp bằng cách sử dụng `try/catch` block.

## Môi trường

- Bash shell

## Kiến thức cơ bản 

- Terminal là môi trường cho phép nhận `input` (text) từ bàn phím..., gửi `input` (command) đến `shell` và nhận `output` trả về từ `shell`.
- Shell vừa là một trình thông dịch (`command line interpreter`) giúp bạn thực thi câu lệnh mà người dùng gửi đến thông qua `terminal/terminal-emulator`, vừa là một ngôn ngữ lập trình. Nếu như với Java, chúng ta sử dụng file với đuôi `.java` thì với shell, mà trong phạm vi bài viết này là `bash` (`Bourne-Again SHell`), chúng ta sử dụng file với đuôi `.sh` (stand for shell) hay còn được gọi là `bash script`.

- Pipeline là một chuỗi của một hoặc nhiều câu lệnh được phân cách bằng kí tự `|` hay `|&`. Ví dụ:
    ```shell script
     ls $PWD | grep logbasex.sh
    ```
  Đây là một `pipeline` cơ bản có thể được hiểu như sau: Liệt kê các tập tin trong thư mục hiện tại rồi tìm kiếm những tập tin có tên là `logbasex.sh` và in kết quả ra màn hình, tức là `output` của câu lệnh thứ nhất `ls $PWD` được sử dụng làm `input` của câu lệnh thứ hai `grep logbasex.sh`.

- Mỗi lần bạn gõ câu lệnh trên terminal và nhấn phím `Enter`, `bash` sẽ thực thi câu lệnh đó và trả về một `exit status` hay còn được gọi là `return code`, là một số nguyên nằm trong khoảng [0,255]. Theo quy ước thì khi một câu lệnh thực thi thành công sẽ trả về `0 exit status`, còn nếu có lỗi xảy ra thì [`non-zero exit status`](https://tldp.org/LDP/abs/html/exitcodes.html) sẽ được trả về tỉ như `command not found` là `127`. Chúng ta có một biến đặc biệt lưu trữ `exit status` của câu lệnh đã được thực thi trước đó, và để kiểm tra bạn có thể làm như sau: 

    ```shell script
    echo $?
    ```
  
  Ex:
  ```shell script
  ls logbasex
  echo $?
  ```
  Output
  ```shell script
  ls: cannot access 'logbasex': No such file or directory
  2  
  ```
  
## Tạo   

1. Để xử lý ngoại lệ thì đầu tiên phải có chương trình ném ra ngoại lệ trước đã. Mình tạo một file có tên `error.sh` với nội dung như sau:
    ```shell scriptbug
    #!/bin/bash
     
    ls logbasex
    good morning
    love
    ```
2. Execute file  
    ```shell script
    bash error.sh
    ```
 3. Output 
    ```shell script
    ls: cannot access 'logbasex': No such file or directory
    error.sh: line 4: good: command not found
    error.sh: line 5: love: command not found
    ```

## Xử lý ngoại lệ    

#####I. Trường hợp cơ bản
 
 Như đã thấy ở trên, cả 3 câu lệnh đều thực thi không thành công, chương trình kết thúc và chúng ta có 3 dòng báo lỗi hiển thị trên màn hình. Ok. Nhưng bây giờ nếu bạn muốn chương trình kết thúc ngay tại dòng lệnh đầu tiên mà thực thi không thành công thì làm thế nào? 

Thật may mắn là `bash` có hỗ trợ một `built-in command` gọi là [`set`](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html) với option `-e`.

> Exit immediately if a pipeline (see Pipelines), which may consist of a single simple command (see Simple Commands), a list (see Lists), or a compound command (see Compound Commands) returns a non-zero status. **The shell does not exit if** the command that fails is part of the command list immediately following a while or until keyword, part of the test in an if statement, part of any command executed in a && or || list except the command following the final && or ||, **any command in a pipeline but the last**, or if the command’s return status is being inverted with !. If a compound command other than a subshell returns a non-zero status because a command failed while -e was being ignored, the shell does not exit. A trap on ERR, if set, is executed before the shell exits.
>
> This option applies to the shell environment and each subshell environment separately (see Command Execution Environment), and may cause subshells to exit before executing all the commands in the subshell. 
>
> If a compound command or shell function executes in a context where -e is being ignored, none of the commands executed within the compound command or function body will be affected by the -e setting, even if -e is set and a command returns a failure status. If a compound command or shell function sets -e while executing in a context where -e is ignored, that setting will not have any effect until the compound command or the command containing the function call completes.

1. Chúng ta tạo một script mới có tên `error-e.sh` với nội dung như sau
    ```shell script
    #!/bin/bash
    
    set -e
    
    ls logbasex
    good morning
    love
    ```
2. Thực thi
    ```shell script
    bash -e error.sh 
    
    OR
    
    bash error-e.sh
    ```
3. Output
    ```shell script
    ls: cannot access 'logbasex': No such file or directory
    ```

#####II. Ngoại lệ của `set -e` option (any command in a pipeline but the last)

1. Chúng ta tạo một script mới có tên `error-pipeline.sh` với nội dung như sau
    ```shell script
    #!/bin/bash
    set -e
    
    logbasex | echo "hello"
    echo "good morning"
    ```
2. Thực thi
    ```shell script
    bash error-pipeline.sh
    ```
3. Output
    ```shell script
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
```shell script
set -o pipefail
logbasex | grep some-string /non/existent/file | sort
echo "${PIPESTATUS[0]} ${PIPESTATUS[1]} ${PIPESTATUS[2]}" $?
```
thì output sẽ là 
```shell script
grep: /non/existent/file: No such file or directory
logbasex: command not found
127 2 0 2
```
Như bạn có thể thấy, exit status của pipeline trên chính là bằng `2`.

Và...

Việc bây giờ là sửa lại script và thực thi để thấy ngay điều kì diệu
```shell script
#!/bin/bash

set -eo pipefail
logbasex | echo "hello"
echo "good morning"
```

Oops :(( Output nhìn có vẻ không đẹp lắm, sẽ thật tuyệt nếu bỏ được error log ra khỏi output. Và thật may mắn, `bash` hỗ trợ chúng ta redirect error ouput (`STDERR`) vào `hố đen vô cực` như sau
```shell script
bash error-pipefail.sh 2> /dev/null
```
> Mình sẽ giải thích câu lệnh trên ở một bài viết khác. 

#####III. Bỏ qua ngoại lệ
Trước giờ chúng ta chỉ nói đến vấn đề ném ra ngoại lệ, nhưng bây giờ chúng ta muốn chương trình vẫn tiếp tục thực thi khi bắt gặp ngoại lệ thì làm thế nào? 

Thú thật thì chả biết làm thế nào ngoài sử dụng toán tử `OR` cả. Toán tử này trong `bash script` được biểu diễn dưới kí tự `||` (double pipe) và cách dùng cũng tương tự như trong Java, có hỗ trợ [short-circuit](https://www.geeksforgeeks.org/short-circuiting-in-java-with-examples).

- Việc bây giờ là tạo một script mới với tên `error-suppression.sh` có nội dung như sau: 
    ```shell script
    #!/bin/bash
    
    set -eo pipefail
    logbasex | echo "hello" || true
    echo "good morning"
    ```
- Thực thi  
    ```shell script
    bash error-suppression.sh 2> /dev/null
    ```
- Output
    ```shell script
    hello
    good morning
    ```

#####IV. Xử lí hậu kì
Giả sử bây giờ bạn có một script tạo file trên server và trong quá trình tạo file, có ngoại lệ xảy ra khiến chương trình dừng lại

-----------------------------------

Thank for reading :blush:.