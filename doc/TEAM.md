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
| 2 | Thành viên 2 (Đang cập nhật) | TV2-CHO-CAP-NHAT | Phase C (Presidio, NeMo Guardrails, P95 Latency), Blueprint CI/CD | commits, artifacts/guard_results.json, doc/reports/blueprint.md |

Người tích hợp/chạy batch/nộp link: Trần Anh Đăng - 2A202602992, tích hợp 5 modules Day 18, sinh 50 answers và chạy thực nghiệm Phase A/B.

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

### ThanhVien2-TV2-CHO-CAP-NHAT

- Phần được giao và phần thực tế đã làm: Phụ trách Phase C (xây dựng tầng Guardrails với Presidio quét PII và NeMo Guardrails bảo vệ input/output, đo kiểm benchmark độ trễ P95 latency) và Task 13 (viết đề xuất kiến trúc Blueprint CI/CD).
- Bằng chứng: File src/phase_c_guard.py, artifacts/guard_results.json và doc/reports/blueprint.md.
- Cách kiểm tra: Chạy bộ adversarial test suite (20 attacks + 8 clean inputs), đo lường >=20 mẫu latency timed, chạy check_lab.py nghiệm thu.
- Điều đã học: Cách xây dựng kiến trúc guardrails nhiều lớp không phụ thuộc từ khóa; tích hợp Presidio PII với regex Việt Nam và NeMo Guardrails Colang 1.0.
- Giới hạn/cách xử lý: Cần đảm bảo clean inputs không bị chặn nhầm (false-positive rate = 0).
- Nguồn/AI hỗ trợ nếu có: Tự thực hiện và kiểm chứng kết quả theo rubric.

## Cập nhật sau khi nộp

Không có cập nhật sau deadline.
