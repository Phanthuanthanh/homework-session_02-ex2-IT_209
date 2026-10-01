# Báo Cáo Bài Tập 2: Khởi Tạo User Thường và Thiết Lập Đặc Quyền Quản Trị (Sudoers Configuration)

## 1. Mục tiêu
- Khởi tạo tài khoản người dùng thường (`devops`) trên Ubuntu Server nhằm tuân thủ nguyên tắc đặc quyền tối thiểu (Least Privilege).
- Cấp quyền quản trị cho user `devops` thông qua nhóm `sudo`.
- Sao chép và phân quyền chính xác cấu hình SSH Key để cho phép đăng nhập từ xa qua SSH mà không cần dùng quyền `root`.

---

## 2. Các bước thực hiện và Lệnh Linux

### Bước 1: Tạo tài khoản người dùng mới `devops`
```bash
sudo adduser devops
```
- **Mục đích:** Tạo user mới tên là `devops`, tự động khởi tạo thư mục home `/home/devops` và yêu cầu thiết lập mật khẩu cho user.

### Bước 2: Thêm user `devops` vào nhóm quản trị `sudo`
```bash
sudo usermod -aG sudo devops
```
- **Mục đích:** Thêm user `devops` vào group `sudo` để user này có quyền thực thi các lệnh quản trị hệ thống bằng lệnh `sudo`.

### Bước 3: Sao chép cấu hình SSH Key từ `root` sang `devops`
```bash
sudo rsync --archive --chown=devops:devops ~/.ssh /home/devops/
```
- **Mục đích:** Sao chép toàn bộ thư mục `.ssh` và file `authorized_keys` từ user `root` sang thư mục home của `devops`, đồng thời tự động cập nhật chủ sở hữu (`chown`) sang `devops:devops`.

### Bước 4: Thiết lập phân quyền bảo mật cho thư mục SSH
```bash
# Phân quyền thư mục .ssh (chỉ chủ sở hữu có quyền Đọc, Ghi, Truy cập)
sudo chmod 700 /home/devops/.ssh

# Phân quyền file authorized_keys (chỉ chủ sở hữu có quyền Đọc và Ghi)
sudo chmod 600 /home/devops/.ssh/authorized_keys
```
- **Mục đích:** Đảm bảo đúng chuẩn phân quyền an toàn của OpenSSH (700 cho thư mục `.ssh` và 600 cho file `authorized_keys`) để dịch vụ SSH chấp nhận khóa xác thực.

---

## 3. Kiểm tra và Log Terminal

### 3.1. Đăng nhập SSH từ máy cá nhân bằng user `devops`
Lệnh thực hiện:
```bash
ssh -i /path/to/private_key devops@<IP_ADDRESS_DROPLET>
```

**Log Terminal kết nối:**
```text
$ ssh -i ~/.ssh/id_rsa devops@159.223.x.x
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Thu Oct  1 14:30:15 2026 from 14.161.x.x
devops@ubuntu-server:~$
```
-> **Kết quả:** Đăng nhập thành công vào server với tài khoản `devops` thông qua SSH Key.

### 3.2. Kiểm tra đặc quyền quản trị với lệnh `sudo whoami`
Lệnh thực hiện trên server:
```bash
devops@ubuntu-server:~$ sudo whoami
```

**Log Terminal kết quả:**
```text
devops@ubuntu-server:~$ sudo whoami
[sudo] password for devops: 
root
```
-> **Kết quả:** Lệnh yêu cầu nhập mật khẩu của user `devops` và hiển thị kết quả trả về là `root`.

---

## 4. Kết quả mong đợi
- Tài khoản `devops` đăng nhập thành công qua SSH Key mà không gặp lỗi xác thực.
- Thực thi lệnh `sudo` thành công mà không gặp bất kỳ lỗi phân quyền hay sudoers nào.
