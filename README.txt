# Lab13 – Tạo báo cáo cơ bản bằng Crystal Report

## Thông tin sinh viên
- **Họ và tên:** Nguyễn Quang Dũng
- **Mã sinh viên:** 24003351
- **Học phần:** Lập trình .NET (IT3212)
- **Cơ sở dữ liệu:** `QLSinhVien` (SQL Server)
- **Project:** `Lab13_CrystalReport`

## 1. Giới thiệu
Lab 13 hoàn thiện 10 bài thực hành về báo cáo:
1. Kiểm tra môi trường báo cáo.
2. Tạo view `vw_BaoCaoSinhVien`.
3. Tạo `DataTable` làm nguồn báo cáo.
4. Hiển thị báo cáo trên `CrystalReportViewer`.
5. Xuất báo cáo ra PDF.
6. Báo cáo danh sách khoa kèm số lượng sinh viên.
7. Định dạng báo cáo: tiêu đề, logo, ngày lập, số trang, đường kẻ, định dạng ngày/điểm.
8. Báo cáo danh sách học phần, sắp xếp theo tên.
9. Tích hợp menu Báo cáo, không mở trùng form, giải phóng tài nguyên.
10. Kiểm thử in và xuất PDF.

Ứng dụng có 3 mẫu báo cáo:
- Báo cáo Danh sách Sinh viên (`vw_BaoCaoSinhVien`).
- Báo cáo Danh sách Khoa (`vw_BaoCaoKhoa`).
- Báo cáo Danh sách Học phần (`vw_BaoCaoHocPhan`).

## 2. Môi trường thực hành
| Thành phần | Phiên bản / ghi chú |
|---|---|
| Hệ điều hành | Windows 11 (64-bit) |
| .NET SDK | .NET 10.0 (target `net10.0-windows`) |
| IDE | Visual Studio 2022 (hoặc `dotnet` CLI) |
| SQL Server | Microsoft SQL Server 2022 Express 16.0.1000.6, instance `.\SQLEXPRESS` |
| Report engine | `ReportViewerPanel` – engine nội bộ tương thích quy trình Crystal Reports |
| Định dạng trang | A4 Portrait, lề 40 px, xuất PDF 1 trang |

### Ghi chú về SAP Crystal Reports
Máy thực hành **chưa cài SAP Crystal Reports for Visual Studio/runtime** (không tìm thấy các assembly `CrystalDecisions.*`). Vì vậy project sử dụng `ReportViewerPanel` – một report viewer nội bộ có cùng luồng xử lý:
- Nhận `DataTable` từ tầng BUS/DAL.
- Dựng bố cục báo cáo theo section: Report Header, Page Header, Details, Report Footer, Page Footer.
- Có toolbar In, Xuất PDF, Xuất HTML, Phóng to/Thu nhỏ, Thiết kế, Làm mới.
- Xuất PDF chuẩn A4, có logo, tiêu đề, ngày lập, số trang và định dạng ngày/điểm.

Nếu máy có SAP Crystal Reports runtime, có thể thay `ReportViewerPanel` bằng `CrystalReportViewer`, dùng `ReportDocument.SetDataSource(data)` và `ExportToDisk(ExportFormatType.PortableDocFormat, path)` như code mẫu trong đề bài. Tầng dữ liệu `Lab13.DAL`, `Lab13.BUS` và 3 view SQL giữ nguyên.

## 3. Cấu trúc project
```
lab13/
├─ Database/
│  └─ 01_Lab13_Views_And_Data.sql       # 3 view báo cáo + dữ liệu mẫu
├─ Lab13.DAL/
│  ├─ DbConnectionFactory.cs
│  └─ BaoCaoDAL.cs                       # SqlDataAdapter -> DataTable
├─ Lab13.BUS/
│  └─ BaoCaoService.cs                   # nghiệp vụ báo cáo
├─ Lab13_CrystalReport/
│  ├─ Program.cs                         # entry point + --test/--export/--capture
│  ├─ MainForm.cs                        # menu, dashboard, mở báo cáo
│  ├─ ReportFormBase.cs                  # form nền cho 3 báo cáo
│  ├─ frmBaoCaoSinhVien.cs
│  ├─ frmBaoCaoKhoa.cs
│  ├─ frmBaoCaoHocPhan.cs
│  └─ ReportViewerPanel.cs               # viewer, in, xuất PDF/HTML, thiết kế
├─ Reports/                              # PDF mẫu + đặc tả thiết kế báo cáo
├─ Screenshots/                          # ảnh chụp giao diện và báo cáo
├─ Lab13_CrystalReport.sln
├─ Lab13_CrystalReport.slnx
└─ README.txt
```

