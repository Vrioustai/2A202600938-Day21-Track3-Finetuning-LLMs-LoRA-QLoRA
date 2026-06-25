# Lab 21 — Evaluation Report

**Học viên**: <Họ tên của bạn> — <MSSV của bạn>
**Ngày nộp**: <YYYY-MM-DD>
**Submission option**: A (lightweight)

## 1. Setup
- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit`
- **Dataset**: `5CD-AI/Vietnamese-alpaca-gpt4-gg-translated`, 200 samples (180 train + 20 eval)
- **max_seq_length**: 1024 (p95 = 562, rounded up)
- **GPU**: Tesla T4, 16 GB VRAM
- **Training cost**: ~$0.2 (~60 phút @ $0.2/hr trên Google Colab Free)

## 2. Rank Experiment Results

| Rank | Trainable Params | Train Time | Peak VRAM | Eval Loss | Perplexity |
|------|-----------------|------------|-----------|-----------|------------|
| 8    | <Điền số>       | <Điền số> min| <Điền số> GB| <Điền số> | <Điền số>  |
| 16   | <Điền số>       | <Điền số> min| <Điền số> GB| <Điền số> | <Điền số>  |
| 64   | <Điền số>       | <Điền số> min| <Điền số> GB| <Điền số> | <Điền số>  |
| Base | -               | -          | -         | <Điền số> | <Điền số>  |

*(Ghi chú: Điền các số liệu được in ra từ notebook của bạn sau khi chạy xong bước 4 & 5 vào bảng trên)*

## 3. Loss Curve Analysis
*(Chèn hình ảnh loss_curve.png được sinh ra từ thư mục results/loss_curve.png)*
![Loss Curve](results/loss_curve.png)

- **Quan sát:** <Bạn hãy xem đồ thị, nếu đường train loss giảm nhưng eval loss đi ngang hoặc tăng lên tức là model bắt đầu overfit. Nhận xét ngắn gọn tại đây.>

## 4. Qualitative Comparison (5 examples)

### Example 1
**Prompt**: <Copy prompt 1 từ kết quả notebook>
**Base**: <Copy câu trả lời của base model>
**Fine-tuned (r=16)**: <Copy câu trả lời của model r=16>
**Nhận xét**: <Model nào trả lời đúng context và format hơn? Có bị lặp từ hay sai kiến thức không?>

### Example 2
**Prompt**: <Copy prompt 2>
**Base**: <Copy câu trả lời của base model>
**Fine-tuned (r=16)**: <Copy câu trả lời của model r=16>
**Nhận xét**: <Nhận xét ngắn gọn>

### Example 3
**Prompt**: <Copy prompt 3>
**Base**: <Copy câu trả lời của base model>
**Fine-tuned (r=16)**: <Copy câu trả lời của model r=16>
**Nhận xét**: <Nhận xét ngắn gọn>

### Example 4
**Prompt**: <Copy prompt 4>
**Base**: <Copy câu trả lời của base model>
**Fine-tuned (r=16)**: <Copy câu trả lời của model r=16>
**Nhận xét**: <Nhận xét ngắn gọn>

### Example 5
**Prompt**: <Copy prompt 5>
**Base**: <Copy câu trả lời của base model>
**Fine-tuned (r=16)**: <Copy câu trả lời của model r=16>
**Nhận xét**: <Nhận xét ngắn gọn>

## 5. Conclusion về Rank Trade-off

- **Rank nào cho ROI tốt nhất trên dataset này? Tại sao?**
  <Trả lời: Thường r=16 sẽ cho ROI tốt nhất vì cân bằng giữa thời gian train, VRAM tiêu thụ và điểm perplexity giảm đáng kể so với r=8.>
- **Khi nào tăng rank không còn cải thiện perplexity (diminishing returns)?**
  <Trả lời: Dựa vào bảng trên, nếu r=64 có perplexity không khác biệt nhiều so với r=16 (hoặc thậm chí cao hơn do overfit) nhưng tốn gấp đôi dung lượng adapter và thời gian, đó là lúc bị diminishing returns.>
- **Recommendation: nếu deploy production, bạn chọn rank nào? Tại sao?**
  <Trả lời: Tôi sẽ chọn rank 16. Vì nó đủ sức để học format/style mới, sinh ra output chất lượng tốt nhưng file adapter vẫn nhỏ gọn, tốn ít VRAM để serve nhiều adapter cùng lúc (multi-tenant).>

## 6. What I Learned
- Tầm quan trọng của việc kiểm tra max_seq_length (p95) để tránh padding rác làm tốn VRAM và kéo dài thời gian huấn luyện.
- Cơ chế của LoRA: chỉ huấn luyện một phần rất nhỏ tham số, giúp tiết kiệm chi phí cực lớn mà vẫn đạt hiệu quả gần bằng Full Fine-Tuning.
- Hiểu được sự đánh đổi (trade-off) khi tinh chỉnh cấu hình rank `r`. Không phải lúc nào rank càng to thì kết quả càng tốt, đặc biệt trên dataset nhỏ (200 samples) rất dễ bị overfit.
