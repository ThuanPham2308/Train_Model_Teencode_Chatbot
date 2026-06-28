# Pipeline phát hiện Teencode tiếng Việt (PhoBERT / ViSoBERT / FastText)

3 notebook độc lập, cùng giải quyết 1 bài toán — phát hiện teencode trong câu
tiếng Việt — bằng 3 cách khác nhau:

| Notebook | Phương pháp |
|---|---|
| `phobert-v8__3_.ipynb` | PhoBERT + token classification (BIO tagging) |
| `visobert-v9__4_.ipynb` | ViSoBERT + token classification (BIO tagging) |
| `fasttext-v7__5_.ipynb` | Dict lookup + FastText classifier (Hybrid Detection) |

## Cài đặt

```bash
pip install -r requirements.txt
cp .env.example .env   # rồi sửa lại đường dẫn dữ liệu cho đúng máy bạn
```

`.env` không được commit lên Git (đã khai báo trong `.gitignore`). Nếu bạn
không tạo `.env` — ví dụ khi chạy trực tiếp trên Kaggle — cả 3 notebook vẫn
chạy bình thường, vì mọi biến đều có giá trị mặc định là đường dẫn
`/kaggle/input/...` như notebook gốc.

Các biến có thể override qua `.env`, xem chi tiết trong `.env.example`:
`TEENCODE_PATH`, `CFS_PATH`, `DICT_PATH`, `RANDOM_SEED`, `CFS_SAMPLE_SIZE`,
`MODEL_DIR`, `OUTPUT_DIR`, `MODEL_DIR_FT`, `OUTPUT_DIR_FT`,
`RUN_FULL_DATASET`, `SAMPLE_SIZE`.

## Những gì đã được dọn lại trong lần cập nhật này

1. **Tách config ra `.env`** — Mọi đường dẫn dữ liệu (`Dataset_Teencode.xlsx`,
   `Dataset_CFS.xlsx`) và một vài tham số chạy (`RANDOM_SEED`,
   `RUN_FULL_DATASET`, `SAMPLE_SIZE`, `MODEL_DIR`, `OUTPUT_DIR`, ...) trước
   đây bị hard-code thẳng trong notebook (đường dẫn Kaggle cụ thể của 1
   máy). Giờ đọc qua `os.getenv(...)` với `python-dotenv`, vẫn giữ nguyên
   giá trị mặc định cũ để không phá vỡ workflow Kaggle hiện tại. → 3 file
   `.ipynb` giờ không còn chứa thông tin riêng của máy/tài khoản nào, có thể
   push lên Git an toàn.
2. **Sửa 1 bug copy-paste**: notebook ViSoBERT (`visobert-v9__4_.ipynb`) lưu
   model vào thư mục `models/phobert_teencode_detector` (sao chép nhầm từ
   notebook PhoBERT) — đã sửa thành `models/visobert_teencode_detector`.
3. **Bỏ code trùng lặp**: `normalize_phrase()` và `partial_match_score()` bị
   định nghĩa lại 2 lần trong cả `phobert` và `visobert` (1 lần ở cell dò
   ngưỡng trên validation set, 1 lần ở cell đánh giá trên test set). Đã gộp
   về 1 định nghĩa duy nhất (cell chạy trước), cell sau dùng lại.
4. **Sửa 2 cell markdown bị hỏng**: cell tiêu đề đầu và cell tổng kết cuối
   trong notebook FastText bị một công cụ nào đó tách lưu từng *ký tự*
   thành 1 phần tử riêng trong JSON (`source` có ~1700 phần tử thay vì vài
   chục dòng bình thường) — về mặt hiển thị thì Markdown vẫn gộp lại đúng,
   nhưng file rất nặng và gần như không thể diff/review trên Git. Đã build
   lại thành text nhiều-dòng bình thường, nội dung giữ nguyên 100% (đã đối
   chiếu lại từng câu).
5. **`requirements.txt` + `.gitignore`**: gộp toàn bộ thư viện cần cho cả 3
   notebook vào 1 file `requirements.txt`; `.gitignore` loại trừ `.env`,
   các file dữ liệu/model lớn (`*.xlsx`, `models/`, `outputs/`, `*.bin`,
   `*.vec`) và rác Jupyter/Python thường gặp.

### Về phần "clean text"

Hàm làm sạch văn bản (`clean_text` trong PhoBERT/ViSoBERT, `preprocess` +
`normalize_token` trong FastText) đã được rà lại — logic hiện tại (chuẩn hóa
Unicode NFC, lowercase, bỏ URL/emoji, rút gọn ký tự lặp, chuẩn hóa khoảng
trắng) hợp lý và nhất quán giữa các notebook, nên không có thay đổi nào ở
phần logic xử lý text. Nếu bạn có yêu cầu cụ thể hơn (vd. thêm rule xử lý 1
loại ký tự đặc biệt nào đó, hoặc thấy `clean_text` xử lý sai 1 trường hợp
thực tế), gửi ví dụ cụ thể để mình chỉnh tiếp.

## Không nên đẩy lên Git

Đã liệt kê trong `.gitignore`: file `.env`, dữ liệu `*.xlsx`, thư mục
`models/`, `outputs/`, file model `*.bin`/`*.vec`. Những file này nặng và/hoặc
chứa dữ liệu nội bộ — nên lưu riêng (Kaggle Datasets, Google Drive, Git LFS,
...) chứ không nên commit thẳng vào repo code.
# Train_Model_Teencode_Chatbot
