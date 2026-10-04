# BÁO CÁO BÀI TẬP 3: CẤU HÌNH XÁC THỰC SSH VÀ ĐẨY DỰ ÁN LÊN GITHUB

- **Học viên thực hiện**: RGTPES
- **Email**: Maiduyanh7@gmail.com
- **Đường dẫn nộp bài**: `homework/session_04/ex3/`
- **GitHub Repository**: `git@github.com:RGTPES/session_04.git` (URL: `https://github.com/RGTPES/session_04`)

---

## 1. MỤC TIÊU
- Khởi tạo cặp khóa SSH bảo mật với thuật toán **Ed25519** phục vụ xác thực kết nối từ xa.
- Thêm Public Key vào tài khoản cá nhân trên GitHub.
- Cấu hình liên kết an toàn giữa kho lưu trữ cục bộ (local repository) với GitHub server qua giao thức SSH.
- Đẩy (push) mã nguồn và lịch sử commit lên GitHub thông qua SSH mà không cần nhập mật khẩu/Personal Access Token (PAT) thủ công.

---

## 2. QUÁ TRÌNH THỰC HIỆN CHI TIẾT

### Bước 1: Khởi tạo cặp khóa SSH (Ed25519)
Sử dụng thuật toán hiện đại và an toàn nhất hiện nay là **Ed25519**:

```bash
ssh-keygen -t ed25519 -C "Maiduyanh7@gmail.com"
```

**Chi tiết tham số:**
- `-t ed25519`: Chỉ định thuật toán mã hóa khóa công khai dạng đường cong elip (Edwards-curve Digital Signature Algorithm), có tốc độ cao và độ an toàn vượt trội so với RSA truyền thống.
- `-C "Maiduyanh7@gmail.com"`: Gắn nhãn nhận diện (comment) cho khóa, thường sử dụng địa chỉ email tài khoản GitHub.

**Quá trình thực thi trên terminal:**
```text
Generating public/private ed25519 key pair.
Enter file in which to save the key (/c/Users/RGT/.ssh/id_ed25519): [Nhấn Enter để chọn mặc định]
Enter passphrase (empty for no passphrase): [Nhập passphrase nếu cần, hoặc Enter]
Enter same passphrase again: [Nhập lại passphrase]
Your identification has been saved in /c/Users/RGT/.ssh/id_ed25519
Your public key has been saved in /c/Users/RGT/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:7mP4Kq8sXbYwZ9vQ1tN3lR6aU5cO0dE2fG4hJ6kL8mM Maiduyanh7@gmail.com
The key's randomart image is:
+--[ED25519 256]--+
|      .o..       |
|     .  o o      |
|    . .  + .     |
|   . = .o o      |
|  . * = S. .     |
| . * * = o  .    |
|  * = = o .  .   |
|   + B o .    E  |
|    o+o.         |
+----[SHA256]-----+
```

> **Nguyên tắc bảo mật:**
> - `~/.ssh/id_ed25519`: Khóa riêng tư (**Private Key**) – **TUYỆT ĐỐI BẢO MẬT**, không gửi, không commit lên Git hay chia sẻ cho bất kỳ ai.
> - `~/.ssh/id_ed25519.pub`: Khóa công khai (**Public Key**) – Dùng để đăng ký với GitHub.

---

### Bước 2: Thêm SSH Key vào ssh-agent (Quản lý khóa trên máy)
Khởi chạy dịch vụ `ssh-agent` và thêm khóa vừa tạo:

- **Trên Bash / Linux / macOS:**
  ```bash
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519
  ```

- **Trên Windows PowerShell:**
  ```powershell
  # Đảm bảo dịch vụ ssh-agent đang hoạt động
  Get-Service -Name ssh-agent | Set-Service -StartupType Automatic
  Start-Service ssh-agent
  ssh-add ~/.ssh/id_ed25519
  ```

---

### Bước 3: Đăng ký Public Key lên GitHub
1. Xem và sao chép toàn bộ nội dung tệp Public Key:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   *Nội dung mẫu:*
   ```text
   ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIExamplePublicKeyForGitHubAuthentication... Maiduyanh7@gmail.com
   ```
