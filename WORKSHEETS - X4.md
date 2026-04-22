# THÔNG TIN NHÓM

- **Tên nhóm:** C401-X4

| STT | Thành viên              | Vai trò trong worksheet         |
| --- | ----------------------- | --------------------------------|
| 0   | Dương Khoa Điềm         | Product Lead                    |
| 1   | Đỗ Thế Anh              | Architect                       |
| 2   | Hoàng Thị Thanh Tuyền   | Cost Lead                       |
| 3   | Lê Minh Khang            | Slide, Presenter                |
| 4   | Nguyễn Hồ Bảo Thiên     | Reliability Lead                |
| 5   | Võ Thanh Chung          | Tổng hợp worksheet, Presenter   |

---

# WORKSHEET 0: REFLECTION & PROBLEM DEFINITION

> **Dự án:** VinMecLumina — Trợ lý ảo giải thích kết quả xét nghiệm
> **Mục tiêu worksheet:** Đánh giá nội lực nhóm và xác định lõi bài toán cần giải quyết trước khi bước vào giai đoạn thiết kế hệ thống.

---

## 1. Phản hồi năng lực nhóm (Team Reflection)

### Điểm mạnh

- **Thành thạo LangGraph & Workflow orchestration:** Nhóm đã xây dựng thành công luồng làm việc đa bước với LangGraph, bao gồm các nút Guard, Severity, Explain, Suggest và Chat Agent. Việc quản lý trạng thái (state management) qua các nút logic là năng lực cốt lõi được thể hiện rõ trong kiến trúc hiện tại.

- **Thiết kế Guardrails y tế bài bản:** Nhóm đã xây dựng được hệ thống kiểm soát an toàn gồm: ngưỡng nguy kịch (Critical Thresholds), luồng Escalation khi phát hiện chỉ số đe dọa tính mạng, và nguyên tắc không chẩn đoán/không kê đơn. Đây là nền tảng quan trọng để đảm bảo an toàn bệnh nhân.

- **Tích hợp đa LLM linh hoạt:** Hệ thống hỗ trợ song song nhiều model, kết hợp cơ chế Fallback tự động khi LLM gặp sự cố — đảm bảo tính liên tục dịch vụ (Service Continuity) ngay cả trong điều kiện bất ổn.

- **Nền tảng Observability vững chắc:** Việc tích hợp Langfuse (tracing), Prometheus/Grafana (metrics), structured logging kèm `correlation_id`, và PII Redaction cho thấy nhóm có tư duy hệ thống sản xuất (production-grade thinking) từ sớm, đây là điểm khác biệt lớn so với các dự án MVP thông thường.

- **Kiến trúc Stateless & Redis:** Thiết kế không lưu trạng thái nội bộ trên server mà đẩy toàn bộ vào Redis giúp hệ thống dễ mở rộng theo chiều ngang (horizontal scaling), phù hợp với yêu cầu phục vụ hàng chục nghìn bệnh nhân mỗi ngày của chuỗi Vinmec.

- **Cost Guard & Token Management:** Cơ chế tự động tính toán và giới hạn chi phí token theo từng người dùng cho thấy nhóm đã nhận thức được bài toán tối ưu vận hành ngay từ giai đoạn MVP — một điểm cộng lớn khi trình bày với doanh nghiệp.

---

### Điểm cần cải thiện

- **Kiến thức về hạ tầng Cloud y tế & pháp lý Việt Nam:** Hệ thống hiện tại chưa thể hiện rõ chiến lược tuân thủ Nghị định 13/2023/NĐ-CP về bảo vệ dữ liệu cá nhân nhạy cảm, cũng như Thông tư 46/2018/TT-BYT về hồ sơ bệnh án điện tử. Việc gửi dữ liệu bệnh nhân đến API nước ngoài (Anthropic, Groq) đòi hỏi phải thực hiện DPIA (Data Protection Impact Assessment) và báo cáo cho Bộ Công an trong vòng 60 ngày — đây là rào cản pháp lý chưa được xử lý trong thiết kế hiện tại.

- **Chiến lược De-identification chưa hoàn chỉnh:** PII Redaction hiện đang ở mức ẩn tên và số điện thoại, nhưng chưa có quy trình khử định danh (de-identification) toàn diện trước khi gửi dữ liệu ra ngoài biên giới, bao gồm mã bệnh nhân, ngày sinh, mã xét nghiệm liên kết với danh tính.

- **Tích hợp HIS/LIS còn thiếu:** Kiến trúc hiện tại nhận dữ liệu qua API hoặc Streamlit theo dạng thủ công. Để triển khai thực tế tại Vinmec, hệ thống cần kết nối trực tiếp với HIS (eHospital/Orion Health) và LIS theo chuẩn HL7 FHIR, đảm bảo luồng dữ liệu tự động từ phòng xét nghiệm đến AI mà không cần can thiệp thủ công.

- **Chưa có cơ chế Human-in-the-loop có cấu trúc:** Trong các ca xét nghiệm phức tạp (nhiều chỉ số bất thường tương quan), hệ thống cần định tuyến kết quả đến bác sĩ chuyên khoa phê duyệt trước khi gửi đến bệnh nhân. Điều này chưa được mô hình hóa trong LangGraph hiện tại.

- **Tối ưu hóa chi phí API theo quy mô:** Tại mức 10.000+ người dùng/ngày, chi phí token có thể lên đến $3.300–$4.500/tháng nếu không áp dụng Model Routing, Semantic Caching và Prompt Compression. Nhóm cần nắm vững ba chiến lược tối ưu này để thiết kế hệ thống kinh tế và bền vững.

- **Audit Trail chưa đáp ứng tiêu chuẩn y tế:** Hệ thống cần ghi lại đầy đủ: prompt nào được dùng, mô hình nào, phiên bản nào, dữ liệu đầu vào là gì và phản hồi cuối cùng là gì — không chỉ để debug mà còn để phục vụ thanh tra y tế theo Thông tư 46.

---

### Hướng đi mong muốn

- **Tập trung vào AI Engineering & Medical Accuracy:** Ưu tiên nâng cao độ chính xác y khoa của hệ thống thông qua cải thiện Knowledge Base, tinh chỉnh prompt cho từng nhóm xét nghiệm, và xây dựng bộ test case chuẩn hóa với bác sĩ nội khoa.

