# Đánh giá LLM Judge Phase B

Nhóm: K4-L3-DAY24-TranAnhDang-Eval-Guard

Người phụ trách: Trần Anh Đăng - 2A202602992

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T10:11:18.632352+00:00.
Models generation/judge: generation: openai/gpt-4o-mini; judge: openai/gpt-4o-mini.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ
fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Candidate A | Candidate B | Winner Pass 1 | Winner Pass 2 | Final Winner | Position Consistent | Scores Pass 1 (A/B) | Scores Pass 2 (A/B) | Reasoning |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc khi kết hôn. | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | B | B | B | True | 0.8 / 1.0 | 0.8 / 1.0 | Candidate B nêu rõ tính chất có lương của ngày nghỉ kết hôn, khớp trọn vẹn hơn với chính sách tham chiếu. |
| 5 | Cần Tổng Giám đốc (CEO) phê duyệt. | Cần Giám đốc phòng ban phê duyệt. | A | A | A | True | 1.0 / 0.0 | 1.0 / 0.0 | Candidate A đúng theo chính sách hạn mức mua sắm trên 50 triệu VNĐ, Candidate B chỉ đúng với hạn mức dưới 50 triệu. |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là 1 tháng lương. | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | A | A | A | True | 1.0 / 0.8 | 1.0 / 0.8 | Candidate A cung cấp câu trả lời chuẩn xác và gắn với điều kiện thâm niên từ 6 tháng trở lên như câu hỏi yêu cầu. |
| 21 | Nhân viên có 9 năm thâm niên được 18 ngày phép (15 + 3). Lương của nhân viên này không được đề cập trong tài liệu. | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | B | B | B | True | 0.7 / 1.0 | 0.5 / 1.0 | Candidate B cung cấp đầy đủ cả số ngày phép và khung lương Senior, trong khi Candidate A ảo giác thiếu thông tin lương. |
| 41 | Nhân viên được nghỉ 15 ngày phép năm. | Nhân viên được nghỉ 12 ngày phép năm. | A | A | A | True | 1.0 / 0.0 | 1.0 / 0.2 | Candidate A dựa trên chính sách mới có hiệu lực v2024 (15 ngày), Candidate B dựa trên chính sách cũ v2023 (12 ngày). |

Position bias count/rate: 0 câu / 0.0 (0.0%).
Verbosity win rate: 0.6 (60.0%); decisive cases: 5 / 5 (100.0%).

### Phân tích tính nhất quán và thiên kiến độ dài (Verbosity)

1. **Về Position Bias**: Tỷ lệ position bias đạt 0.0% (0/5 trường hợp bị mâu thuẫn khi đổi thứ tự A/B). Trong cả 5 cặp câu hỏi, LLM Judge đều giữ nguyên phán quyết người thắng cuộc giữa Pass 1 (thứ tự [A, B]) và Pass 2 (thứ tự [B, A] sau khi quy đổi ngược về nhãn gốc). Kỹ thuật swap-and-average đã chứng minh tính hữu hiệu cao trong việc xác nhận tính ổn định của Judge.
2. **Về Verbosity Bias**: Tỷ lệ verbosity win rate là 0.6 (3 trên 5 trường hợp quyết định có câu trả lời dài hơn chiến thắng, cụ thể: A thắng và A dài hơn có 2 trường hợp tại ID 12 và ID 41; B thắng và B dài hơn có 1 trường hợp tại ID 1). Tuy nhiên, con số thống kê 60% không đồng nghĩa với việc LLM Judge thiên vị hình thức câu dài:
   - Điển hình tại **ID 21**, Candidate B (82 ký tự) ngắn hơn Candidate A (122 ký tự), nhưng Judge vẫn chấm B thắng áp đảo (điểm 1.0 so với 0.7 ở Pass 1 và 1.0 so với 0.5 ở Pass 2). Lý do: Candidate A tuy dài hơn nhưng có câu từ chối sai thực tế ("Lương của nhân viên này không được đề cập trong tài liệu"), trong khi B cung cấp đúng khung lương Senior 20-35 triệu.
   - Tại **ID 5**, hai câu có độ dài tương đương (36 ký tự), Judge phân biệt chính xác dựa trên thẩm quyền CEO thay vì Giám đốc phòng ban.
   - Do đó, quyết định của LLM Judge được chi phối bởi tính đầy đủ và tính chính xác ngữ nghĩa (accuracy & completeness) so với reference thay vì thiên vị độ dài thuần túy.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json;
