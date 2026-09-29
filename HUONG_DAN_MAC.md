# Hướng dẫn dùng StringTheory trên Mac (M1 / Apple Silicon)

Tài liệu này tổng hợp những gì cần biết để chạy StringTheory trên Mac chip M (M1, M2, M3…) và dùng nó tập guitar điện ở nhà với card âm thanh riêng.

Tài liệu kỹ thuật gốc (tiếng Anh): [HOW_TO_BUILD_MAC.md](HOW_TO_BUILD_MAC.md), [HOW_TO_BUILD.md](HOW_TO_BUILD.md).

---

## 1. Dự án này là gì

StringTheory là một game/công cụ tập đàn làm bằng **Unity 6 (6000.2.9f1)**, giống Rocksmith:

- Cắm guitar/bass thật qua card âm thanh; game **nghe và chấm điểm** từng nốt và hợp âm theo thời gian thực (model AI Basic Pitch, chạy bằng ONNX Runtime).
- Các chế độ tập:
  - **Guitar Mode**: chơi cả bài, có điểm.
  - **Loop Mode**: chọn một đoạn khó, lặp đến khi sạch; lưu bookmark các đoạn.
  - **Note By Note**: dừng ở từng nốt cho tới khi bạn đánh đúng.
  - **Hero Mode**: có "mạng", sai nhiều là thua.
  - **Chỉnh tốc độ**: chơi chậm lại những đoạn khó.
- **Tone Lab**: giả lập ampli và pedal (hỗ trợ NAM và LV2), để nghe tiếng guitar điện có hiệu ứng khi tập qua tai nghe hoặc loa.
- **Chart Editor**: tự tạo bài từ file tab và file nhạc. Có **SynchTheory** tự khớp tab với nhạc, và **tách stem** (Demucs) để bỏ tiếng guitar gốc khỏi bài.
- Hiển thị kiểu **Highway 3D** hoặc **Tab**.

---

## 2. Trạng thái hỗ trợ Mac

| Hạng mục | Trạng thái |
|---|---|
| Code C# có nhánh xử lý riêng cho macOS | ✅ Có |
| Âm thanh qua **CoreAudio** (ưu tiên ngang ASIO trên Windows) | ✅ Có trong code |
| Xin quyền microphone trên macOS | ✅ Có |
| Script build thư viện native cho Mac (CMake) | ✅ `Tools/Mac/build-macos-native-plugins.sh` |
| Workflow GitHub Actions build bản Mac (Intel + Apple Silicon) | ✅ `.github/workflows/macos-build.yml` |
| Hộp thoại chọn **thư mục** trên Mac | ✅ Dùng `osascript` |
| Hộp thoại chọn **file** trong Chart Editor trên Mac | ✅ **Đã sửa** (trước đây không làm gì trên Mac) |
| Tự cài bộ tách stem qua mạng | ⚠️ Chỉ Windows. Trên Mac phải dùng gói cài sẵn (xem mục 6) |
| Đã kiểm tra trên Mac thật | ❌ **Chưa**. Tác giả ghi rõ phần âm thanh, nhận nốt và Tone Lab cần kiểm tra trên Mac thật |
| Thư viện native `.dylib` có sẵn trong repo | ❌ Không. Phải tự build trên Mac (mục 4) |

### Thay đổi đã làm trong repo này

- `Assets/Platform/StringTheoryPlatform.cs`: thêm `TryPickMacFile(...)`, mở hộp thoại chọn file gốc của macOS qua `osascript` (`choose file of type {...}`), có lọc theo đuôi file.
- `Assets/ChartEditor/ChartEditorFilePicker.cs`:
  - Chọn file: thêm nhánh `UNITY_STANDALONE_OSX` gọi `TryPickMacFile`. Trước đây chỉ ghi log *"File picker is not implemented for this platform"*.
  - Chọn thư mục: gọi `StringTheoryPlatform.TryPickFolder` (có hỗ trợ Mac) thay vì `WindowsFolderPicker` (trên Mac luôn trả về `false`).

> Những thay đổi này chưa được biên dịch và chạy thử trên Mac. Lần build đầu hãy kiểm tra: mở Chart Editor, bấm chọn file tab / file nhạc, xem hộp thoại có hiện ra và có lọc đúng loại file không.

---

## 3. Card âm thanh

- Trên Mac, hầu hết card phổ biến (Focusrite Scarlett, Audient iD/EVO, Behringer UMC, Steinberg UR, Zoom, M-Audio…) là **class-compliant**: cắm vào là chạy với CoreAudio, **không cần driver**.
- App dùng **PortAudio** và tự ưu tiên thiết bị CoreAudio.
- Cách chỉnh:
  1. Cắm guitar vào cổng **Hi-Z / Instrument** trên card. Bật nút "INST" nếu card có.
  2. Trong app, mở **Tone Lab**, chọn input và output là card của bạn.
  3. Chỉnh **buffer size / latency**: bắt đầu ở 128 samples. Nếu nghe lẹt xẹt thì tăng lên 256. Sample rate để Auto, 44100 hoặc 48000.
  4. Tắt monitor trực tiếp (direct monitor) trên card nếu nghe tiếng đàn bị đôi (tiếng khô cộng tiếng đã qua hiệu ứng).