- **Hướng tới Enterprise Readiness:** Phát triển theo hướng đáp ứng tiêu chuẩn doanh nghiệp y tế, bao gồm: tuân thủ pháp lý Việt Nam, tích hợp hệ thống bệnh viện, và khả năng mở rộng lên hàng triệu lượt sử dụng/năm theo quy mô của Vinmec (8+ triệu lượt bệnh nhân/năm).

- **Xây dựng nền tảng cho Phase 2:** Thiết kế kiến trúc hiện tại sao cho có thể mở rộng sang Fine-tuning SLM nội bộ (Llama-3-8B với LoRA), Knowledge Graph y khoa chuyên biệt của Vinmec, và tích hợp sâu với ứng dụng MyVinmec.

---

## 2. Xác định bài toán (Problem Statement)

### Vấn đề giải quyết

Bệnh nhân tại Vinmec nhận kết quả xét nghiệm máu dưới dạng bảng số liệu kỹ thuật với các chỉ số như HbA1c, Creatinine, WBC, PLT... kèm đơn vị đo lường và khoảng tham chiếu. Đa số bệnh nhân không có nền tảng y khoa để tự hiểu ý nghĩa của từng chỉ số, đặc biệt là khi nhiều chỉ số cùng lúc vượt ngưỡng bình thường theo các hướng khác nhau. Tình trạng này dẫn đến hai hệ quả tiêu cực: (1) Bệnh nhân lo lắng quá mức hoặc bỏ qua dấu hiệu cảnh báo quan trọng do thiếu ngữ cảnh giải thích; (2) Bác sĩ phải dành thời gian đáng kể để giải thích lại kết quả trong mỗi lần tái khám, làm giảm hiệu suất phòng khám.

VinMecLumina giải quyết vấn đề này bằng cách tự động chuyển đổi dữ liệu xét nghiệm thô thành giải thích ngôn ngữ tự nhiên, dễ hiểu, có phân loại mức độ nghiêm trọng và gợi ý hành động tiếp theo phù hợp — tất cả trong thời gian dưới 8 giây.

### Đối tượng người dùng

- **Người dùng chính:** Bệnh nhân ngoại trú và người khám sức khỏe tổng quát tại Vinmec, sử dụng ứng dụng MyVinmec hoặc nhận kết quả qua email/cổng thông tin bệnh nhân. Đây là nhóm không có chuyên môn y tế, cần ngôn ngữ đơn giản và hành động rõ ràng.

- **Người dùng thứ cấp:** Điều dưỡng và nhân viên y tế sử dụng hệ thống để sàng lọc sơ bộ trước khi bác sĩ xem xét, đặc biệt trong các ca khám sức khỏe tập thể số lượng lớn.

- **Người giám sát:** Bác sĩ chuyên khoa tham gia vào quy trình Human-in-the-loop, phê duyệt các giải thích cho ca có chỉ số phức tạp hoặc nằm trong ngưỡng nguy kịch trước khi gửi đến bệnh nhân.

### Giá trị cốt lõi

- **Chuyển đổi dữ liệu số liệu thành ngôn ngữ dễ hiểu:** Thay thế bảng số kỹ thuật bằng đoạn văn bản giải thích ngắn gọn theo ngôn ngữ bệnh nhân, có thể tùy chỉnh độ phức tạp theo trình độ người đọc.

- **Cảnh báo mức độ nghiêm trọng dựa trên ngưỡng y tế chuẩn hóa:** Phân loại rõ ràng từng chỉ số theo bốn mức — Bình thường, Cần theo dõi, Cần gặp bác sĩ, Nguy kịch — dựa trên ngưỡng lâm sàng được xây dựng theo hướng dẫn của Bộ Y tế và chuẩn quốc tế.

- **An toàn tuyệt đối, không thay thế bác sĩ:** Hệ thống không chẩn đoán bệnh, không kê đơn thuốc. Mọi giải thích đều kèm theo lời khuyên tham khảo bác sĩ, và các ca nguy kịch được ưu tiên chuyển ngay đến nhân viên y tế.

- **Phù hợp với hệ sinh thái Vinmec:** Tích hợp vào luồng vận hành lâm sàng hiện có (HIS, LIS, MyVinmec), đảm bảo dữ liệu được truy vết đầy đủ theo yêu cầu pháp lý và tiêu chuẩn JCI mà Vinmec đang theo đuổi.

---

## 3. Bối cảnh & Ràng buộc (Context & Constraints)

### Môi trường triển khai

VinMecLumina được thiết kế để vận hành trong môi trường y tế kỹ thuật số tại Việt Nam, nơi các quy định pháp lý về dữ liệu y tế ngày càng được siết chặt. Hệ thống cần đồng thời đáp ứng yêu cầu của ba nhóm: bệnh nhân (trải nghiệm dễ dùng), tổ chức y tế (tuân thủ pháp lý và tích hợp hệ thống), và cơ quan quản lý (audit trail và bảo vệ dữ liệu).

### Ràng buộc kỹ thuật chính

- Thời gian phản hồi tối đa: < 3 giây cho tác vụ đơn giản, < 8 giây cho phân tích phức tạp.
- Khả năng xử lý đồng thời ít nhất 100 yêu cầu trong giờ cao điểm (8–10 giờ sáng).
- Tỷ lệ hallucination y khoa mục tiêu: dưới 0.5% đối với thông tin lâm sàng quan trọng.
- Hệ thống phải duy trì hoạt động khi API bên thứ ba gặp sự cố, thông qua cơ chế fallback đa tầng.

### Ràng buộc pháp lý chính

- Tuân thủ Nghị định 13/2023/NĐ-CP: dữ liệu y tế là dữ liệu cá nhân nhạy cảm, bắt buộc khử định danh trước khi gửi ra ngoài lãnh thổ.
- Tuân thủ Thông tư 46/2018/TT-BYT: mọi diễn giải AI phải có khả năng liên kết với hồ sơ bệnh án gốc và đảm bảo audit trail đầy đủ.
- Thực hiện DPIA và báo cáo Bộ Công an trong vòng 60 ngày kể từ khi vận hành thực tế.

---

## 4. Phân tích rủi ro (Risk Analysis)

### Rủi ro kỹ thuật

