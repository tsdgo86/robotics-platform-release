# Hướng dẫn cài đặt `cubon-robot-base` / `cubon-shared-libs`

Thư mục này chứa các wheel đã build sẵn từ `robotics-platform`:

- `cubon_robot_base-0.1.0-py3-none-any.whl` — import name: `robot_base`
- `cubon_shared_libs-0.1.0-py3-none-any.whl` — import name: `shared_libs` (phụ thuộc `cubon-robot-base`)

Dùng cho `robotics-arm` và `robotics-humanoid` (không còn dùng Git submodule nữa).

## 1. Yêu cầu

- Python **3.11** (bản build/test chính thức — xem `infrastructure/docker/Dockerfile.base`,
  `Dockerfile.ros2` của `robotics-platform`). Python mới hơn (vd. 3.13/3.14) có thể không có
  wheel nhị phân sẵn cho `numpy<2.0`, sẽ phải build từ source (cần compiler) hoặc lỗi cài đặt.
- `pip` (khuyến nghị nâng lên bản mới nhất: `python -m pip install --upgrade pip`).

## 2. Cài đặt

Chạy trong venv của `robotics-arm` / `robotics-humanoid`, trỏ `--find-links` vào thư mục này
(đường dẫn tuyệt đối hoặc tương đối tùy vị trí checkout của bạn):

```bash
pip install --find-links /duong/dan/toi/robotics-platform/dist cubon-shared-libs
```

`cubon-shared-libs` khai báo phụ thuộc `cubon-robot-base`, nên chỉ cần lệnh trên là pip tự
cài luôn cả hai. Nếu chỉ cần `robot_base` (không cần `shared_libs`):

```bash
pip install --find-links /duong/dan/toi/robotics-platform/dist cubon-robot-base
```

Trên Windows PowerShell:

```powershell
pip install --find-links G:\tsdgo86\Cubon\robotics-platform\dist cubon-shared-libs
```

Trong `Dockerfile` của `robotics-arm`/`robotics-humanoid`, copy thư mục `dist/` này vào build
context rồi cài bằng đường dẫn cục bộ, ví dụ:

```dockerfile
COPY dist/ /tmp/cubon-wheels/
RUN pip install --find-links /tmp/cubon-wheels cubon-shared-libs
```

## 3. Import trong code

Tên gói phân phối có tiền tố `cubon-` (để tránh trùng tên với gói `robot-base` không liên
quan đã có sẵn trên PyPI công khai — dependency confusion), nhưng **tên import không đổi**:

```python
from robot_base.core.container import Container
from robot_base.safety.humanoid_safety_runtime import HumanoidSafetyRuntime
from shared_libs.zmq_ros_bridge import ...
```

## 4. Kiểm tra sau khi cài

```bash
python -c "import robot_base, shared_libs; print(robot_base.__file__); print(shared_libs.__file__)"
```

Đường dẫn in ra phải nằm trong `site-packages` của venv hiện tại — nếu nó trỏ vào source tree
của `robotics-platform` thì có nghĩa `cwd` của bạn đang che (shadow) gói đã cài, không phải
lỗi cài đặt.

## 5. Cập nhật lên bản wheel mới hơn

Khi `robotics-platform` phát hành bản cập nhật:

1. Bên `robotics-platform`: bump `version` trong `robot_base/pyproject.toml` và/hoặc
   `shared_libs/pyproject.toml`, rồi chạy `.\scripts\build_wheels.ps1` để build lại wheel
   vào `dist/`.
2. Copy/đồng bộ thư mục `dist/` mới sang chỗ `robotics-arm`/`robotics-humanoid` dùng.
3. Chạy lại `pip install --upgrade --find-links <dist> cubon-shared-libs` để lên bản mới.

Ghi chú: `dist/` không nằm trong Git của `robotics-platform` (bị `.gitignore`), nên phải copy
thủ công hoặc qua kênh chia sẻ artifact riêng (chưa có private package index cho hai gói này).

## 6. Xử lý lỗi thường gặp

- **`ModuleNotFoundError: No module named 'pydantic'` (hoặc `numpy`, `dependency_injector`,
  `pyzmq`)**: dependency chưa được cài — dùng đúng lệnh `pip install --find-links ... cubon-*`
  (không thêm `--no-deps`) để pip tự kéo theo dependency.
- **Lỗi build `numpy` từ source (thiếu compiler)**: đổi sang Python 3.11 — các bản `numpy<2.0`
  không có wheel nhị phân cho Python 3.13+ trên nhiều nền tảng.
- **Cài nhầm gói `robot-base` (không có tiền tố `cubon-`) từ PyPI công khai**: đó là gói khác
  không liên quan tới platform này — luôn dùng đúng tên `cubon-robot-base` / `cubon-shared-libs`.
