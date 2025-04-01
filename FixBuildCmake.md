# Hướng dẫn cài đặt tmxparser sử dụng vcpkg
## Cài đặt thư viện phụ thuộc bằng vcpkg
tmxparser yêu cầu các thư viện phụ thuộc sau:
- zlib
- tinyxml2 (phiên bản >= 6.0.0)

Bạn có thể sử dụng vcpkg để cài đặt các thư viện này một cách dễ dàng.
### Bước 1: Cài đặt vcpkg (nếu chưa có)
Nếu bạn chưa cài đặt vcpkg, hãy làm theo các bước sau:
``` bash
# Clone repo vcpkg
git clone https://github.com/microsoft/vcpkg.git
cd vcpkg

# Chạy script bootstrap
.\bootstrap-vcpkg.bat  # Windows
./bootstrap-vcpkg.sh   # Linux/macOS
```
### Bước 2: Cài đặt các thư viện phụ thuộc
``` bash
# Cài đặt tinyxml2 và zlib
vcpkg install tinyxml2:x64-windows zlib:x64-windows
```
Lưu ý: Thay thế `x64-windows` bằng triplet phù hợp với hệ điều hành và kiến trúc của bạn nếu cần thiết.
### Bước 3: Tạo thư mục build và chạy CMake
``` bash
mkdir build
cd build
```
Sử dụng lệnh CMake sau để chỉ định vị trí của thư viện tinyxml2 và zlib đã cài đặt thông qua vcpkg:
``` bash
cmake .. -G "Visual Studio 17 2022" -A x64 ^
    -DBUILD_SHARED_LIBS=ON ^
    -DBUILD_STATIC_LIBS=ON ^
    -DCMAKE_TOOLCHAIN_FILE=<VCPKG_ROOT>/scripts/buildsystems/vcpkg.cmake ^
    -Dtinyxml2_DIR="<VCPKG_ROOT>/installed/x64-windows/share/tinyxml2" ^
    -DZLIB_ROOT="<VCPKG_ROOT>/installed/x64-windows"
```
Thay thế `<VCPKG_ROOT>` bằng đường dẫn thực tế đến thư mục vcpkg của bạn.
Ví dụ:
``` bash
cmake .. -G "Visual Studio 17 2022" -A x64 ^
    -DBUILD_SHARED_LIBS=ON ^
    -DBUILD_STATIC_LIBS=ON ^
    -DCMAKE_TOOLCHAIN_FILE=D:/Workspace/BE/visual-studio/Build-libraries/vcpkg/scripts/buildsystems/vcpkg.cmake ^
    -Dtinyxml2_DIR="D:/Workspace/BE/visual-studio/Build-libraries/vcpkg/installed/x64-windows/share/tinyxml2" ^
    -DZLIB_ROOT="D:/Workspace/BE/visual-studio/Build-libraries/vcpkg/installed/x64-windows"
```
### Bước 4: Build và cài đặt
``` bash
# Sử dụng Visual Studio để build
cmake --build . --config Release

# Hoặc cài đặt trực tiếp
cmake --build . --config Release --target install
```
## Lưu ý
- Đảm bảo rằng bạn đã cài đặt phiên bản mới nhất của tinyxml2 và zlib thông qua vcpkg để đảm bảo tương thích.
- Tùy chọn `-DBUILD_SHARED_LIBS=ON` và `-DBUILD_STATIC_LIBS=ON` cho phép xây dựng cả thư viện động và tĩnh.
- Khi chạy với Visual Studio, hãy mở file solution (.sln) được tạo ra bởi CMake trong thư mục build để tiếp tục quá trình phát triển.

## Xử lý sự cố
Nếu bạn gặp lỗi khi CMake không tìm thấy tinyxml2 hoặc zlib, hãy kiểm tra:
1. Đường dẫn đến toolchain file của vcpkg đã chính xác chưa
2. Các thư viện đã được cài đặt đúng kiến trúc (x64-windows) chưa
3. Đường dẫn đến tinyxml2_DIR và ZLIB_ROOT đã chính xác chưa

Để kiểm tra các thư viện đã cài đặt qua vcpkg, bạn có thể dùng lệnh:
``` bash
vcpkg list
```