- **Hallucination y khoa:** Nguy cơ LLM đưa ra giải thích sai lệch về ý nghĩa lâm sàng của một chỉ số, đặc biệt trong các ca bệnh hiếm gặp hoặc khi nhiều chỉ số tương tác phức tạp với nhau. Biện pháp giảm thiểu: kết hợp RAG với Knowledge Base nội bộ, áp dụng Human-in-the-loop cho ca phức tạp, và định kỳ đánh giá ngẫu nhiên bởi bác sĩ.

- **Sự cố API bên thứ ba:** Azure OpenAI hoặc Groq có thể gặp gián đoạn dịch vụ bất ngờ, đặc biệt trong giờ cao điểm xử lý hàng loạt. Biện pháp: triển khai cơ chế fallback đa tầng (Claude → Groq → mô hình nội bộ → thông báo nhân viên y tế).

- **Quá tải hệ thống giờ cao điểm:** Từ 8–10 giờ sáng, hàng trăm kết quả xét nghiệm có thể đổ về cùng lúc. Biện pháp: Message Queue với phân loại ưu tiên (nội trú/cấp cứu > ngoại trú > khám tổng quát) và Batch API cho ca không khẩn cấp.

### Rủi ro pháp lý

- **Rò rỉ dữ liệu bệnh nhân:** Nếu PII không được khử định danh triệt để trước khi gửi đến LLM nước ngoài, Vinmec có thể vi phạm Nghị định 13/2023/NĐ-CP. Biện pháp: xây dựng gateway khử định danh tại chỗ, chỉ gửi chỉ số thuần túy (vd: "Glucose: 6.5 mmol/L") mà không kèm thông tin định danh.

- **Trách nhiệm pháp lý y tế:** Nếu bệnh nhân hành động theo giải thích của AI và gây hại cho sức khỏe, trách nhiệm pháp lý chưa có khung rõ ràng tại Việt Nam. Biện pháp: thêm disclaimer pháp lý rõ ràng trong mỗi giải thích, và ghi nhận sự đồng ý (consent) của bệnh nhân khi sử dụng dịch vụ.

### Rủi ro vận hành

- **Chi phí token vượt kiểm soát:** Nếu không có Cost Guard và Model Routing, chi phí có thể tăng đột biến trong các sự kiện khám sức khỏe tập thể lớn. Biện pháp: kết hợp Rate Limiting per-user trong Redis với cơ chế cảnh báo ngưỡng chi phí theo thời gian thực.

- **Kiến thức y tế lỗi thời:** Các hướng dẫn lâm sàng thay đổi theo thời gian. Nếu Knowledge Base không được cập nhật, hệ thống có thể đưa ra giải thích không còn phù hợp với phác đồ điều trị hiện hành. Biện pháp: lên lịch review Knowledge Base định kỳ 6 tháng/lần với đội ngũ bác sĩ.

---

## 5. Tiêu chí thành công (Success Criteria)

### Tiêu chí kỹ thuật (Giai đoạn MVP)

- Hệ thống xử lý thành công 95%+ các kết quả xét nghiệm máu thông thường mà không cần fallback thủ công.
- Thời gian phản hồi trung bình dưới 5 giây cho 90% request trong điều kiện tải bình thường.
- Tỷ lệ phân loại mức độ nghiêm trọng chính xác (so với đánh giá của bác sĩ) đạt trên 92%.
- Không có trường hợp nào bỏ sót chỉ số nguy kịch (zero miss rate cho Critical Thresholds).

### Tiêu chí người dùng (3 tháng đầu triển khai)

- Điểm hài lòng của bệnh nhân trên ứng dụng MyVinmec đạt tối thiểu 4.3/5 sao.
- Tỷ lệ bệnh nhân hiểu được giải thích mà không cần hỏi thêm nhân viên y tế đạt trên 70% (đo qua khảo sát).
- Tỷ lệ bác sĩ chấp thuận giải thích AI trong luồng Human-in-the-loop đạt trên 85% mà không cần chỉnh sửa đáng kể.

### Tiêu chí doanh nghiệp

- Chi phí vận hành trên mỗi lượt giải thích không vượt 1.000 VNĐ tại quy mô 10.000 người dùng/ngày.
- Hệ thống đáp ứng các yêu cầu kiểm toán trong lần thanh tra đầu tiên của Bộ Y tế hoặc kiểm tra JCI.
- Thời gian tích hợp với HIS/LIS hiện có của Vinmec không vượt quá 3 tháng kể từ khi bắt đầu giai đoạn Enterprise Deployment.
- Hệ thống đóng góp giảm tối thiểu 20% thời gian bác sĩ dành cho việc giải thích kết quả xét nghiệm thông thường tại phòng khám ngoại trú, được đo lường thông qua khảo sát nội bộ sau 6 tháng triển khai.

---

---

### BẢNG VIẾT TẮT

- **PII (Personally Identifiable Information)**
  Thông tin định danh cá nhân (tên, CCCD, ID bệnh nhân).

- **PHI (Protected Health Information)**
  Dữ liệu sức khỏe cá nhân (hồ sơ bệnh án, kết quả xét nghiệm).

- **De-identification**
  Khử định danh – loại bỏ PII trước khi gửi ra ngoài.

- **Re-identification**
  Ghép lại thông tin bệnh nhân sau khi AI xử lý xong.

- **Encryption (TLS 1.3)**
  Mã hóa dữ liệu khi truyền qua mạng.

- **Data Sovereignty**
  Chủ quyền dữ liệu – dữ liệu phải nằm trong phạm vi kiểm soát của tổ chức/quốc gia.

- **HIS (Hospital Information System)**
  Hệ thống quản lý bệnh viện (quản lý hồ sơ, bệnh nhân, lịch khám).

- **LIS (Laboratory Information System)**
  Hệ thống quản lý xét nghiệm.

- **EMR (Electronic Medical Record)**
  Hồ sơ bệnh án điện tử.

- **HL7 (Health Level Seven)**
  Chuẩn trao đổi dữ liệu y tế.

- **FHIR (Fast Healthcare Interoperability Resources)**
  Chuẩn API hiện đại cho dữ liệu y tế.

- **DICOM (Digital Imaging and Communications in Medicine)**
  Chuẩn cho dữ liệu hình ảnh y khoa (X-ray, MRI…).

