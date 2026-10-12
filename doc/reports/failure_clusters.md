# Phân tích lỗi Phase A

Nhóm: K4-L3-DAY24-TranAnhDang-Eval-Guard

Người phụ trách: Trần Anh Đăng - 2A202602992

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-12T10:10:09.882720+00:00.
Generation mode/fallback và điều kiện chạy: Chạy trực tiếp (`run_kind: live`), 100% câu hỏi sinh bằng LLM (`generation_mode: llm`), không kích hoạt extractive fallback. Sử dụng Qdrant v1.9.7, mô hình BGE-M3 dense retrieval kết hợp cross-encoder reranker m3.
Models generation/evaluator: generation: openai/gpt-4o-mini; evaluator: BAAI/bge-m3 và openai/gpt-4o-mini (được trích xuất và báo cáo chính xác từ m4_eval).

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.9083 | 0.4677 | 0.8000 |
| answer_relevancy | 0.7922 | 0.6078 | 0.5873 |
| context_precision | 0.9667 | 0.9667 | 0.9833 |
| context_recall | 0.9250 | 0.8125 | 0.6833 |
| avg_score | 0.8980 | 0.7137 | 0.7635 |

## Bottom 10 và nguyên nhân

| Rank | Question ID | Question | Distribution | Avg Score | Worst Metric | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|---|
| 1 | 39 | So sánh yêu cầu mật khẩu giữa policy v1.0 và v2.0 về độ dài tối thiểu, thời hạn đổi và MFA. | multi_hop | 0.1250 | faithfulness | LLM nhầm lẫn giữa 2 phiên bản policy v1.0 và v2.0 do context chứa cả 2 văn bản nhưng không phân tách rõ ràng mốc thời gian hiệu lực. | Bổ sung metadata version tagging vào chunking và thêm chỉ dẫn phân biệt phiên bản chính sách vào system prompt. |
| 2 | 44 | Bao lâu phải đổi mật khẩu một lần? | adversarial | 0.3750 | faithfulness | Bẫy xung đột phiên bản: v1.0 yêu cầu đổi định kỳ 90 ngày, trong khi v2.0 đã bãi bỏ quy định này. LLM vẫn trích dẫn thông tin 90 ngày từ v1.0. | Thiết lập bộ lọc thời gian hiệu lực (metadata filter) trong khâu retrieval để ưu tiên policy đang có hiệu lực mới nhất. |
| 3 | 7 | Nghỉ phép không lương 20 ngày cần ai phê duyệt? | factual | 0.3750 | faithfulness | LLM trả lời "Không tìm thấy" do context bị phân mảnh giữa chính sách nghỉ phép năm và nghỉ không lương, dẫn đến ảo giác từ chối trả lời. | Cải thiện chunking và semantic routing để gom đúng tài liệu nghi_phep_khong_luong.md, giảm abstention threshold. |
| 4 | 33 | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm theo v2024 là bao nhiêu? | multi_hop | 0.3750 | faithfulness | Tổng hợp đa nguồn thất bại: LLM tính đúng số ngày phép năm nhưng bỏ sót khoản phụ cấp điện thoại 500k dành riêng cho cấp Manager. | Bổ sung kỹ thuật Multi-query Decomposition và Chain-of-Thought để bóc tách từng phần câu hỏi con độc lập. |
| 5 | 50 | Nhân viên Manager có thể dùng VPN cá nhân (như NordVPN) khi WFH để tăng bảo mật thêm không? | adversarial | 0.4167 | faithfulness | Bẫy câu hỏi dẫn dắt (leading question): LLM bị thiên kiến đồng thuận, không phát hiện điều khoản cấm tuyệt đối VPN cá nhân trong mạng công ty. | Thêm nguyên tắc Negative Constraint Checking vào prompt nhằm phát hiện và từ chối các hành vi bị chính sách nghiêm cấm. |
| 6 | 30 | So sánh quyền lợi bảo hiểm giữa nhân viên thử việc và nhân viên chính thức. | multi_hop | 0.5750 | answer_relevancy | Trả lời lan man sang chi tiết gói gia đình thay vì tập trung vào bảng so sánh điều kiện thụ hưởng giữa 2 đối tượng nhân viên. | Chuẩn hóa định dạng câu trả lời theo khuôn mẫu bảng so sánh có cấu trúc rõ ràng. |
| 7 | 22 | Nếu cần mua một chiếc laptop 30 triệu cho nhân viên mới, ai phê duyệt và cần gì từ phòng CNTT? | multi_hop | 0.6250 | answer_relevancy | Trả lời đúng thẩm quyền Director phê duyệt và cần xác nhận cấu hình CNTT nhưng thiếu chi tiết bắt buộc phải có ít nhất 3 báo giá cạnh tranh. | Tinh chỉnh prompt để liệt kê đầy đủ các điều kiện tiên quyết trong quy trình mua sắm. |
| 8 | 24 | Nhân viên tạm ứng 15 triệu, sau 20 ngày mới thanh toán. Bị phạt bao nhiêu? | multi_hop | 0.6556 | faithfulness | Phép tính số học phức tạp bị sai lệch do LLM suy luận sai số ngày quá hạn (20 - 15 = 5 ngày) dẫn đến con số phạt bị sai công thức pro-rata. | Tích hợp công cụ tính toán số học (code execution tool) hoặc cung cấp few-shot ví dụ tính phạt theo ngày trong prompt. |
| 9 | 48 | Nhân viên thử việc có được hưởng bảo hiểm sức khỏe PVI không? | adversarial | 0.6667 | answer_relevancy | Câu trả lời chưa trực diện, giải thích dài dòng về gói bảo hiểm thay vì khẳng định ngay nhân viên thử việc chưa được hưởng quyền lợi này. | Ép khuôn phản hồi trả lời khẳng định Có/Không ngay câu đầu tiên trước khi nêu căn cứ. |
| 10 | 31 | Nhân viên đi công tác trong nước, ở khách sạn 2 đêm với giá 1.500.000 VNĐ/đêm. Công ty thanh toán tối đa bao nhiêu cho tiền khách sạn? | multi_hop | 0.6807 | faithfulness | LLM nhầm lẫn giữa chi phí thực tế phát sinh và hạn mức tối đa được thanh toán (1.200.000 VNĐ/đêm), dẫn đến tính sai tổng tiền thanh toán. | Nhấn mạnh quy tắc áp dụng trần định mức công tác phí trong prompt sinh phản hồi. |

