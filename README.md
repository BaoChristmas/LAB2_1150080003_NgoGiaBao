# LAB 2 – MÃ HÓA ĐỐI XỨNG HIỆN ĐẠI

## 1. Thông tin sinh viên

**Họ và tên:** Ngô Gia Bảo  
**MSSV:** 1150080003  
**Lớp:** 11_THMT  
**Môn học:** An toàn và bảo mật hệ thống thông tin  
**Video thực hành:** *https://youtu.be/PvMDlhI1oQg*  

## 2. Môi trường và công cụ thực hành

- **Hệ điều hành:** Windows 11
- **Ngôn ngữ & JDK:** Java (JDK 21, `maven.compiler.release = 17`) và Python 3
- **IDE:** Apache NetBeans IDE 25 (GUI Builder)
- **Quản lý dự án & thư viện Java:** Maven (`pom.xml`), `com.microsoft.sqlserver:mssql-jdbc:12.8.1.jre11`, `org.bouncycastle:bcprov-jdk18on:1.78.1`
- **Thư viện Python:** `cryptography`, `tkinter`
- **Cơ sở dữ liệu:** Microsoft SQL Server 2022 Express (instance `SQLEXPRESS`, cổng TCP `14333`, chế độ Mixed Mode) và SQL Server Management Studio (SSMS) 22
- **Cấu trúc project Java:** `LAB2_MaHoaHienDai`, package `week_02`, lớp khởi chạy chính `week_02.frm_Menu`


## 3. Nội dung kỹ thuật và kết quả thực hành

### 3.1. Chuẩn bị dự án, lớp dùng chung và SQL Server

- Khởi tạo dự án Maven `LAB2_MaHoaHienDai`, cấu hình dependency `mssql-jdbc` và `bcprov-jdk18on` (BouncyCastle) trong `pom.xml`.
- Xây dựng các lớp tiện ích và kết nối: `CipherUtil` (UTF-8, Base64, HEX, `SecureRandom`, `PBKDF2WithHmacSHA256`), `FileTextIO` (đọc/lưu tệp UTF-8), `DbConfig`, `HistoryDao`, `UserDao`, cùng hai form `frm_KetNoiSQL` và `frm_LichSu`.
- Khởi tạo cơ sở dữ liệu `LAB2_MaHoaHienDai` gồm hai bảng `dbo.app_users` (lưu `salt` 16 byte và `password_hash` PBKDF2 32 byte) và `dbo.crypto_history` (chỉ lưu độ dài dữ liệu, không lưu bản rõ hay khóa).

### 3.2. Các thuật toán mã hóa đối xứng hiện đại (Bài 2.1 – 2.6)

| Bài | Thuật toán / Chế độ | Dữ liệu kiểm thử | Kết quả đạt được |
| :--- | :--- | :--- | :--- |
| 2.1 | DES (`DES/CBC/PKCS5Padding`) | Plaintext `Xin chào Lab 2 - Mã hóa hiện đại!`, Key `12345678` | Mã hóa/giải mã đúng tiếng Việt UTF-8; mỗi lần Encrypt ra bản mã khác nhau (IV ngẫu nhiên); sai khóa báo lỗi. |
| 2.2 | TripleDES (`DESede/CBC/PKCS5Padding`) | Key `123456789012345678901234` (24 ký tự) | Mã hóa/giải mã đúng; nhập Key 23 ký tự báo lỗi đúng quy định. |
| 2.3 | AES-256-GCM + Register/Login SQL | Tài khoản `sinhvien01`, Key `MatKhauAES@2026` | Register/Login qua SQL Server thành công, cấp Ticket 128-bit; sửa 1 ký tự bản mã báo lỗi `AEADBadTagException`. |
| 2.4 | RC4 (KSA + PRGA) | Plaintext `Plaintext`, Key `Key` | Bản mã HEX đúng chuẩn `BBF316E8D940AF0AD3`; giải mã khôi phục chính xác. |
| 2.5 | Twofish-CBC (BouncyCastle) | Key `Twofish@Lab2`, IV 16 byte (`Generate IV`) | Mã hóa/giải mã đúng; `Save File` lưu `IV=...` + bản mã (không lưu Key), `Open File` khôi phục đầy đủ IV. |
| 2.6.1 | AES-256-CTR (Java) | Key 32 byte + Nonce 12 byte ngẫu nhiên (Base64) | Độ dài bản mã bằng đúng độ dài bản rõ (không đệm); giải mã chính xác văn bản dài. |
| 2.6.2 | AES-256-CBC (Python Tkinter) | Văn bản dài tiếng Việt + `--self-test` | Chạy `py aes_cbc_gui.py --self-test` đạt `SELF-TEST OK`; giao diện mã hóa/giải mã đúng toàn văn. |

### 3.3. Menu chính (`frm_Menu`) và kiểm thử tổng thể

- Form `frm_Menu` (`EXIT_ON_CLOSE`) điều hướng tới toàn bộ 6 form thuật toán cùng `frm_KetNoiSQL` và `frm_LichSu` (`DISPOSE_ON_CLOSE`).
- Biên dịch `Clean and Build` đạt `BUILD SUCCESS`, chạy project (`F6`) kiểm tra đầy đủ các chức năng và đối chiếu dữ liệu trong `dbo.app_users`, `dbo.crypto_history` trên SSMS.

## 4. Hướng dẫn chạy chương trình

1. **Khởi tạo cơ sở dữ liệu (chạy 1 lần trên SSMS):** Mở `01_Tao_CSDL.sql` và thực thi trên SQL Server Express (`SQLEXPRESS`, cổng TCP `14333`), đồng thời tạo tài khoản SQL có quyền `db_datareader` và `db_datawriter` trên `LAB2_MaHoaHienDai`.
2. **Chạy chương trình Java (NetBeans):**
   - Mở project `LAB2_MaHoaHienDai` trong Apache NetBeans.
   - Nhấn **`Shift+F11`** (Clean and Build), sau đó nhấn **`F6`** (Run Project) để mở `frm_Menu`.
3. **Chạy chương trình Python (`2.6 AES-CBC`):**
   - Cài thư viện: `py -m pip install cryptography`
   - Chạy tự kiểm thử: `cd LAB2_Python && py aes_cbc_gui.py --self-test`
   - Chạy giao diện: `py aes_cbc_gui.py`

## 5. Kết quả đánh giá

**Trạng thái: PASS**

Hoàn thành đầy đủ các thuật toán mã hóa đối xứng hiện đại (DES, TripleDES, AES-GCM, RC4, Twofish, AES-CTR, AES-CBC Python), tích hợp đăng ký/đăng nhập và lưu lịch sử trên SQL Server, menu chính, các tệp SQL và báo cáo kiểm thử của **Lab 2**.