# WORKSHEET 1: DEPLOYMENT CLINIC - ĐỖ THẾ ANH

**Dự án:** Vinmec Lumina — AI Medical Lab Explainer
**Nhóm:** X1 (Vinmec Track) | **Vai trò:** Solution Architect
**Mục tiêu:** Thiết kế kiến trúc triển khai phù hợp với bối cảnh bệnh viện doanh nghiệp.

---

## Mô hình lựa chọn: Hybrid Architecture

- **On-premise Private Layer** xử lý dữ liệu định danh và tích hợp hệ thống lõi.
- **Cloud LLM Layer** thực hiện suy luận AI trên dữ liệu đã khử định danh.
- **Nguyên tắc kiến trúc:** _"Data sovereignty on-prem — AI capability on cloud."_

---

## Lý do lựa chọn

Quyết định Hybrid xuất phát từ ba ràng buộc không thể đồng thời giải quyết bằng một kiến trúc đơn lẻ:

**1. Ràng buộc pháp lý (Compliance-first)**
Nghị định 13/2023/NĐ-CP phân loại dữ liệu y tế là _dữ liệu cá nhân nhạy cảm_, yêu cầu kiểm soát chặt việc chuyển dữ liệu xuyên biên giới. Cloud thuần túy vi phạm yêu cầu này vì gửi PHI ra máy chủ quốc tế. On-premise thuần túy không đáp ứng nhu cầu suy luận lâm sàng phức tạp. Hybrid là điểm cân bằng pháp lý duy nhất khả thi.

**2. Ràng buộc năng lực AI (Capability Gap)**
LLM mạnh đủ khả năng suy luận y khoa đa chỉ số (Claude 3.5 Sonnet, Llama-3.3-70B qua Groq) chỉ có trên Cloud. On-premise LLM hiện tại (Llama-3-8B, Phi-3) phù hợp làm fallback cục bộ, không đủ năng lực làm primary inference engine cho ca lâm sàng phức tạp.

**3. Ràng buộc tích hợp vận hành (HIS/LIS Integration)**
HIS (Orion Health/eHospital FPT) và LIS nội bộ của Vinmec hoạt động trên môi trường private, không expose ra internet. Bắt buộc phải có on-prem integration layer để kết nối và chuẩn hóa dữ liệu theo chuẩn HL7/FHIR trước khi gọi AI.

---

## Kiến trúc tổng thể (High-Level Architecture)

```
┌─────────────────────────────────────────────────────────┐
│                    ON-PREMISE LAYER                      │
│                                                         │
│  LIS/HIS (HL7/FHIR)                                     │
│       │                                                  │
│       ▼                                                  │
│  [Gateway & Pre-processing]                             │
│   • Validate & normalize dữ liệu xét nghiệm            │
│   • De-identification: xóa tên, CCCD, ID bệnh án        │
│   • Encryption in transit (TLS 1.3)                     │
│   • Audit log ghi nhận request đầu vào                  │
│       │                                                  │
└───────┼─────────────────────────────────────────────────┘
        │ (Chỉ số xét nghiệm đã khử định danh)
        ▼
┌─────────────────────────────────────────────────────────┐
│                    CLOUD LLM LAYER                       │
│                                                         │
│  [LangGraph Router]                                      │
│   • Ca đơn giản (chỉ số bình thường) → Groq Llama-3.3  │
│   • Ca phức tạp (đa chỉ số bất thường) → Claude Sonnet  │
│       │                                                  │
│  [LLM Inference]                                         │
│   • Sinh giải thích lâm sàng                            │
│   • Không nhận PII — chỉ nhận structured lab values     │
│                                                         │
└───────┼─────────────────────────────────────────────────┘
        │ (AI explanation response)
        ▼
┌─────────────────────────────────────────────────────────┐
│                    ON-PREMISE LAYER                      │
│                                                         │
│  [Post-processing & Safety Gate]                        │
│   • Ghép lại thông tin bệnh nhân (re-identification)    │
│   • Áp Rule Engine + Guardrails y khoa                  │
│   • Phát hiện Red Flag → route sang Human-in-the-loop   │
│   • Audit log ghi nhận response cuối                    │
│       │                          │                       │
│       ▼                          ▼                       │
│  [MyVinmec App]          [Bác sĩ xác nhận]              │
│  Hiển thị cho bệnh nhân  (Ca nguy hiểm / bất thường)   │
└─────────────────────────────────────────────────────────┘
```

---

## Ràng buộc Enterprise (Compliance & Governance)

| Ràng buộc                   | Yêu cầu bắt buộc                                                                                                                       |
| :-------------------------- | :------------------------------------------------------------------------------------------------------------------------------------- |
| **Thông tư 46/2018/TT-BYT** | Mọi diễn giải AI phải gắn với hồ sơ bệnh án gốc trong HIS; cập nhật tối đa 12h sau khi có kết quả; hỗ trợ audit trail đầy đủ           |
| **Nghị định 13/2023/NĐ-CP** | Khử định danh PII bắt buộc trước khi gọi API ngoài; nộp DPIA cho Bộ Công an trong 60 ngày; kiểm soát luồng dữ liệu xuyên biên giới     |
| **Data Governance**         | Không gửi ra cloud: tên, CCCD, ID bệnh án, bệnh sử. Chỉ gửi: chỉ số xét nghiệm dạng structured (vd: `Glucose: 6.5 mmol/L`)             |
| **Chuẩn tích hợp**          | HL7/FHIR cho HIS/LIS; DICOM cho imaging (nếu mở rộng); không expose hệ thống lõi ra internet                                           |
| **Audit & Logging**         | Log 100% request/response: model version, prompt hash, input data, output, timestamp — phục vụ truy vết và kiểm toán y tế              |
| **Clinical Safety**         | AI không đưa ra chẩn đoán hoặc phác đồ điều trị; bắt buộc Human-in-the-loop cho mọi ca Red Flag hoặc đa chỉ số bất thường nghiêm trọng |

---

## Đánh đổi (Trade-off Analysis)

