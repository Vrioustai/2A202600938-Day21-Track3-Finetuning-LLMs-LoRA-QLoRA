# Hướng dẫn chi tiết Lab 21 — LoRA/QLoRA Fine-tuning

Bài lab này tập trung vào việc Fine-tune LLM bằng kỹ thuật LoRA/QLoRA 4-bit (sử dụng thư viện Unsloth và TRL). Trọng tâm là thực hiện **Rank Experiment** (thử nghiệm thay đổi thông số rank `r`) để hiểu được sự đánh đổi (trade-off) giữa: thời gian train, dung lượng VRAM và chất lượng mô hình.

## Các bước thực hành chi tiết (2.5 giờ)

### Bước 1: Chuẩn bị Dataset (15 phút)
* Chuẩn bị 100-500 mẫu (samples) chuẩn định dạng Alpaca (`instruction`, `input`, `output`).
* **Tùy chọn:** Bạn có thể chọn dữ liệu tuỳ thích (code, y tế, tóm tắt luật, review, tiếng Việt, v.v.).
* Làm sạch dữ liệu, chia 90% train / 10% eval và tính toán độ dài `max_seq_length` (lấy p95).

### Bước 2 & 3: Train Baseline (Cơ sở) với r=16 (40-50 phút)
* **Cấu hình LoRA:** `r=16`, `lora_alpha=32`, target vào `q_proj` và `v_proj`.
* **Mô hình:** Tùy chọn mô hình vừa với GPU của bạn:
  * T4 Colab: `Qwen2.5-3B`, `Llama-3.2-3B`, `Gemma-2-2b`.
  * L4/A100: Có thể chọn mô hình lớn hơn như `Qwen2.5-7B`, `Llama-3.1-8B`.
* Huấn luyện trong 3 epochs, sử dụng `SFTTrainer`. 
* Vẽ biểu đồ hàm loss xem có bị overfitting không.

### Bước 4: Thử nghiệm Rank (Rank Experiment) - Bắt buộc (30 phút)
* Huấn luyện thêm 2 adapter nữa trên cùng dataset và hyperparameters, **chỉ đổi cấu hình Rank**:
  * Adapter 2: `r=8`, `alpha=16`
  * Adapter 3: `r=64`, `alpha=128`
* Phải ghi nhận lại các chỉ số cho mỗi lần chạy: thời gian train, VRAM tiêu thụ lớn nhất, trainable parameters.

### Bước 5: Đánh giá (Evaluation) (15 phút)
* **Định lượng:** Tính điểm Perplexity (`exp(eval_loss)`) của Base Model, r=8, r=16, r=64.
* **Định tính:** Tạo ít nhất 5 prompt test để so sánh câu trả lời của mô hình gốc (Base) và mô hình đã Fine-tune (của r=16).

---

## Yêu cầu nộp bài (Deliverables)

Bạn phải nộp một file `REPORT.md` (theo mẫu có sẵn) gồm 6 phần bắt buộc:

1. **Setup:** Khai báo mô hình, dataset, cấu hình GPU, ước tính chi phí.
2. **Kết quả Rank Experiment:** Lập bảng so sánh 4 thông số (Train time, Peak VRAM, Perplexity, Trainable params) giữa Base, r=8, r=16, r=64.
3. **Phân tích Loss Curve:** Đính kèm biểu đồ và nhận xét có overfitting hay không.
4. **So sánh định tính:** Viết 5 ví dụ minh họa kết quả trả lời trước và sau fine-tune.
5. **Kết luận về Rank Trade-off (>= 100 từ):** Nhận xét rank nào hiệu quả đầu tư (ROI) tốt nhất, điểm nào thì việc tăng rank không còn tác dụng (diminishing returns), đề xuất rank cho production.
6. **What I Learned:** 2-3 gạch đầu dòng những gì tự rút ra được sau bài lab.

### Các tuỳ chọn nộp bài
* **Option A:** Nộp ZIP nhẹ (~5-15MB) chứa `REPORT.md`, `notebook.ipynb` (đã clear output), adapter `r=16` và thư mục `results/` (các file CSV chứa loss và evaluation).
* **Option B (Khuyên dùng, +5 điểm bonus):** Push adapter lên HuggingFace Hub công khai. Nộp file `LINKS.md` kèm theo `REPORT.md` và file csv.
* **Option C:** Nộp file code thuần tuý.

---

## Điểm Thưởng (Bonus)
* **+5 điểm:** Thực hiện nộp bài theo Option B (HuggingFace Hub).
* **+10 điểm (Stretch goals):**
  * Target ALL layers (q, k, v, o, gate, up, down).
  * Dùng DoRA thay vì LoRA.
  * Merge adapter và convert qua GGUF format để chạy với llama.cpp.
  * Track metrics qua W&B (Weights & Biases).
  * Dùng một bộ dữ liệu domain custom tự tạo hoàn toàn mới.