- Lần đầu mở app, macOS sẽ hỏi quyền **Microphone**, hãy bấm **Allow**. Nếu lỡ từ chối, vào *System Settings → Privacy & Security → Microphone* và bật cho StringTheory.

---

## 4. Build trên Mac M1

### Bước 0: Kiểm tra bản build sẵn

Trước tiên xem trang **Releases** của dự án gốc có bản macOS build sẵn không. Nếu có thì tải về dùng luôn, không cần làm các bước dưới.

### Bước 1: Cài công cụ

```bash
# Xcode Command Line Tools
xcode-select --install

# Homebrew (nếu chưa có): https://brew.sh
brew install cmake git git-lfs
brew install --cask dotnet-sdk        # .NET SDK (9.x)
```

- Cài **Unity Hub**, rồi cài Unity **6000.2.9f1** (Apple Silicon), nhớ tick module **Mac Build Support**.

### Bước 2: Clone repo

```bash
git clone https://github.com/linkhoang19931-hub/guitar_train.git
cd guitar_train
git lfs pull
```

### Bước 3: Build thư viện native (chỉ cần arm64 cho M1)

```bash
MACOS_ARCHS=arm64 bash Tools/Mac/build-macos-native-plugins.sh
```

Script tự tải aubio, libsamplerate, PortAudio (có CoreAudio) và ONNX Runtime 1.19.2, rồi build và copy vào `Assets/Plugins/macOS/`:

- `libNativeNotesDetectorBridgeNative_v6.dylib`: nhận diện nốt
- `libStringTheoryToneHost.dylib`: Tone Lab (NAM/LV2)
- `libportaudio.dylib`: âm thanh CoreAudio
- `libonnxruntime.dylib`: chạy model AI

### Bước 4: Model nhận diện nốt (Basic Pitch)

```bash
python3 -m pip install --no-deps --target /tmp/bp basic-pitch==0.4.0
MODEL=$(find /tmp/bp \( -name 'basic_pitch_nmp.onnx' -o -name 'nmp.onnx' \) | head -n 1)
mkdir -p Assets/StreamingAssets/NotesReader/Models
cp "$MODEL" Assets/StreamingAssets/NotesReader/Models/basic_pitch_nmp.onnx
```

### Bước 5: Các DLL .NET đọc file Guitar Pro

Tạo nhanh một project tạm để NuGet tải về:

```bash
mkdir -p /tmp/deps && cd /tmp/deps
cat > deps.csproj <<'EOF'
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup><TargetFramework>netstandard2.0</TargetFramework></PropertyGroup>
  <ItemGroup>
    <PackageReference Include="AlphaTab" Version="1.6.0-alpha.1444" />
    <PackageReference Include="AlphaSkia" Version="3.3.135" />
    <PackageReference Include="System.Drawing.Common" Version="9.0.5" />
    <PackageReference Include="Microsoft.Win32.SystemEvents" Version="9.0.5" />
  </ItemGroup>
</Project>
EOF
dotnet restore /p:NuGetAudit=false
cd -   # quay lại thư mục repo
mkdir -p Assets/Plugins/Managed
N=~/.nuget/packages
cp $N/alphatab/1.6.0-alpha.1444/lib/netstandard2.0/AlphaTab.dll Assets/Plugins/Managed/
cp $N/alphaskia/3.3.135/lib/netstandard2.0/AlphaSkia.dll Assets/Plugins/Managed/
cp $N/system.drawing.common/9.0.5/lib/netstandard2.0/System.Drawing.Common.dll Assets/Plugins/Managed/
cp $N/microsoft.win32.systemevents/9.0.5/lib/netstandard2.0/Microsoft.Win32.SystemEvents.dll Assets/Plugins/Managed/
```

### Bước 6: Hiển thị Tab (tùy chọn, để dùng chế độ "Tabs (AlphaTab)")

```bash
dotnet publish Tools/AlphaTabRenderHelper/AlphaTabRenderHelper.csproj \
  -c Release -r osx-arm64 --self-contained true \
  -o Assets/StreamingAssets/AlphaTabRenderHelper/osx-arm64
chmod +x Assets/StreamingAssets/AlphaTabRenderHelper/osx-arm64/AlphaTabRenderHelper
```

### Bước 7: Gói MIDI Maestro (bắt buộc)

1. Mở project bằng Unity 6000.2.9f1 qua Unity Hub. Nếu báo lỗi compile lúc đầu thì cứ bỏ qua.
2. `Window → Package Manager → My Assets`, tìm **Maestro - Midi Player Tool Kit - Free** (miễn phí trên Asset Store), bấm Download rồi Import.
3. Kiểm tra đã có thư mục `Assets/MidiPlayer/`.

### Bước 8: Build app