| Chiều                      | Đánh giá                                                                                                                                            |
| :------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Bảo mật & Tuân thủ**     | Dữ liệu PHI không rời hạ tầng nội bộ — đáp ứng Nghị định 13 và Thông tư 46                                                                          |
| **Năng lực AI**            | Tận dụng Claude Sonnet cho ca phức tạp, Groq cho ca đơn giản — tối ưu chi phí/chất lượng                                                            |
| **Scalability**            | Cloud side scale linh hoạt theo lưu lượng; on-prem side cần capacity planning riêng                                                                 |
| **Latency**                | Round-trip on-prem → cloud → on-prem cộng thêm ~1–3s so với Cloud thuần. Chấp nhận được ở ngưỡng mục tiêu <8s cho ca phức tạp                       |
| **Engineering Complexity** | Phải xây và duy trì: gateway, de-identification pipeline, LangGraph router, re-identification, rule engine — tăng đáng kể effort so với Cloud thuần |
| **Chi phí vận hành**       | Cao hơn do vừa duy trì on-prem infra, vừa trả Cloud API cost. Bù lại, model routing giảm ~40% token cost so với dùng Claude cho mọi request         |
| **Rủi ro còn lại**         | Nếu de-identification pipeline thiết kế kém, vẫn có nguy cơ lộ thông tin gián tiếp qua context xét nghiệm                                           |

**Thứ tự ưu tiên chiến lược:** `Safety > Compliance > AI Quality > Cost > Speed`

---

## Kết luận kiến trúc

Hybrid Architecture cho Vinmec Lumina không phải một lựa chọn kỹ thuật tùy ý — đây là **ràng buộc xuất phát từ môi trường pháp lý, năng lực AI thị trường và thực tế vận hành bệnh viện**. Phân chia trách nhiệm rõ ràng:

- **On-prem:** Chủ quyền dữ liệu, tích hợp HIS/LIS, audit log, safety gate
- **Cloud LLM:** Suy luận AI trên dữ liệu đã khử định danh

Đây là mô hình chuẩn cho **AI healthcare enterprise systems** trong giai đoạn pháp lý và công nghệ hiện tại tại Việt Nam.

---

# Worksheet 2 — Cost Structure của Vinmec Lumina

## Mục tiêu

Tách chi phí của hệ thống AI ra khỏi góc nhìn hẹp chỉ tính token/API. Với bài toán Vinmec Lumina, chi phí thực tế cần nhìn theo 5 lớp:

1. Token / LLM inference.
2. Compute / orchestration / API gateway.
3. Storage / logging / audit trail.
4. Human review / clinical validation.
5. Maintenance / compliance / knowledge base update.

Kết luận ngắn gọn: ở MVP, token cost là khoản nhìn thấy rõ nhất; khi scale lớn, human review, logging, và hạ tầng quan sát thường tăng nhanh hơn team dự tính.

---

## 1) Ước lượng demand: user/ngày, request/ngày, peak traffic

### 1.1 Giả định MVP

Dựa trên spec của dự án, flow chính là bệnh nhân mở kết quả xét nghiệm, AI giải thích cá nhân hóa, gắn severity, và gợi ý bước tiếp theo. Một user thường phát sinh 1-2 lượt LLM-backed request trong cùng một phiên.

| Chỉ số             |               Ước lượng MVP | Ghi chú                                                        |
| ------------------ | --------------------------: | -------------------------------------------------------------- |
| Daily active users |       800 - 1,000 user/ngày | Quy mô MVP cho một nhóm bệnh nhân đang xem kết quả xét nghiệm  |
| Requests per user  | 1.2 - 1.8 request/user/ngày | Gồm explain, follow-up, hỏi lại ngắn                           |
| Total requests     |  1,200 - 1,800 request/ngày | Đây là số request có thể chạm LLM hoặc lớp reasoning           |
| Peak traffic       |         8 - 12 request/phút | Giả định traffic dồn vào giờ hành chính và sau giờ trả kết quả |

### 1.2 Giả định tăng trưởng

| Chỉ số             |               5x MVP |                10x MVP |
| ------------------ | -------------------: | ---------------------: |
| Daily active users |        4,000 - 5,000 |         8,000 - 10,000 |
| Total requests     | 6,000 - 9,000 / ngày | 12,000 - 18,000 / ngày |
| Peak traffic       | 40 - 60 request/phút |  80 - 120 request/phút |

### 1.3 Điểm cần lưu ý

- Peak không tăng tuyến tính hoàn toàn theo daily average vì traffic y tế thường dồn vào khung giờ bệnh nhân nhận kết quả.
- Nếu có chat follow-up, số request/user có thể tăng từ 1.2 lên 2.0+ rất nhanh.
- Nếu bật human-in-the-loop cho ca khó, request không đổi nhiều nhưng chi phí vận hành tăng mạnh.

---

## 2) Ước lượng input/output tokens

Theo nội dung tham chiếu trong đề bài, một báo cáo xét nghiệm máu điển hình thường nằm trong khoảng 1,000 - 2,000 token tổng cộng.

### 2.1 Token breakdown

| Thành phần             | Token ước tính | Giải thích                                                                          |
| ---------------------- | -------------: | ----------------------------------------------------------------------------------- |
| Input prompt + context |    800 - 1,500 | Bao gồm hồ sơ bệnh nhân, kết quả xét nghiệm structured, reference range, KB snippet |
| Output explanation     |      200 - 500 | Giải thích dễ hiểu, severity, next step                                             |
| Tổng                   |  1,000 - 2,000 | Phù hợp với flow giải thích ngắn trong MVP                                          |

### 2.2 Cost mỗi request theo provider

| Provider           | Input cost / 1k | Output cost / 1k | Cost/request ước tính |
| ------------------ | --------------: | ---------------: | --------------------: |
| Claude 3.5 Sonnet  |         $0.0037 |          $0.0157 |      $0.0054 - $0.012 |
| Groq Llama-3.3-70B |      $0.0005918 |       $0.0007918 |  $0.000628 - $0.00128 |

### 2.3 Nhận định

- Claude đắt hơn Groq khoảng 8-10x cho cùng một khối lượng dữ liệu.
- Nhưng Claude phù hợp hơn cho case phức tạp, nhiều bất thường, cần diễn giải lâm sàng an toàn.
- Groq phù hợp cho case đơn giản, follow-up ngắn, hoặc fallback tiết kiệm chi phí.

---

## 3) Các lớp cost của hệ thống

### 3.1 Token / LLM inference

Đây là chi phí dễ nhìn thấy nhất nhưng không phải lớn nhất khi hệ thống tăng trưởng.

