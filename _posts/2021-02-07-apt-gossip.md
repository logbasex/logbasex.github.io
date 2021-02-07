## Gói tin và cơ chế xác thực từ Ubuntu Repository

1. Một số khái niệm cơ bản
 - Nếu bạn từng dành thời gian dùng Window, thì có thể bạn đã quen với việc mỗi lần muốn có một phần mềm nào mới trên máy tính thì bạn sẽ tìm kiếm trên internet file installer (thường là `.exe`) để tải về và cài đặt, hoặc là cài đặt từ CDs, DVDs, USBs... Tuy nhiên trên những hệ điều hành mã nguồn mở như Ubuntu, GNU/Linux... các tập tin cài đặt trôi nổi như thế thì không nhiều. Với Ubuntu thì hầu hết phần mềm được đóng gói dưới dạng `.deb` (tương tự `.exe` ở Window) gọi là `package` và **Ubuntu Repository là tập hợp của những server lưu trữ các `package` đó**.  Bạn hãy tưởng tượng nó giống như **App Store/CH Play** trên điện thoại, đó là nơi lưu trữ tập trung của rất nhiều phần mềm, giúp cho end-users/distributors có thể cài đặt hay cập nhật phần mềm theo một cách nhất quán (centralized way), thế cho nó dễ. 
 Rồi thì cài đặt phần mềm như thế nào? Tải file `.deb` về rồi install bằng tay thì cực quá, mà [Larry Wall](http://threevirtues.com/) đã dặn dò rằng lười biếng mới là đức tính tốt của lập trình viên, thế nên năm 1998, `APT` (**A**dvanced **P**ackage **T**ool) software chính thức ra đời. `APT` là package manager ban đầu được design cho `Debian` (`Ubuntu` là một distro hard-forked từ `Debian` nên vẫn dùng `APT` làm package manager) mục đích chính thì để đơn giản hóa quá trình cài cắm, cập nhật packages/depedencies thôi. `APT` có thể được sử dụng qua những câu lệnh như`apt*` (apt-get, apt-cache..) hoặc nếu ai không thích dùng qua CLI thì có thể dùng GUI cho nó trực quan và dễ hiểu tỉ như `Ubuntu Software Center `...
   > Ngoài APT thì còn có một số package manager khác như Snap, Synaptic, aptitude...
   > 
    
- Liên quan đến Ubuntu Repository thì chúng ta có một khái niệm gọi là mirror server. Nói chung thì `Ubuntu` được phân phối (distributed/mirrored) trên rất nhiều servers, và mục đích của việc này là để tải về cho nó nhanh, ví dụ như gửi một request từ Việt Nam sang Lào thì sẽ nhanh hơn sang Singapore vậy đó, tiếp nữa là để giảm tải cho con server chính, thế cho nó nhẹ :satisfied:. Mà mirror thì có hai loại, `primary` hoặc là `secondary`. Loại `primary` thì nghe bảo băng thông khá tốt, up 24/7 tại địa chỉ `<contry-code>.archive.ubuntu.com` (Ex: [vn.archive.ubuntu.com](http://vn.archive.ubuntu.com/)). Còn loại `secondary` thì mình cũng không rõ lắm và có vẻ như không phổ biến. Thường thì trong quá trình cài đặt Ubuntu đã setup sẵn mirror xịn xò cho chúng ta luôn rồi, nhưng vốn dĩ là một hệ điều hành dễ tính, bạn có thể thay đổi mirror thủ công hoặc thông qua command như `apt-select`...
  > Danh sách những repository lưu trữ package nằm ở:
  >> /etc/apt/sources.list
  >
  >> /etc/apt/sources.list.d
  > 
  > Một repository bao gồm ít nhất một thư mực với một vài tập tin .deb ở trong, và hai tập tin đặc biệt
  >> [Packages.gz for the binary packages, và Sources.gz for the source packages](https://www.debian.org/doc/manuals/repository-howto/repository-howto#:~:text=%2Dsrc%20keyword).-,Packages.,Sources.). 

- Giờ đến một ví dụ nhé. Mình muốn đổi tên của một tập tin dùng `rename` command, `rename` không phải là shell built-in command nên mình cần phải cài nó từ Ubuntu Repository thì phải làm thế nào?
  
  Đầu tiên là phải cập nhật cái đã, hì :smiley:
  > sudo apt update
  >>(Từ phiên bản 16.04 apt dùng thay thế cho apt-get)
  
  Việc cập nhật này tải xuống thông tin trạng thái (state information) của những package mới nhất từ repo và lưu hết chúng ở đây. (Nếu không cập nhật thì khi mình cài đặt thì lúc cài mình sẽ chỉ cài bản cũ thôi, chú ý nhe')
  > /var/lib/apt/lists
  
  Tiếp theo thì tiến hành cành cài đặt bằng cách chạy câu lệnh:
  > sudo apt install rename
  
  Nói chi tiết một chút thì khi câu lệnh này được chạy sẽ thực thi những tiến trình con có thể được liệt kê như sau:

  + Từ danh sách repository lưu trong sources.list, nó kiểm tra xem trong hệ thống package rename đã tồn tại chưa, nếu chưa thì kiểm tra xem trên remote repository có package nào là rename không, nếu có thì tiến hành tải xuống package và các dependencies của nó xuống máy local rồi lưu tạm ở đây... (À quên còn quá trình xác thực nữa nhưng nói sau nhe')
    > /var/cache/apt/archives
    > 
  + Tiếp đến là quá trình cài đặt tập tin. Bản chất của quá trình này là sử dụng `dpkg` command:
    > dpkg -i /path/to/.deb 

  + Nếu sau này một ti gian mà hệ thống hết ổ cứng thì nhớ vào đây mà dọn nhé, lên cả chục GB lận (À xóa trong cả `var/lib/apt/lists` nữa tuck_out_tongue_winking_eye.

2. HTTP và vấn đề xác thực
- Vào một ngày đẹp trời thì mình có ngó qua source file (/etc/apt/sources.list) và chợt thấy một vài điều khác lạ:
  > deb http://vn.archive.ubuntu.com/ubuntu/ focal-updates main restricted
  
  Vấn đề ở đây là repository url sử dụng giao thức `HTTP` (unsecured) chứ không phải `HTTPS` như mình nghĩ. Thế là bao nhiêu lo lắng trong ngày dồn hết cả vào đây. Có thể có ai đó đã có hết thông tin của mình về mọi package từ trước đến nay mình đã tải xuống, và cũng có thể là đã có một cuộc tấn công [Man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) để chèn mã độc vào máy trong mỗi lần update rồi chăng? Thào nào dạo này thấy máy chạy chậm quá :(( 


- Tuy nhiên với danh tiếng của một distro với độ bảo mật cao như Ubuntu thì khả năng có lỗi rò rỉ bảo mật với APT hơn 20 năm tuổi thì nghe có vẻ không đúng (Thông tin về các file `.deb` bị leak ra ngoài thì chả sao, đằng nào cũng bán dữ liệu cho bạn Google suốt, hì), và tất nhiên sau khi điều tra thì mình thấy không đúng thật. Và mọi chuyện như thế nào thì chúng ta hãy cùng xem nhé.
  > Ubuntu có hỗ trợ tải xuống package từ https repository thông qua command apt-transport-https package. Và nhớ là sửa repository's url trong sources.list từ http thành https nhé.
- Nhưng nếu người dùng vẫn dùng giao thức `HTTP` thì như thế nào? Làm sao để xác định được package được tải xuống và package được lưu trữ trên repository là giống nhau? Nếu có mã độc được chèn vào source package thì khi tải xuống, người dùng có thể bị đánh cắp tài khoản ngân hàng chẳng hạn, điều đó khá là nguy hiểm. Và để loại bỏ những rủi ro đó, Ubuntu cung cấp một loại niêm phong không thể làm giả (tamper-proof seal) gọi là `package signature` để đảm bảo rằng khi cài đặt thì package này chắc chắn đến từ nhà phát hành chính thức và không bị thay đổi bởi bên thứ 3 dù chỉ là 1 bit.
  

- Nói cho nó dễ thì quy trình xác thực `package signature` bao gồm hai phần chính:
  + Kiểm tra tính xác thực (Authenticity) của tập tin. Đại khái là để đảm bảo tập tin này đến từ nhà phát hành chính thức chứ không phải là cá nhân/tổ chức khác.
    > Cái này dùng gnupg/gpg command nhé.
  + Kiểm tra tính toàn vẹn (Integrity) của tập tin. Kiểu là để đảm bảo nội dung tập tin này không bị thay đổi so với tập tin được lữu trữ trên repository trong quá trình cài đặt.
    >  Cái này dùng sha256sum command nhé.

  Và cụ thể thì:
  1. Như đã nói ở trước thì mỗi distro (distribution) đều có một tập tin `Release` bao gồm binary packages và source packages. 
     > Ngoài ra còn có InRelease[10] nữa.
     > 
     Tập tin`Release` bao gồm mã băm (hash/checksum) MD5Sum của tập tin Packages.gz (trong này bao gồm  các mã băm MD5 của các package khác nữa) sẽ được ký(signed). Chữ ký (signature) này đánh dấu đây là một nguồn đáng tin cậy.
     > Thực ra thì tập tin `Release` hỗ trợ nhiều loại mã hóa ví dụ như bao gồm mã băm MD5Sum/SHA1/SHA256. Nhưng trong bài này mình sẽ dùng SHA256 nhé.
     > 
  2. Sau đó, signed `Release` file sẽ được tải xuống bởi command `apt update` và được lưu chung với tập tin Package.gz
     > Bạn có thể vào /var/lib/apt/lists mở một tập tin Release ra để xem.
     > 
  3. Khi một package sắp được cài đặt, thì đầu tiên package sẽ được tải xuống, sau đó mã băm `MD5Sum` được tạo ra. 
  4. Tiếp theo thì tập tin `Release` đã được ký (signed Release) đó sẽ được kiểm tra về tính xác thực (Authenticity), nếu bước này thành công thì chúng ta sẽ có được mã băm `MD5Sum` của tập tin `Packages.gz`
  cũng như là tập tin đã tải xuống trước đó.
  5. Nếu mã băm ở step 3 và 4 giống nhau thì package sẽ được cài đặt, còn không thì hệ thống sẽ ném ra cảnh báo, vậy thôi. Và tất nhiên là package đã được tải xuống rồi, dù bị cảnh báo bạn vẫn có thể cài đặt nếu bạn muốn nhưng sẽ đi kèm với rửi ro.
  
        > Vẫn còn một số vấn đề khái niệm đến apt-key, apt-secure, gpg... mình lười đưa vào quá, bạn có thể tìm hiểu thêm ở bên ngoài để hiểu rõ hơn. 
  
- Bây giờ lấy luôn cái ví dụ cho nó trực quan chứ lý thuyết quá mà. Mình cùng thử xác thực `libpam-modules` source code nhé.

  1. Đầu tiên bao giờ cũng thế, cập nhật hệ thống cái đã
     > sudo apt update
  
      ```bash
      Hit:1 https://download.docker.com/linux/ubuntu focal InRelease
      Get:2 https://dl.yarnpkg.com/debian stable InRelease [17.1 kB]                                                                                                                            
      Hit:3 https://cli.github.com/packages focal InRelease                                                                                                                                     
      Hit:4 https://artifacts.elastic.co/packages/7.x/apt stable InRelease                                                                                                                      
      Get:5 http://security.ubuntu.com/ubuntu focal-security InRelease [109 kB]                                                                                                                 
      Hit:6 https://apt.mopidy.com buster InRelease                                                                                                                                             
      Ign:7 https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 InRelease                                                                                                                 
      Hit:8 http://vn.archive.ubuntu.com/ubuntu focal InRelease                                                                                                                                 
      Hit:9 http://ppa.launchpad.net/bamboo-engine/ibus-bamboo/ubuntu focal InRelease                                                                                                           
      Hit:10 http://deb.playonlinux.com precise InRelease                                                                                                                                       
      Ign:12 https://dl.bintray.com/tizonia/ubuntu focal InRelease                                                                                                                              
      Get:13 http://vn.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]                                                                                                           
      Hit:14 http://archive.ubuntu.com/ubuntu focal InRelease                                                                                                                                   
      Get:15 https://dl.bintray.com/tizonia/ubuntu focal Release [1,838 B]                                                                                                                      
      Hit:16 https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 Release                                                                                                                  
      Hit:17 http://ppa.launchpad.net/hluk/copyq/ubuntu focal InRelease                                                                      
      Hit:20 http://ppa.launchpad.net/jgmath2000/et/ubuntu focal InRelease                                                                                  
      Get:21 http://vn.archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]                            
      Hit:22 http://ppa.launchpad.net/jonathonf/vim/ubuntu focal InRelease                                                
      Hit:11 https://packages.gitlab.com/gitlab/gitlab-ee/ubuntu focal InRelease
      Hit:23 http://ppa.launchpad.net/linrunner/tlp/ubuntu focal InRelease
      Hit:24 http://ppa.launchpad.net/linuxuprising/apps/ubuntu focal InRelease
      Hit:25 http://ppa.launchpad.net/ondrej/php/ubuntu focal InRelease
      Hit:26 http://ppa.launchpad.net/peek-developers/stable/ubuntu focal InRelease
      Hit:27 http://ppa.launchpad.net/regolith-linux/release/ubuntu focal InRelease
      Fetched 343 kB in 5s (74.6 kB/s)                             
      Reading package lists... Done
      Building dependency tree       
      Reading state information... Done
      27 packages can be upgraded. Run 'apt list --upgradable' to see them.
      ```
      > Giải thích 1 chút
      >> Hit: Package is not change
      >>
      >> Ign: Package is ignore
      >>
      >> Get: Package has new version
  2. Tìm Release file
  
     Di chuyển vào thư mục `/var/lib/apt/lists`, dễ thấy nó với tên...
     > vn.archive.ubuntu.com_ubuntu_dists_focal-updates_InRelease
     >> Tại sao không phải là focal-security hay focal-backports thì xem ở đây: https://help.ubuntu.com/community/UbuntuUpdates
     > 
  3. Tải về tập tin Sources.gz 
     > curl -O http://vn.archive.ubuntu.com/ubuntu/dists/focal-updates/main/source/Sources.gz
  4. Xác thực chữ ký (signature)
     
     Nói qua một chút, GPG là một phần mềm mã hóa không đối xúng sử dụng public key để mã hóa và dùng private key để giải mã. 
     > gpg --verify vn.archive.ubuntu.com_ubuntu_dists_focal-updates_InRelease
     
      Nếu xảy ra lỗi như thế này
     ```bash 
      gpg: Signature made Sat 06 Feb 2021 11:08:29 PM +07
      gpg:                using RSA key 871920D1991BC93C
      gpg: Can't check signature: No public key
     ```
     thì nhớ add public key vào
     > gpg --keyserver hkp://keys.gnupg.net --recv-keys 871920D1991BC93C
  5. Tải về source code
      > sudo apt source libpam-modules
     > 
  6. Kiểm tra tính toàn vẹn của tập tin Sources.gz
     
     Tạo mã băm SHA256 của tập tin đã tải xuống bằng câu lệnh
     > sha256sum Sources.gz
     
     Và của tập tin Release
     > grep main/source/Sources.gz vn.archive.ubuntu.com_ubuntu_dists_focal-updates_InRelease
     
     Mình thử và thấy ra cùng một kết quả:
     > cca9f54803347214ed35bccea9ca19a255cb67c5c767f7636bf4761453748f90
     > 
  7. Kiểm tra tính toàn vẹn của `libpam-modules` source code
     
     Đầu tiên mình kiểm tra version của package `libpam-modules` trước
     > apt show libpam-modules
     ```bash
     Package: libpam-modules
     Version: 1.3.1-5ubuntu4.1
     Priority: required
     Section: admin
     Source: pam
     Origin: Ubuntu
     Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
     ...
     ```
     
      Sau đó thì so sánh mã băm thu được từ hai câu lệnh `sha256sum pam_1.3.1-5ubuntu4.1.dsc` and `zcat Sources.gz | grep pam_1.3.1-5ubuntu4.1.dsc`. Nếu ra cùng một kết quả thì chúng ta đã xác thực tính toàn vẹn thành công.
      > .dsc file chứa metadata cũng như cái loại mã băm của package.
  
Mọi thứ đã xong. Hẹn mọi người ở bài viết tiếp theo !

References:
1. https://ubuntu.com/tutorials/how-to-verify-ubuntu#4-retrieve-the-correct-signature-key
2. https://askubuntu.com/questions/509487/whats-the-official-method-for-checking-integrity-of-a-source-package/509816#509816
3. https://askubuntu.com/questions/253728/how-to-safely-download-and-gpg-verify-a-debian-source-package
4. https://debian-handbook.info/browse/stable/sect.package-authentication.html
5. https://wiki.debian.org/DebianRepository/Format
6. https://www.debian.org/doc/manuals/securing-debian-manual/deb-pack-sign.en.html
7. https://difyel.com/linux/usr/bin/apt-key/
8. https://askubuntu.com/questions/352952/are-repository-lists-secure-is-there-an-https-version
9. https://stackoverflow.com/questions/13507162/how-dsc-files-are-related-to-deb-and-source-code-files
10. https://wiki.debian.org/DebianRepository/Format#A.22Release.22_files