1. `File → Build Profiles`, chọn **macOS**, Architecture chọn **Apple Silicon**.
2. Bấm **Build**, lưu ra ví dụ `build/StringTheory.app`.
3. Lần đầu mở app tự build, macOS có thể chặn. Chuột phải vào app, chọn **Open**, hoặc chạy:
   ```bash
   xattr -dr com.apple.quarantine build/StringTheory.app
   ```

> Cách khác: build trên GitHub Actions (`Actions → macOS build → Run workflow`). Cách này cần secrets license Unity (`UNITY_LICENSE`, `UNITY_EMAIL`, `UNITY_PASSWORD`) và các asset riêng của tác giả, nên với dùng cá nhân thì **build trực tiếp trên Mac dễ hơn**.

---

## 5. Nạp bài bất kỳ để tập

### Điều quan trọng cần biết

App **không tự chuyển file mp3 thành tab**. Mỗi bài cần:

1. **Tab/bản nhạc** (bắt buộc): `.gp`, `.gp3`, `.gp4`, `.gp5`, `.gp8`, `.gpx` (Guitar Pro), `.musicxml` / `.xml`, hoặc `.psarc` (file bài của Rocksmith; cần cài add-on importer riêng).
2. **File nhạc** (nên có): `.mp3`, `.wav`, `.ogg`, `.flac`, `.m4a`, `.aiff`.

### Tìm tab ở đâu

- Bài phổ biến: tìm file Guitar Pro trên các trang tab (Ultimate Guitar "Guitar Pro" tabs, Songsterr…).
- Bài không có tab: tự viết bằng **TuxGuitar** (miễn phí), **MuseScore** (miễn phí) hoặc Guitar Pro, rồi xuất ra `.gp5`/`.gp`/MusicXML.

### Cách 1: Thả thư mục bài (nhanh nhất)

Thư mục bài hát trên Mac nằm ở:

```
~/Library/Application Support/StringTheory/StringTheory/Songs/
```

Bạn cũng có thể bấm nút mở thư mục Songs ngay trong màn hình Library của app.

1. Tạo một thư mục con cho mỗi bài, ví dụ `Songs/November Rain/`.
2. Bỏ file tab (`.gp5`…) và file nhạc (`.mp3`…) vào đó.
3. (Tùy chọn) thêm `song.json`:
   ```json
   {
     "songId": "november-rain",
     "displayName": "November Rain",
     "artist": "Guns N' Roses",
     "difficulty": 4
   }
   ```
   `difficulty`: 1 Beginner, 2 Novice, 3 Standard, 4 Advanced, 5 Master.
4. Trong app, bấm **Refresh** thư viện.

### Cách 2: Chart Editor (khi tab lệch nhịp so với nhạc)

1. Mở **Chart Editor**, chọn file tab, rồi chọn file nhạc.
2. Chạy **SynchTheory** để tự căn tab khớp với nhạc thật.
3. Chỉnh tay những chỗ còn lệch (tempo, offset).
4. (Tùy chọn) **Tách stem** để bỏ tiếng guitar gốc, chỉ còn band đệm.
5. Xuất ra file `.theory`, bài sẽ xuất hiện trong thư viện.

### Tập hiệu quả

- Chọn track Lead / Rhythm / Bass tùy phần muốn tập.
- Đoạn khó: bật **Loop Mode** và giảm **tốc độ** xuống 60–70%, rồi tăng dần.
- Mới học bài: dùng **Note By Note**.
- Nếu game chấm lệch nhịp, chỉnh **timing offset** theo track hoặc theo cả bài.

---

## 6. Hạn chế đã biết trên Mac

- **Chưa kiểm tra thực tế trên Mac**: có thể gặp lỗi ở phần âm thanh, nhận nốt hoặc Tone Lab. Khi gặp lỗi, xem log tại `~/Library/Logs/StringTheory/StringTheory/Player.log`.
- **Tách stem**: bộ cài online chỉ chạy trên Windows. Trên Mac cần gói `stem-separator-runtime-macos-universal.zip` đặt trong `Assets/StreamingAssets/StemSeparator/`. Có thể tự build bằng:
  ```bash
  bash Tools/Mac/build-macos-stem-runtime.sh osx-arm64 Temp/stem-runtime
  ```
  rồi nén thư mục `osx-arm64` thành zip với tên trên.
- **Plugin LV2** phải là bản build cho macOS (các file `.dll` của Windows sẽ bị bỏ qua). Đặt trong `Assets/StreamingAssets/ToneLab/LV2/`.
- **Importer add-on** (ví dụ `.psarc`) chỉ chạy trên Mac nếu add-on đó có kèm bản build cho macOS.

---

## 7. Checklist lần chạy đầu trên Mac

- [ ] App mở được, có hỏi quyền Microphone
- [ ] Tone Lab thấy card âm thanh (CoreAudio)
- [ ] Nghe được tiếng đàn qua tai nghe/loa, độ trễ chấp nhận được
- [ ] Tuner nhận đúng dây
- [ ] Chơi một bài: nốt đánh đúng được chấm điểm
- [ ] Chart Editor: hộp thoại chọn file tab và file nhạc mở được
- [ ] Thêm bài mới vào `Songs/`, bấm Refresh là thấy bài