- Bao gồm prompt, context, retrieval, output generation.
- Tăng theo số request và độ dài prompt.
- Bị ảnh hưởng mạnh bởi cách routing model và cách cắt context.

### 3.2 Compute / API / orchestration

Gồm FastAPI, worker, routing logic, validation, RAG orchestration, Redis session, và các lớp guardrail.

- MVP thường không quá đắt.
- Khi lên 10k+ user/ngày, cần autoscaling, load balancing, và theo dõi latency.
- Nếu workflow có nhiều bước tuần tự, chi phí compute tăng theo độ dài pipeline chứ không chỉ theo token.

### 3.3 Storage / logging / audit trail

Phần này thường bị đánh giá thấp.

- Lưu conversation, request metadata, audit log, trace, metrics.
- Y tế thường cần retention dài hơn app thông thường.
- Nếu giữ cả raw prompt/response để kiểm toán, dung lượng và chi phí truy xuất tăng nhanh.

### 3.4 Human review / clinical validation

Đây là hidden cost dễ quên nhất.

- Các case red flag, borderline, hoặc low-confidence cần bác sĩ/chuyên gia duyệt lại.
- Nếu tỷ lệ review chỉ 5% - 10% và mỗi lượt mất vài phút, chi phí nhân sự đã có thể vượt token cost ở một số giai đoạn.
- Với use case y tế, review không chỉ là chi phí tiền mà còn là chi phí lịch của bác sĩ.

### 3.5 Maintenance / compliance / knowledge base update

Y học thay đổi liên tục, nên hệ thống phải cập nhật prompt, rule severity, KB, và nội dung an toàn.

- Cập nhật reference range và guideline mới.
- Sửa prompt để tránh hallucination hoặc over-interpretation.
- Kiểm tra tuân thủ bảo mật, PII, và audit.

---

## 4) Tính sơ bộ cost ở mức MVP

### 4.1 Base case theo đề bài

| Hạng mục                                | Monthly cost ước tính |
| --------------------------------------- | --------------------: |
| Token cost hybrid Claude/Groq           |           $250 - $400 |
| Cloud infra (server, database, gateway) |           $200 - $500 |
| Tổng                                    |  ~$500 - $900 / tháng |

### 4.2 Diễn giải thực tế

Con số trên hợp lý nếu:

- Một phần case đơn giản đi qua Groq để tối ưu chi phí.
- Claude chỉ dùng cho case khó hoặc khi cần chất lượng diễn giải cao.
- Hệ thống chưa giữ log/audit quá lâu và chưa có reviewer full-time.

### 4.3 Nếu cộng thêm hidden cost

Nếu tính đầy đủ hơn cho vận hành doanh nghiệp, MVP có thể tách thêm:

| Hidden cost             |       Ước lượng MVP | Ghi chú                            |
| ----------------------- | ------------------: | ---------------------------------- |
| Human review            | $100 - $300 / tháng | Tùy tỷ lệ case cần bác sĩ duyệt    |
| Logging / observability |   $20 - $80 / tháng | Lưu log, metrics, dashboard, trace |
| Maintenance / KB update | $100 - $250 / tháng | Chủ yếu là công sức team nội bộ    |

Khi đó tổng OPEX thực dụng có thể tiến gần $700 - $1,400/tháng nếu vận hành nghiêm túc hơn mức demo.

---

## 5) Điều gì tăng mạnh nhất khi user tăng 5x hoặc 10x?

### 5.1 Khi tăng 5x

| Lớp cost     | Mức tăng                        | Lý do                                                     |
| ------------ | ------------------------------- | --------------------------------------------------------- |
| Token        | Tăng gần tuyến tính             | Mỗi request vẫn phải inference                            |
| Infra        | Tăng vừa phải                   | API, Redis, DB, autoscaling                               |
| Logging      | Tăng tuyến tính                 | Mỗi request sinh thêm log/trace                           |
| Human review | Có thể tăng mạnh hơn tuyến tính | Ca phức tạp và borderline thường tăng theo volume thực tế |

### 5.2 Khi tăng 10x

| Lớp cost                 | Mức tăng                              | Vì sao                                            |
| ------------------------ | ------------------------------------- | ------------------------------------------------- |
| Token                    | Tăng gần 10x nếu routing không tối ưu | Đây là chi phí trực tiếp nhất                     |
| Infra                    | Tăng rõ rệt                           | Cần autoscaling, cache, queue, monitoring tốt hơn |
| Human review             | Có thể thành khoản lớn nhất           | Nếu tỷ lệ case cần duyệt không giảm theo quy mô   |
| Compliance / maintenance | Tăng đáng kể                          | Nhiều team, nhiều workflow, nhiều audit hơn       |

Kết luận: nếu system routing tốt, token cost không nhất thiết là khoản lớn nhất ở quy mô lớn. Hidden cost thường thắng ở human review và compliance.

---

## 6) Trả lời 3 câu hỏi bắt buộc

### 6.1 Cost driver lớn nhất của hệ thống là gì?

Ở MVP, cost driver lớn nhất là token / LLM inference. Ở giai đoạn tăng trưởng, cost driver lớn nhất có thể chuyển sang human review + compliance + logging nếu hệ thống được triển khai như một sản phẩm y tế nghiêm túc.

### 6.2 Hidden cost nào dễ bị quên nhất?

Human review là hidden cost dễ bị quên nhất, vì nó không hiện ra trong bảng giá API nhưng lại ăn ngân sách và thời gian chuyên gia y tế rất nhanh. Sau đó mới đến logging/audit trail và maintenance knowledge base.

### 6.3 Đội có chỗ nào đang ước lượng quá lạc quan không?

Có 3 điểm dễ lạc quan quá mức:

1. Giả sử mỗi user chỉ hỏi 1 request/ngày, trong khi thực tế follow-up có thể làm số request tăng gấp 2.
2. Giả sử token cost là toàn bộ OPEX, trong khi review và audit có thể vượt token ở môi trường y tế.
3. Giả sử prompt/context không lớn lên theo thời gian, trong khi use case cá nhân hóa thường kéo prompt dài hơn khi thêm lịch sử, KB, và cảnh báo an toàn.

---

## 7) Kết luận ngắn

