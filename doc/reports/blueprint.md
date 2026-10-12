# Blueprint CI/CD cho RAG và guardrails

Nhóm: K4-L3-DAY24-TranAnhDang-Eval-Guard

Người phụ trách: Nguyễn Minh Thái — 2A202602726

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-12T12:02:38Z; guard model: gpt-4o-mini.
Models: generation openai/gpt-4o-mini; evaluator BAAI/bge-m3 and
openai/gpt-4o-mini; judge openai/gpt-4o-mini (from Phase A/B metadata).

## Kiến trúc và failure action

Input → Presidio → NeMo input → RAG → NeMo output → response.
Request thực dừng sớm khi bị chặn; khác phép đo input layers chạy đủ hai tầng.

Phase C dùng Presidio với ba loại `VN_CCCD` (CCCD/CMND), `VN_PHONE` và
`EMAIL_ADDRESS`; không dùng nhận diện tên/ngày tháng để chặn câu hỏi policy sạch.
Input chứa PII dừng ở Presidio; nếu không có PII, NeMo chỉ chạy input rail.
Quyết định chặn lấy từ `log.activated_rails[].stop`, không dò từ trong câu trả lời
và không dùng nhãn `expected` hoặc `category` để quyết định. NeMo output rail
kiểm tra answer có sẵn: giữ nguyên answer sạch, dùng response của rail khi bị chặn.
Batch trả lại nguyên input để đối chiếu fixture, vì vậy artifacts có thể chứa PII;
đề xuất production chỉ log loại/vị trí và nội dung đã anonymize, giới hạn quyền
truy cập và thời gian lưu. API keys lấy từ biến môi trường, không ghi vào repo.
Lỗi engine/API được truyền lên runner, không tự cho phép hoặc tạo kết quả giả;
production đề xuất trả lỗi an toàn và không gọi RAG khi guard chưa hoàn tất.
Không có fallback bỏ qua guard. Đây là mô tả code và đề xuất failure action,
chưa phải bằng chứng live; số đo bên dưới cần điền sau khi dependencies sẵn sàng.

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không
short-circuit. Engines tạo trước đo; có hai warm-up thật không tính vào mẫu.
Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2; phương pháp: linear interpolation.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 12.17605 | 40.012915 | 44.585303 | Đo thật từ 20 mẫu |
| NeMo input | 1627.5308 | 2155.85756 | 2640.133512 | Đo thật từ 20 mẫu |
| Input guard sequential | 1637.76465 | 2188.34304 | 2652.481728 | Đo thật; vượt 500ms |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit request | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

Input-layer budget target: 500ms; p95 thực tế 2188.34304ms; budget_ok=false.
Bottleneck là NeMo input (p95 2155.85756ms), do model/API round-trip. RAG,
output rail và full request latency chưa đo; nếu cần production sizing, lấy thêm
20 warm-up-excluded samples cho từng scope với timestamps riêng.

## CI gates đề xuất

- RAGAS faithfulness ≥0.75, avg_score ≥0.65 trên 50 câu.
- Attacks ≥18/20 cho CI đề xuất; clean 8/8 allowed; output cases đúng.
- Full guard P95 <500ms chỉ áp dụng sau khi thực sự đo full guard.

Mốc bắt buộc lab vẫn attacks ≥15/20; bonus ≥18/20 kèm clean controls đạt.
Đề xuất CI step: `python scripts/check_lab.py --with-tests` trong GitHub Actions.
Script tự fail (exit code 1) khi RAGAS <0.75, attacks <18/20 (với bonus gate),
clean false positives >0 hoặc artifacts sai schema. Workflow này là đề xuất CI,
chưa triển khai chạy tự động trong repo.

## Monitoring và cải tiến

Giám sát live: log anonymized inputs/outputs, alert khi attack pass rate <90%,
clean false positive rate >0%, hoặc p95 input latency >2500ms. False positives
được review bởi đội bảo mật/HR policy theo chu kỳ tuần. Bộ 50/20/8/4 là test set
cố định, cần bổ sung truy vấn production thật để tránh overfitting.

RAGAS mean: 0.797385; worst distribution: multi_hop (0.713677).
Cohen’s κ: 1.0.
Attacks passed: 20/20; clean false_positive_rate: 0.0.
Cải tiến đề xuất:
1. Chuyển NeMo input self-check sang local SLM hoặc semantic cache để giảm latency từ ~2188ms xuống dưới 500ms.
2. Cải thiện multi_hop retrieval bằng multi-query/decomposed retrieval để nâng faithfulness vượt 0.85.
3. Mở rộng bộ regex Presidio hỗ trợ thêm mã số thuế cá nhân và số hộ chiếu Việt Nam.
4. Tự động thu thập mẫu truy vấn biên từ logs để mở rộng clean control suite từ 8 lên 50 câu.