### Đối chiếu bằng chứng đại diện theo nhóm lỗi

1. **Lỗi Retrieval (Context Precision/Recall thấp - đại diện: Q7)**:
   - Question: "Nghỉ phép không lương 20 ngày cần ai phê duyệt?"
   - Pipeline Contexts: Lấy được 3 đoạn nhưng 2 đoạn là về phép năm (v2023 và v2024), chỉ 1 đoạn nói về nghỉ không lương (từ 16-30 ngày cần CEO phê duyệt).
   - Pipeline Answer: "Không tìm thấy."
   - Ground Truth: "Nghỉ từ 16-30 ngày không lương cần phê duyệt của Giám đốc điều hành (CEO)."
   - Nhận định: Mặc dù retriever kéo được văn bản chứa đáp án ở rank 1, retriever lại kéo kèm 2 đoạn gây nhiễu về phép năm. Generator bị phân vân và rơi vào cơ chế từ chối trả lời ảo giác, kéo điểm faithfulness xuống 0.0.

2. **Lỗi Generation & Phủ định (Negation Trap - đại diện: Q50)**:
   - Question: "Nhân viên Manager có thể dùng VPN cá nhân (như NordVPN) khi WFH để tăng bảo mật thêm không?"
   - Ground Truth: "KHÔNG. Chính sách VPN quy định rõ: không được sử dụng VPN cá nhân trong mạng công ty dưới bất kỳ hình thức nào."
   - Nhận định: Câu hỏi mang tính gợi ý đánh lừa ("để tăng bảo mật thêm"). Mặc dù tài liệu quy định rõ ràng "Không sử dụng VPN cá nhân (NordVPN, ExpressVPN)", generator lại giải thích dài dòng và làm suy yếu tính phủ định tuyệt đối của chính sách an ninh mạng.

3. **Lỗi Xung đột phiên bản (Version Conflict - đại diện: Q39, Q44)**:
   - Question: "Bao lâu phải đổi mật khẩu một lần?"
   - Ground Truth: "Theo chính sách mật khẩu hiện hành (v2.0), nhân viên KHÔNG bắt buộc đổi mật khẩu định kỳ... Quy định đổi mật khẩu mỗi 90 ngày thuộc chính sách cũ v1.0 đã bị thay thế."
   - Nhận định: Corpus lưu trữ cả 2 phiên bản văn bản. Reranker m3 đưa cả 2 chunk lên top. Generator không có cơ chế đối chiếu metadata ngày hiệu lực nên chọn nhầm quy định cũ đã hết hiệu lực.