Với Vinmec Lumina, chi phí token ở MVP là chấp nhận được, nhưng không nên định giá hệ thống chỉ theo API cost. Khi scale lên 5x-10x, cần theo dõi đồng thời token, peak traffic, human review, và chi phí audit/logging. Nếu không, phần tốn tiền nhất có thể không còn là LLM mà là vận hành y tế và tuân thủ.

---

# WORKSHEET 3: COST OPTIMIZATION

**Dự án:** Vinmec Lumina — AI Medical Lab Explainer  
**Mục tiêu:** Tối ưu chi phí vận hành hệ thống AI ở quy mô enterprise, đồng thời đảm bảo độ chính xác y khoa và tuân thủ.

---

## 1. Chiến lược 1: Model Routing (Định tuyến mô hình)

### Mô tả

Không phải mọi yêu cầu đều cần sử dụng mô hình mạnh và chi phí cao. Hệ thống cần một lớp điều phối thông minh để phân loại mức độ phức tạp và chọn model phù hợp.

### Cách triển khai

- Xây dựng **classification layer (router)** dựa trên:

  - Số lượng chỉ số bất thường
  - Mức độ lệch chuẩn (so với reference range)
  - Pattern nguy hiểm (rule-based)

- Phân tầng xử lý:
  - **Level 1 (Simple):** Chỉ số bình thường → Groq (Llama-3)
  - **Level 2 (Moderate):** 1–2 chỉ số lệch nhẹ → Groq + rule-based
  - **Level 3 (Complex):** Nhiều chỉ số bất thường → Claude

### Logic định tuyến (pseudo-code)

```python
if abnormal_count == 0:
    route = "Groq"
elif abnormal_count <= 2:
    route = "Groq + rule_engine"
else:
    route = "Claude"
```

### Lợi ích

- Giảm ~40–60% chi phí token
- Giảm tải cho model cao cấp trong giờ cao điểm
- Tăng tốc phản hồi cho đa số user

### Trade-off

- Nguy cơ phân loại sai → dùng model yếu cho case phức tạp
- Giải pháp:
  - Rule override cho các chỉ số nguy hiểm (red flag)
  - Ví dụ: Glucose > ngưỡng nguy hiểm → luôn dùng Claude

---

## 2. Chiến lược 2: Semantic Caching (Bộ nhớ đệm ngữ nghĩa)

### Mô tả

Lưu trữ và tái sử dụng các kết quả giải thích cho các input tương tự về mặt ngữ nghĩa.

### Cách triển khai

- Sử dụng:

  - Vector Database (Redis, FAISS, Pinecone)

- Lưu trữ:

  - Embedding của input (lab pattern)
  - Response đã được kiểm chứng

- Pipeline:
  1. Convert input → embedding
  2. Search similarity (cosine similarity)
  3. Nếu similarity > 0.9 → return cached response
  4. Nếu không → gọi LLM và lưu cache

### Phạm vi áp dụng

- **Áp dụng:**
  - Giải thích chỉ số phổ biến (WBC, RBC, Glucose)
- **Không áp dụng:**
  - Kết luận cá nhân hóa
  - Khuyến nghị điều trị

### Lợi ích

- Giảm 20–40% số request LLM
- Giảm latency từ giây xuống mili giây

### Trade-off

- Nguy cơ trả lời không phù hợp ngữ cảnh
- Giải pháp:
  - Tách response thành:
    - Phần chung → cache
    - Phần cá nhân hóa → LLM xử lý

---

## 3. Chiến lược 3: Prompt Compression (Nén ngữ cảnh)

### Mô tả

Giảm kích thước prompt đầu vào bằng cách chỉ giữ lại thông tin y khoa liên quan.

### Cách triển khai

- Pipeline:

  1. Retrieve tài liệu (RAG)
  2. Lọc nội dung liên quan
  3. Nén bằng công cụ (LLMLingua hoặc custom)

- Kỹ thuật:
  - Keyword filtering
  - Section ranking
  - Attention-based compression

### Ví dụ

**Trước:**

```
Full guideline (~2000 tokens)
```

**Sau:**

```
Relevant section (~200 tokens)
```

### Lợi ích

- Giảm 50–80% token đầu vào
- Tăng throughput hệ thống
- Giảm chi phí tuyến tính theo quy mô

### Trade-off

- Mất thông tin nếu nén quá mức
- Giải pháp:
  - Luôn giữ:
    - Reference range
    - Chỉ số bất thường
    - Clinical rules quan trọng

---

## Tổng hợp chiến lược

| Chiến lược         | Vai trò            | Impact                    |
| ------------------ | ------------------ | ------------------------- |
| Model Routing      | Giảm chi phí model | Tối ưu chi phí lớn nhất   |
| Semantic Caching   | Giảm số request    | Tăng tốc + giảm chi phí   |
| Prompt Compression | Giảm token         | Tối ưu chi phí tuyến tính |

---

## Kiến trúc tổng thể

```
Input Lab Result
      ↓
[Classifier / Router]
      ↓
[Cache Check] → hit → return
      ↓ miss
[Prompt Compression]
      ↓
[LLM (Groq / Claude)]
      ↓
[Post-process + Safety Check]
      ↓
[Cache Store]
      ↓
Output
```

---

## Kết luận

Ba chiến lược cần được triển khai đồng thời:

- **Model Routing:** kiểm soát chi phí lớn nhất
- **Semantic Caching:** giảm tải hệ thống
- **Prompt Compression:** tối ưu chi phí theo scale

→ Đây là nền tảng bắt buộc để Vinmec Lumina vận hành ở quy mô enterprise với chi phí hợp lý và độ chính xác y khoa cao.

---

# WORKSHEET 4: Reliability and Scaling

**Dự án:** Vinmec Lumina — AI Medical Lab Explainer

**Nhóm:** X1 (Vinmec Track)

**Mục tiêu:** Đảm bảo hệ thống luôn hoạt động ổn định, an toàn và duy trì chất lượng phản hồi ngay cả khi xảy ra lỗi hoặc khi lưu lượng truy cập tăng cao.

### Tổng quan

Đối với một hệ thống hỗ trợ y tế như _Vinmec Lumina_, độ tin cậy (reliability) không đơn thuần là uptime hay latency, mà trực tiếp liên quan đến **an toàn bệnh nhân (patient safety)**. Một phản hồi sai lệch hoặc chậm trễ trong bối cảnh y khoa có thể dẫn đến hậu quả nghiêm trọng. Do đó, hệ thống phải được thiết kế theo nguyên tắc: **fail gracefully, degrade safely, và luôn có human fallback**.