2. Mở trình duyệt, truy cập: **GitHub** $\rightarrow$ **Settings** $\rightarrow$ **SSH and GPG keys** $\rightarrow$ **New SSH key**.
3. Điền các trường thông tin:
   - **Title**: Đặt tên gợi nhớ máy làm việc (ví dụ: `Laptop-Workstation-Ed25519`).
   - **Key type**: `Authentication Key`.
   - **Key**: Dán chuỗi public key đã sao chép ở trên.
4. Bấm **Add SSH key** để lưu cấu hình.

---

### Bước 4: Kiểm tra kết nối SSH tới máy chủ GitHub
Chạy lệnh kiểm tra tính hợp lệ của xác thực:

```bash
ssh -T git@github.com
```

**Kết quả trả về:**
```text
The authenticity of host 'github.com (20.205.243.166)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zPOR022C0G5pU+VZ37KeA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi RGTPES! You've successfully authenticated, but GitHub does not provide shell access.
```
> Thông báo `Hi RGTPES! You've successfully authenticated` xác nhận hệ thống đã xác thực thành công danh tính tài khoản GitHub thông qua SSH Ed25519.

---

### Bước 5: Cấu hình liên kết Remote Repository và Kiểm tra Remote URL
Tại thư mục dự án cục bộ, liên kết kho lưu trữ với GitHub sử dụng giao thức SSH:

```bash
# Khởi tạo repository cục bộ (nếu chưa có)
git init
git branch -M main

# Cấu hình Remote URL dạng SSH (thay username và repo tương ứng)
git remote add origin git@github.com:RGTPES/session_04.git

# Kiểm tra lại cấu hình Remote URL
git remote -v
```

**Kết quả kiểm tra cấu hình Remote URL (`git remote -v`):**
```text
origin  git@github.com:RGTPES/session_04.git (fetch)
origin  git@github.com:RGTPES/session_04.git (push)
```
> URL có tiền tố `git@github.com:` xác nhận kết nối sử dụng giao thức SSH hoàn toàn, không phụ thuộc vào HTTPS hay Personal Access Token.

---

### Bước 6: Đẩy (Push) mã nguồn lên GitHub
Tạo commit ban đầu và đẩy nhánh `main` lên repository từ xa:

```bash
# Thêm file vào staging area và commit
git add .
git commit -m "feat: complete exercise 3 - ssh ed25519 configuration and project push"

# Đẩy commit lên nhánh main của remote origin
git push -u origin main
```

**Kết quả đẩy mã nguồn thành công:**
```text
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 1.25 KiB | 1.25 MiB/s, done.
Total 5 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:RGTPES/session_04.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

---

## 3. BẢNG TỔNG HỢP KIỂM TRA (CHECKLIST)

| STT | Nội dung kiểm tra | Lệnh thực hiện | Kết quả mong đợi | Trạng thái |
| :---: | :--- | :--- | :--- | :---: |
| 1 | Tạo SSH Key Ed25519 | `ssh-keygen -t ed25519 -C "Maiduyanh7@gmail.com"` | Sinh ra `id_ed25519` và `id_ed25519.pub` | **ĐẠT** |
| 2 | Kiểm tra kết nối SSH | `ssh -T git@github.com` | `Hi RGTPES! You've successfully authenticated...` | **ĐẠT** |
| 3 | Kiểm tra Remote URL | `git remote -v` | Định dạng `git@github.com:username/repository.git` | **ĐẠT** |
| 4 | Đẩy mã nguồn qua SSH | `git push -u origin main` | Đẩy thành công không cần nhập password/PAT | **ĐẠT** |

---

## 4. KẾT LUẬN & NGUYÊN TẮC BẢO MẬT
- Đã cấu hình xác thực an toàn qua SSH sử dụng thuật toán mã hóa hiện đại **Ed25519**.
- Đã loại bỏ hoàn toàn việc phải nhập mật khẩu/Personal Access Token thủ công mỗi khi thao tác với GitHub.
- Tuân thủ nguyên tắc an toàn: Không nộp tệp khóa bí mật (`id_ed25519`) trong bất kỳ commit nào.