không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference.
Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| Question ID | Model Answer Gốc | Nhãn Judge (0/1) | Reasoning | Human Label | Agree |
|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | 1 | Câu trả lời chính xác theo reference, nêu rõ số ngày nghỉ và điều kiện không trừ vào phép năm. | 1 | True |
| 5 | Cần Giám đốc phòng ban phê duyệt. | 0 | Câu trả lời sai vì yêu cầu phê duyệt cho đơn hàng trên 50 triệu là từ Tổng Giám đốc, không phải Giám đốc phòng ban. | 0 | True |
| 12 | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | 1 | Câu trả lời chính xác theo thông tin trong reference, nêu rõ điều kiện và mức thưởng cho nhân viên chính thức có từ 6 tháng trở lên. | 1 | True |
| 21 | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1 | Câu trả lời chính xác theo reference, nêu rõ số ngày phép và khoảng lương cho nhân viên Senior với 9 năm thâm niên. | 1 | True |
| 23 | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | Câu trả lời chính xác theo reference, nêu rõ rằng nhân viên phải hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | True |
| 29 | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu. | 0 | Câu trả lời sai vì thiếu thẩm quyền Kế toán trưởng và tính sai công thức phí phạt quá hạn 15 ngày. | 0 | True |
| 33 | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng. | 1 | Câu trả lời cung cấp đúng số ngày phép và tổng phụ cấp hàng tháng theo thông tin trong reference. | 1 | True |
| 41 | Nhân viên được nghỉ 12 ngày phép năm. | 0 | Câu trả lời sai vì dựa trên chính sách cũ v2023 thay vì chính sách hiện hành v2024 (15 ngày phép). | 0 | True |
| 46 | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần. | 1 | Câu trả lời chính xác theo reference, nêu rõ nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương. | 1 | True |
| 50 | Được, miễn là đảm bảo kết nối an toàn. | 0 | Câu trả lời sai vì chính sách nghiêm cấm sử dụng VPN cá nhân trong mạng công ty. | 0 | True |

Cohen’s κ: 1.0000.
Observed/chance agreement và diễn giải:
- Tỷ lệ đồng thuận quan sát được (Observed Agreement): $P_o = 10 / 10 = 1.0$ (100%).
- Tỷ lệ đồng thuận ngẫu nhiên kỳ vọng (Chance Agreement): Với phân bố 6 nhãn 1 và 4 nhãn 0 cho cả Judge và Human, $P_e = (6 \times 6 + 4 \times 4) / (10 \times 10) = (36 + 16) / 100 = 0.52$.
- Hệ số Cohen's $\kappa = (P_o - P_e) / (1 - P_e) = (1.0 - 0.52) / (1.0 - 0.52) = 0.48 / 0.48 = 1.0000$. Mức độ tương đồng hoàn hảo (Perfect Agreement).

## Giới hạn và cách dùng judge

1. **Mô hình và Prompt**: Mô hình LLM Judge sử dụng là `openai/gpt-4o-mini` với định dạng đầu ra bắt buộc `json_object`. Prompt hướng dẫn phân định nhị phân và pairwise được tinh chỉnh kỹ lưỡng, nhấn mạnh việc bám sát reference ground truth và phát hiện các bẫy đảo phủ định hoặc chính sách cũ.
2. **Kích thước mẫu và Độ tin cậy**: Mặc dù đạt $\kappa = 1.0$, mẫu hiệu chuẩn chỉ gồm 10 câu hỏi và 5 cặp A/B. Kích thước mẫu này tuy đủ để phát hiện các lỗi định hướng lớn nhưng chưa phản ánh đầy đủ phân phối sai lệch trong môi trường production quy mô lớn.
3. **Nguyên tắc triển khai thực tế**:
   - Không được coi LLM Judge là sự thật tuyệt đối (Ground Truth thay thế con người).
   - Trong pipeline đánh giá tự động (CI/CD hoặc monitoring), LLM Judge nên được dùng làm tầng lọc phát hiện bất thường (anomaly detector / triaging tool) trước khi chuyển các case nghi ngờ cho chuyên viên nhân sự kiểm duyệt thủ công.
   - Luôn duy trì cơ chế swap-and-average định kỳ để kiểm soát rủi ro drift và bias vị trí.