---

## **1. Failure Scenarios**

Để đảm bảo độ tin cậy, trước tiên cần xác định rõ các loại failure có thể xảy ra:

### **1.1. Lỗi từ bên thứ ba (External Dependency Failure)**

- API LLM (Claude, Groq) không phản hồi
- Rate limit bị vượt quá trong giờ cao điểm
- Độ trễ tăng cao do overload từ phía provider

Đây là failure phổ biến nhất trong hệ thống phụ thuộc AI-as-a-service.

---

### **1.2. Lỗi hạ tầng mạng**

- Mất kết nối internet quốc tế
- Đứt cáp hoặc latency tăng mạnh khi gọi API nước ngoài

Đặc biệt quan trọng tại Việt Nam (thực tế xảy ra thường xuyên).

---

### **1.3. Lỗi mô hình AI (Model-level failure)**

- Hallucination (trả lời sai về y khoa)
- Output không đầy đủ / không rõ ràng
- Model từ chối trả lời (safety filter)

---

### **1.4. Lỗi hệ thống nội bộ**

- Queue backlog quá lớn → request bị delay
- Service crash (container / pod)
- Database quá tải

---

## **2. Thiết kế Fallback (Cơ chế dự phòng nhiều tầng)**

Hệ thống cần thiết kế theo **multi-layer fallback architecture** thay vì chỉ 1 phương án dự phòng.

---

### **2.1. Dự phòng API (Model-level fallback)**

- Primary: Claude 3.5 Sonnet
- Secondary: Groq Llama-3.3-70B

Cơ chế:

- Timeout threshold (ví dụ: 5s)
- Nếu vượt timeout hoặc lỗi → auto switch model

**Trade-off:**

- Claude: chất lượng cao, nhưng đắt và có rate limit
- Groq: nhanh, rẻ hơn nhưng có thể kém chính xác hơn

Giải pháp này đảm bảo:

- **Availability tăng**
- **Latency ổn định hơn trong peak time**

---

### **2.2. Dự phòng mô hình cục bộ (Offline fallback)**

Trong trường hợp mất internet:

- Sử dụng model nội bộ:

  - Llama-3-8B
  - Phi-3

Vai trò:

- Chỉ cung cấp:

  - Giải thích cơ bản
  - Không đưa ra khuyến nghị y khoa quan trọng

**Trade-off:**

- Ưu điểm: luôn available
- Nhược điểm: độ chính xác thấp hơn cloud model

Vì vậy cần gắn nhãn rõ: _“Thông tin tham khảo – cần xác nhận với bác sĩ”_

---

### **2.3. Human-in-the-loop fallback**

Nếu tất cả AI fail:

- Chuyển sang:

  - “Hỗ trợ từ nhân viên y tế”

- Notify rõ cho bệnh nhân

Đây là **fallback quan trọng nhất về mặt an toàn**.

---

## **3. Monitoring & KPI (Giám sát hệ thống)**

Một hệ thống reliable không chỉ là “ít lỗi” mà là **phát hiện lỗi sớm và phản ứng nhanh**.

---

### **3.1. Các chỉ số kỹ thuật**

- **Latency**

  - < 3s (simple query)
  - < 8s (complex analysis)

- **Throughput**

  - ≥ 100 concurrent requests

- **Error rate**

  - % request fail / timeout

---

### **3.2. Các chỉ số AI-specific**

- **Hallucination Rate**

  - Đo bằng audit từ bác sĩ
  - Target: < 0.5%

Đây là KPI cực kỳ quan trọng (điểm cộng lớn nếu bạn mention).

---

### **3.3. Các chỉ số trải nghiệm người dùng**

- **User satisfaction (MyVinmec)**

  - Target: > 4.5/5

Kết nối giữa reliability và business outcome.

---

### **3.4. Monitoring system**

- Real-time dashboard (Prometheus / Grafana)
- Alert khi:

  - Latency spike
  - Error rate tăng
  - Queue backlog lớn

---

## **4. Scaling & Load Handling (Xử lý tải và mở rộng)**

---

### **4.1. Đặc thù tải tại Vinmec**

- Peak: 8h–10h sáng
- Nguyên nhân:

  - Kết quả xét nghiệm trả hàng loạt

Đây là **burst traffic**, không phải steady load.

---

### **4.2. Message Queue (Giải quyết burst load)**

Kiến trúc:

- Request → Queue → Worker → AI

Ưu điểm:

- Tránh overload trực tiếp vào AI API
- Có thể scale worker độc lập

---

### **4.3. Priority-based processing**

Thứ tự ưu tiên:

1. Cấp cứu / nội trú
2. Ngoại trú
3. Khám tổng quát

Đảm bảo critical case không bị delay

---

### **4.4. Batch Processing (Cost vs Latency trade-off)**

- Non-urgent:

  - Dùng Batch API
  - Delay: ≤ 24h
  - Cost giảm ~50%

Trade-off rõ ràng:

- Giảm cost
- Tăng latency (acceptable với non-critical)

---

### **4.5. Horizontal Scaling**

- Scale worker theo:

  - Queue length
  - CPU usage

- Auto-scaling (Kubernetes HPA)

---

## **5. Đảm bảo Stability (Độ ổn định hệ thống)**

---

### **5.1. Circuit Breaker Pattern**

- Nếu API lỗi liên tục:

  - Tạm ngắt gọi
  - Chuyển sang fallback

Tránh cascade failure

---

### **5.2. Retry + Exponential Backoff**

- Retry thông minh
- Tránh spam API

---

### **5.3. Load Shedding**

- Khi quá tải:

  - Drop request low priority

Bảo vệ hệ thống khỏi crash toàn bộ

---

## **Kết luận**

Thiết kế Reliability & Scaling cho Vinmec Lumina cần tiếp cận theo hướng:

- **Multi-layer fallback** → đảm bảo luôn có câu trả lời
- **Monitoring real-time** → phát hiện và xử lý sớm
- **Load-aware architecture** → xử lý burst traffic hiệu quả
- **Safety-first mindset** → luôn có human fallback
