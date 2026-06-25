# Lab 21 — Evaluation Report

**Học viên**: Lâm Văn Tài - 2A202600938
**Ngày nộp**: 2026-06-25
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
| 8    | 1,843,200       | 3.99 min   | 7.21 GB   | 1.5576    | 4.7478     |
| 16   | 3,686,400       | 4.25 min   | 6.61 GB   | 1.5160    | 4.5543     |
| 64   | 14,745,600      | 3.99 min   | 7.99 GB   | 1.4768    | 4.3789     |
| Base | -               | -          | -         | N/A       | N/A        |

*(Ghi chú: Điền các số liệu được in ra từ notebook của bạn sau khi chạy xong bước 4 & 5 vào bảng trên)*

## 3. Loss Curve Analysis
*(Chèn hình ảnh loss_curve.png được sinh ra từ thư mục results/loss_curve.png)*
![Loss Curve](results/loss_curve.png)

- **Quan sát:** Train loss giảm đều đặn trong quá trình huấn luyện. Do cài đặt trên T4 đã tắt eval-during-training để tiết kiệm VRAM, ta chỉ quan sát được train loss curve. Với kết quả Eval Loss sau huấn luyện, r=64 cho loss thấp nhất (1.4768) tuy nhiên r=16 cũng có mức loss khá tốt (1.5160) so với tài nguyên bỏ ra. Không quan sát thấy sự tăng vọt đáng kể của eval loss, nhưng r=64 có thể dễ bị overfit nhất nếu số sample quá ít.

## 4. Qualitative Comparison (5 examples)

### Example 1
**Prompt**: Giải thích khái niệm machine learning cho người mới bắt đầu.
**Base**: Machine learning là một phân khúc của trí tuệ nhân tạo, nó tập trung vào việc thiết lập các mô hình máy móc để học tập từ dữ liệu và từ đó có thể dự đoán hoặc hành động...
**Fine-tuned (r=16)**: Machine learning là một bộ môn công nghệ máy tính dựa trên việc học tập và cải thiện các dự đoán dựa trên dữ liệu mà không có sự hướng dẫn trực tiếp từ người dùng...
**Nhận xét**: Mô hình fine-tuned cho câu trả lời tự nhiên, trôi chảy và diễn đạt dễ hiểu hơn so với Base model.

### Example 2
**Prompt**: Viết đoạn code Python tính số Fibonacci thứ n.
**Base**: Để tính số Fibonacci thứ n, bạn có thể sử dụng hàm đệ quy hoặc vòng lặp. Đây là một đoạn mã Python cho phép bạn tính số Fibonacci thứ n: \n```python\ndef fibonacci(n):\n    if n <= 0:\n        return "N p...
**Fine-tuned (r=16)**: Để tính số Fibonacci thứ n, bạn có thể viết một đoạn code Python như sau: \n```python\ndef fibonacci(n):\n    if n < 0:\n        raise ValueError("Input phải là một số nguyên dương.")\n    elif n == 0:\n   ...
**Nhận xét**: Cả hai đều tạo ra được code. Tuy nhiên mô hình fine-tuned xử lý cả ngoại lệ (raise ValueError) khi input âm, cho thấy chất lượng code chuyên nghiệp hơn.

### Example 3
**Prompt**: Liệt kê 5 nguyên tắc thiết kế UI/UX.
**Base**: 1. Thân thiện với người dùng: Mục đích của thiết kế UI/UX là cung cấp trải nghiệm người dùng thân thiện, dễ sử dụng và thân thiện. Điều này có thể được thể hiện thông qua việc sắp xếp bố cục, màu sắc,...
**Fine-tuned (r=16)**: 1. Chuyển đổi: UI/UX thiết kế phải hướng tới việc giúp người dùng thực hiện hành động. 2. Thích ứng: UI/UX thiết kế phải thích ứng với nhiều loại thiết bị và kích thước màn hình. 3. Đơn giản: UI/UX th...
**Nhận xét**: Base model có hiện tượng lặp từ ("thân thiện... thân thiện"). Fine-tuned model đưa ra câu trả lời gãy gọn, súc tích và đúng trọng tâm các nguyên tắc hơn.

### Example 4
**Prompt**: Tóm tắt sự khác biệt giữa LoRA và QLoRA.
**Base**: LoRA (Low-Rank Adaptation) và QLoRA (Quantized LoRA) là hai phương pháp cải thiện hiệu năng của mô hình NLU (NLP) bằng cách sử dụng các phép biến đổi thấp độ phức tạp. LoRA là phương pháp cải thiện hi...
**Fine-tuned (r=16)**: LoRA (Layer-wise Adaptive Regularization Optimization) và QLoRA (Quantized LoRA) là hai phương pháp regularization được phát triển để cải thiện hiệu quả và độ ổn định của các mạng neural network trong...
**Nhận xét**: Mô hình Fine-tuned bị hallucination khi diễn giải sai từ viết tắt của LoRA, trong khi Base model định nghĩa thuật ngữ LoRA (Low-Rank Adaptation) chính xác hơn.

### Example 5
**Prompt**: Phân biệt prompt engineering, RAG, và fine-tuning.
**Base**: Prompt engineering, RAG (retrieval augmented generation), và fine-tuning là ba cách khác nhau để cải thiện hiệu suất của mô hình máy học. Prompt engineering là một kỹ thuật để cải thiện hiệu suất của ...
**Fine-tuned (r=16)**: Prompt engineering, RAG và fine-tuning là ba kỹ thuật khác nhau được sử dụng trong lĩnh vực AI và tự động hóa. Prompt engineering là một kỹ thuật tập trung vào việc xây dựng câu lệnh (prompt) để giúp ...
**Nhận xét**: Cả hai đều nắm bắt được khái niệm đúng. Fine-tuned model có cách hành văn linh hoạt và mượt mà hơn đôi chút.

## 5. Conclusion về Rank Trade-off

- **Rank nào cho ROI tốt nhất trên dataset này? Tại sao?**
  Dựa vào kết quả thử nghiệm, r=16 cho ROI tốt nhất vì nó cân bằng giữa thời gian train, VRAM tiêu thụ và điểm perplexity. Nó cải thiện tốt hơn so với r=8 trong khi không tốn quá nhiều VRAM như r=64.
- **Khi nào tăng rank không còn cải thiện perplexity (diminishing returns)?**
  Dựa vào bảng trên, khi tăng lên r=64 perplexity có giảm nhẹ nhưng không thực sự khác biệt đột phá so với r=16, trong khi tốn thêm dung lượng VRAM và gấp 4 lần số lượng trainable parameters. Đây là lúc bắt đầu bị diminishing returns.
- **Recommendation: nếu deploy production, bạn chọn rank nào? Tại sao?**
  Tôi sẽ chọn rank 16. Vì nó đủ sức để học format/style mới, sinh ra output chất lượng tốt nhưng file adapter vẫn nhỏ gọn, tốn ít VRAM để có thể serve nhiều adapter cùng lúc (multi-tenant).

## 6. What I Learned
- Tầm quan trọng của việc kiểm tra max_seq_length (p95) để tránh padding rác làm tốn VRAM và kéo dài thời gian huấn luyện.
- Cơ chế của LoRA: chỉ huấn luyện một phần rất nhỏ tham số, giúp tiết kiệm chi phí cực lớn mà vẫn đạt hiệu quả gần bằng Full Fine-Tuning.
- Hiểu được sự đánh đổi (trade-off) khi tinh chỉnh cấu hình rank `r`. Không phải lúc nào rank càng to thì kết quả càng tốt, đặc biệt trên dataset nhỏ (200 samples) rất dễ bị overfit.