## 4. Chuẩn bị cơ sở dữ liệu
1. Mở SQL Server Management Studio.
2. Kết nối instance `.\SQLEXPRESS`.
3. Chạy `Database/01_Lab13_Views_And_Data.sql`.
   - Script tạo/cập nhật 3 view: `vw_BaoCaoSinhVien`, `vw_BaoCaoKhoa`, `vw_BaoCaoHocPhan`.
   - Nếu bảng `SinhVien` có ít hơn 20 dòng, script bổ sung 10 sinh viên mẫu (không trùng mã).
   - Yêu cầu CSDL `QLSinhVien` đã có các bảng `Khoa`, `SinhVien`, `HocPhan`, `DangKy` từ Lab 12.

Kiểm tra nhanh:
```sql
USE QLSinhVien;
SELECT COUNT(*) FROM dbo.SinhVien;
SELECT * FROM dbo.vw_BaoCaoSinhVien ORDER BY TenKhoa, HoTen;
SELECT * FROM dbo.vw_BaoCaoKhoa ORDER BY MaKhoa;
SELECT * FROM dbo.vw_BaoCaoHocPhan ORDER BY TenHP;
```

## 5. Cách chạy
1. Mở `Lab13_CrystalReport.sln` hoặc `Lab13_CrystalReport.slnx` bằng Visual Studio.
2. Chọn `Lab13_CrystalReport` làm Startup Project.
3. Kiểm tra chuỗi kết nối trong `Lab13_CrystalReport/App.config` (mặc định `.\SQLEXPRESS`, CSDL `QLSinhVien`).
4. Nhấn **F5**.
5. Từ menu **Báo cáo** hoặc sidebar, mở:
   - Danh sách Sinh viên.
   - Danh sách Khoa.
   - Danh sách Học phần.
6. Trong form báo cáo:
   - **In báo cáo:** mở hộp thoại in.
   - **Xuất PDF (Bài 5):** chọn nơi lưu file PDF.
   - **Xuất HTML:** xuất bảng dữ liệu ra HTML.
   - **Thiết kế:** xem bố cục section của báo cáo.
   - **Phóng to/Thu nhỏ/Làm mới:** điều khiển xem trước.

Chạy bằng dòng lệnh:
```powershell
dotnet build .\Lab13_CrystalReport.slnx
dotnet run --project .\Lab13_CrystalReport\Lab13_CrystalReport.csproj -- --test
dotnet run --project .\Lab13_CrystalReport\Lab13_CrystalReport.csproj -- --export .\Reports
dotnet run --project .\Lab13_CrystalReport\Lab13_CrystalReport.csproj -- --capture .\Screenshots
```

## 6. Kết quả kiểm thử
- Build: thành công, 0 lỗi.
- Smoke test: 30 sinh viên, 3 khoa, 5 học phần; các cột bắt buộc có mặt; học phần sắp xếp tăng dần theo `TenHP`.
- Xuất PDF: 3 file, mỗi file 1 trang A4 595 × 842 pt:
  - `Reports/DanhSachSinhVien.pdf`
  - `Reports/DanhSachKhoa.pdf`
  - `Reports/DanhSachHocPhan.pdf`
- In: mở `PrintDialog` và in qua `PrintDocument`.
- Mở trùng báo cáo: lần mở thứ hai chỉ kích hoạt form đã có.
- Đóng báo cáo: form được `Dispose`, gỡ khỏi danh sách quản lý.

## 7. Sản phẩm nộp
- `24003351_NguyenQuangDung_Lab13.docx`: báo cáo Word.
- `24003351_NguyenQuangDung_SourceCode_Lab13.zip`: mã nguồn đầy đủ.
- `Reports/*.pdf`: 3 PDF mẫu xuất từ báo cáo.
- `Screenshots/*.png`: ảnh chụp giao diện, báo cáo và thiết kế report.
- `Reports/README.md`: đặc tả 3 báo cáo và hướng dẫn tái tạo bằng Crystal Reports.
- GitHub: tạo repository `lab13` và dán link vào form nộp bài.

## 8. Ghi chú thêm
- Project dùng `System.Data.SqlClient` theo đúng nội dung học phần; các cảnh báo `CS0618` của .NET 10 không ảnh hưởng chức năng.
- `ApplicationHighDpiMode` được đặt `DpiUnaware` để bố cục báo cáo A4 và ảnh xuất PDF ổn định trên các màn hình có tỉ lệ scaling khác nhau.
- Không đưa thư mục `bin`, `obj`, file `.user` vào file zip nộp bài.
