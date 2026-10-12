# Nhóm thực hiện K4 Track 3B

Tên nhóm: K4-L3-DAY24-TranAnhDang-Eval-Guard

Repo chung: https://github.com/B22DCCN211-TranAnhDang/K4-Track-3B-Day24-Eval-Guard

Trưởng nhóm/người nộp: Trần Anh Đăng - 2A202602992

Ngày lab/deadline áp dụng: 23:59 ngày làm lab — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: HEAD

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Trần Anh Đăng | 2A202602992 | Trưởng nhóm, tích hợp pipeline Day 18, Phase A (RAGAS), Phase B (LLM-as-Judge) | commits, artifacts/ragas_50q.json, artifacts/judge_results.json |
| 2 | Nguyễn Minh Thái | 2A202602726 | Phase C (Presidio, NeMo Guardrails, P95 Latency), Blueprint CI/CD | commits, artifacts/guard_results.json, doc/reports/blueprint.md |

Người tích hợp/chạy batch/nộp link: Trần Anh Đăng - 2A202602992, tích hợp 5 modules Day 18, sinh 50 answers và chạy thực nghiệm Phase A/B. Phase C do Nguyễn Minh Thái - 2A202602726 phụ trách triển khai và kiểm chứng.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11, Python 3.11.4, MSVC C++ Build Tools.
Virtualenv/dependencies/imports: .venv với constraints-core, import ragas, langchain, openai, jsonschema thành công.
Models/Qdrant/50 answers đã chuẩn bị: Docker Qdrant v1.9.7 cổng 6333, BGE-M3 embedding, cross-encoder reranker, answers_50q.json hoàn tất qua OpenRouter gpt-4o-mini.

## INDIVIDUAL

### TranAnhDang-2A202602992

- Phần được giao và phần thực tế đã làm: Tích hợp pipeline Day 18 vào src/, thực hiện Phase A (RAGAS 4 metrics, phân tích bottom 10 và ma trận failure cluster), thực hiện Phase B (Pairwise LLM judge với swap-and-average, tính Cohen's kappa và phân tích position/verbosity bias).
- Bằng chứng: Các file src/phase_a_ragas.py, src/phase_b_judge.py, artifacts/answers_50q.json, artifacts/ragas_50q.json, artifacts/judge_results.json và báo cáo failure_clusters.md, bias_report.md.
- Cách kiểm tra: Đạt 74/74 unit tests offline pytest cho Phase A/B/support/contracts, chạy live batch experiment tạo artifacts JSON hợp lệ.
- Điều đã học: Cách đánh giá độ tin cậy của LLM Judge qua swap-and-average để khử position bias; tính Cohen's kappa để đo tương đồng với nhãn người; phân tích failure cụm của RAG pipeline qua 4 chỉ số RAGAS.
- Giới hạn/cách xử lý: Xử lý rate limit và format JSON object khi gọi LLM judge; chuẩn hóa adapter m4 để lưu metadata.evaluator_models chính xác.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity AI pair programming để rà soát schema và viết unit tests; tự kiểm chứng toàn bộ kết quả qua pytest và check_lab.py.

### NguyenMinhThai-2A202602726

- Phần được giao và phần thực tế đã làm: Phụ trách Phase C (xây dựng tầng Guardrails với Presidio quét PII CCCD/SĐT/Email, cấu hình NeMo Guardrails self-check cho input và output rails, chạy thực nghiệm adversarial suite 28 inputs, đo lường độ trễ P95 latency tuần tự) và Task 13 (hoàn thiện báo cáo đề xuất kiến trúc Blueprint CI/CD).
- Bằng chứng: File src/phase_c_guard.py, guardrails/config.yml, artifacts/guard_results.json và doc/reports/blueprint.md.
- Cách kiểm tra: Chạy bộ adversarial test suite (20/20 attacks blocked, 8/8 clean inputs allowed, false_positive_rate = 0.0), đo 20 mẫu latency timed (P95 input guard sequential = 2188.34ms), chạy kiểm tra check_lab.py --with-tests đạt 86/86 offline unit tests.
- Điều đã học: Cách thiết kế guardrail nhiều lớp kết hợp quy tắc regex PII nhanh và LLM self-check; tinh chỉnh prompt chính sách NeMo để phân biệt câu hỏi chính sách mật khẩu hợp lệ với tấn công đánh cắp mật khẩu, triệt tiêu false positives; kỹ thuật đo độ trễ tuần tự không short-circuit và tính percentile nội suy tuyến tính.
- Giới hạn/cách xử lý: Độ trễ NeMo input qua cloud LLM API vượt budget 500ms (P95 đạt ~2.18s); đã đề xuất giải pháp thay thế bằng local SLM/semantic cache trong blueprint CI/CD.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Copilot SDK in VS Code hỗ trợ rà soát edge cases, sửa config prompt NeMo và kiểm tra contracts.

## Cập nhật sau khi nộp

Không có cập nhật sau deadline.