## Matrix và nhóm yếu nhất

| Worst Metric | factual (20) | multi_hop (20) | adversarial (10) | Tổng (50) |
|---|---|---|---|---|
| faithfulness | 3 | 13 | 2 | 18 |
| answer_relevancy | 12 | 4 | 2 | 18 |
| context_recall | 2 | 3 | 6 | 11 |
| context_precision | 3 | 0 | 0 | 3 |

Nhóm yếu theo mean thấp nhất: multi_hop; mean: 0.7137.
Worst metric xuất hiện nhiều nhất: faithfulness; số câu: 18 (đồng hạng với answer_relevancy: 18).

### Phân tích nguyên nhân và giải pháp

Nhóm câu hỏi **multi_hop** đạt điểm trung bình thấp nhất (0.7137 so với factual 0.8980 và adversarial 0.7635) vì các lý do cốt lõi sau:
1. **Phụ thuộc đa nguồn tài liệu**: Các câu hỏi multi-hop đòi hỏi phải kết hợp thông tin từ 2 đến 3 tài liệu khác nhau (ví dụ: Q21 cần chính sách nghỉ phép và bảng lương; Q33 cần chính sách phụ cấp và chính sách nghỉ phép). Khi đó, chỉ cần một chunk bị thiếu trong retrieval là điểm `faithfulness` bị sụt giảm nghiêm trọng (13/20 câu multi_hop có worst metric là faithfulness).
2. **Khả năng suy luận tính toán của LLM**: Các câu multi_hop thường kèm theo bài toán tính toán (phạt quá hạn, tổng phụ cấp, ngày phép thâm niên). LLM dạng gpt-4o-mini dễ gặp lỗi số học khi suy luận văn bản thuần túy.

**Đề xuất khắc phục cụ thể:**
- Triển khai **Query Decomposition**: Bẻ câu hỏi phức tạp thành các câu hỏi đơn lẻ, thực hiện retrieval riêng biệt cho từng sub-query rồi tổng hợp contexts.
- Bổ sung **Structured Chain-of-Thought** trong prompt generation để LLM thực hiện tính toán từng bước rõ ràng.
- Chạy lại thực nghiệm với lệnh `python scripts/run_experiments.py --phase a --live` và so sánh mean score của phân bố multi_hop: nếu mean tăng từ 0.7137 lên trên 0.80 thì cải tiến đạt hiệu quả rõ rệt.

## Bẫy adversarial và giới hạn

So sánh với 2 phân bố còn lại, nhóm **adversarial** có điểm context_recall thấp nhất (0.6833 so với factual 0.9250):
- **Bẫy xung đột phiên bản (Version Conflicts)**: Điển hình ở câu 41 và 44, câu hỏi khai thác sự thay đổi giữa chính sách 2023 và 2024. Pipeline đôi khi kéo lẫn cả tài liệu cũ và tài liệu mới, khiến điểm context_recall bị tụt khi retriever chỉ lấy được 1 trong 2 ngữ cảnh cần thiết để phủ định.
- **Bẫy câu hỏi phủ định (Negation)**: Ở câu 46 và 50, câu hỏi đưa ra tiền đề sai ("có được dùng VPN cá nhân không", "thử việc có được nghỉ phép năm không"). Điểm avg_score của adversarial là 0.7635, cao hơn multi_hop (0.7137), tuy nhiên điều này không đồng nghĩa với việc pipeline đã thực sự nhận diện được bẫy lừa một cách hoàn hảo; thực tế LLM đôi khi vẫn đưa ra câu trả lời ba phải thay vì kiên quyết bác bỏ tiền đề sai.
- **Giới hạn thực nghiệm**: Thử nghiệm được thực hiện trên tập mẫu 50 câu (20 factual, 20 multi_hop, 10 adversarial). Tất cả 50 câu đều chạy live qua API của mô hình `openai/gpt-4o-mini`, không sử dụng fallback. Với kích thước mẫu 10 câu adversarial, các kết luận mang tính định hướng phát hiện lỗ hổng hệ thống và cần được kiểm chứng thêm trên các tập tấn công mở rộng lớn hơn.
