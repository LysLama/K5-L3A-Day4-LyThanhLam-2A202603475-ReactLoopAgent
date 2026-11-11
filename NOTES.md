# Ghi chú phân tích · Day 04

Họ và tên: Lý Thanh Lâm
MSSV: 2A202603475

## 1. Case được chọn

- ID case: H1
- Câu trả lời của agent:
> ### Thông tin về Incident GitHub (ID: h02n9572mx9y)
> - **Tên:** Incident with Actions
> - **Trạng thái:** Đã giải quyết
> - **Thời gian bắt đầu:** 2026-11-04T17:40:33.675Z
> - **Thời gian kết thúc:** 2026-11-07T01:10:18.506Z
> - **Components ảnh hưởng:** Actions
> - **Cập nhật:**
>   - **2026-11-04T17:40:33.887Z:** Đang điều tra báo cáo về khả năng sử dụng bị giảm đối với Actions.
>   - **2026-11-04T18:07:51.114Z:** Sự giảm hiệu suất đối với Actions đã được khắc phục. Đang theo dõi để đảm bảo tính ổn định.
>   - **2026-11-04T18:16:08.361Z:** Vui lòng tham khảo tóm tắt tổng hợp trong sự cố liên quan: [Link](https://www.githubstatus.com/incidents/xr3myzj7ms51)
> 
> Nguồn: [GitHub Status](https://stspg.io/7pwv28hvkmz8)
> 
> ### Thông tin về Incident Cloudflare (ID: yaa0witbl2me)
> - **Tên:** Workers Builds Issues
> - **Trạng thái:** Đã giải quyết
> - **Thời gian bắt đầu:** 2026-11-05T19:51:39.176Z
> - **Thời gian kết thúc:** 2026-11-05T20:20:10.166Z
> - **Components ảnh hưởng:** Workers Builds
> - **Cập nhật:**
>   - **2026-11-05T19:51:39.176Z:** Cloudflare đã nhận thức và đang điều tra vấn đề với Cloudflare Workers Builds có khả năng ảnh hưởng đến nhiều khách hàng.
>   - **2026-11-05T19:55:51.237Z:** Các builds đang được xử lý và chúng tôi đang làm việc để vượt qua backlog.
>   - **2026-11-05T20:20:10.166Z:** Thời gian xếp hàng đã trở về trạng thái bình thường.
> 
> Nguồn: [Cloudflare Status](https://www.cloudflarestatus.com/incidents/yaa0witbl2me)
> 
> ### So sánh và phân tích
> - **Thời gian:** GitHub: 2026-11-04 đến 2026-11-07; Cloudflare: 2026-11-05.
> - **Trạng thái:** Cả hai sự cố đều đã được giải quyết.
> - **Component:** GitHub ảnh hưởng "Actions", Cloudflare ảnh hưởng "Workers Builds".
> - **Nguyên nhân chung:** Không thể xác định nguyên nhân chung giữa hai sự cố dựa trên thông tin có sẵn...
- Điểm ở lần chạy 1: 60/100 (Bị trần điểm cơ học `score_cap: 60` do vi phạm mechanical check: `"Cần tool calls ở ít nhất 2 model steps khác nhau."`, mặc dù điểm thô của Judge là 80/100).

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra:

```json
[
  {
    "event": "tool",
    "step": 1,
    "name": "get_incident_details",
    "arguments": "{\"provider\": \"github\", \"incident_id\": \"h02n9572mx9y\"}",
    "result": {
      "ok": true,
      "provider": "github",
      "id": "h02n9572mx9y",
      "name": "Incident with Actions",
      "status": "resolved",
      "updates": [
        {
          "status": "resolved",
          "created_at": "2026-11-04T18:16:08.361Z",
          "body": "Please refer to the combined summary in this related incident: https://www.githubstatus.com/incidents/xr3myzj7ms51",
          "truncated": false
        }
      ],
      "url": "https://stspg.io/7pwv28hvkmz8"
    }
  },
  {
    "event": "tool",
    "step": 1,
    "name": "get_incident_details",
    "arguments": "{\"provider\": \"cloudflare\", \"incident_id\": \"yaa0witbl2me\"}",
    "result": {
      "ok": true,
      "provider": "cloudflare",
      "id": "yaa0witbl2me",
      "name": "Workers Builds Issues",
      "status": "resolved",
      "url": "https://www.cloudflarestatus.com/incidents/yaa0witbl2me"
    }
  }
]
```

**Nguyên nhân:**
* Đề bài yêu cầu: *"Điều tra incident GitHub h02n9572mx9y. Theo liên kết do incident cung cấp để tìm nguyên nhân và ảnh hưởng..."*.
* Tại `step 1`, agent gọi song song 2 tools để lấy thông tin `h02n9572mx9y` và `yaa0witbl2me`.
* Trong kết quả của `h02n9572mx9y`, bản cập nhật nêu rõ sự cố đã được gộp: `"Please refer to the combined summary in this related incident: https://www.githubstatus.com/incidents/xr3myzj7ms51"`.
* Tuy nhiên, agent dừng lại ngay sau step 1 và đưa ra câu trả lời, không tiếp tục thực hiện thêm bước 2 để gọi tool tra cứu incident liên quan (`xr3myzj7ms51`).
* Việc thiếu bước gọi tool thứ 2 khiến agent không lấy được nguyên nhân chi tiết của incident GitHub gộp, dẫn đến vi phạm ràng buộc kỹ thuật `min_tool_steps: 2` và bị phạt trần điểm 60.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu:
  * Cập nhật `SYSTEM_PROMPT` trong `agent.py`: Bổ sung chỉ dẫn tường minh về việc điều tra chuỗi sự cố liên kết:
    *"Khi kết quả điều tra một incident có chứa liên kết hoặc thông báo gộp vào incident khác (ví dụ: 'combined summary in this related incident' hoặc link incident ID), bạn PHẢI tiếp tục gọi get_incident_details để đọc incident liên kết đó trước khi kết luận."*
  * Đồng thời bổ sung chỉ dẫn ứng xử linh hoạt với các yêu cầu ngoài phạm vi trạng thái hạ tầng (ví dụ yêu cầu làm thơ hoặc trò chuyện thông thường) bằng cách phản hồi ngắn gọn, lịch sự mà không bị từ chối máy móc hoặc gọi tool thừa.
- Vì sao chọn thay đổi này:
  * Vòng lặp ReAct có bản chất là chu trình Suy luận (Reason) và Hành động (Act) liên tiếp. Khi quan sát thấy bằng chứng chưa đầy đủ hoặc có con trỏ dẫn tiếp tới nguồn dữ liệu khác, agent cần biết lặp lại bước Act thay vì kết luận vội vàng.
  * Tinh chỉnh prompt hướng dẫn mô hình nhận diện các liên kết incident chaining giúp giải quyết triệt để lỗi ở các case đòi hỏi điều tra nhiều bước (như H1, H6) mà không phá vỡ hợp đồng của các case khác.

## 4. So sánh hai lần chạy

| Chỉ số                 | `eval_before.json` | `eval_after.json` |
| ------------------------ | -------------------- | ------------------- |
| Điểm tổng             | 74.09 / 100          | **80.68 / 100** (+6.59 điểm) |
| Điểm case đã chọn (H1) | 60 / 100             | 60 / 100            |
| Số model calls          | 50 (agent) / 72 (tổng)| 48 (agent) / 70 (tổng) |
| Token, chi phí nếu có | 47,844 tokens (~$0.011 USD) | 49,536 tokens (~$0.010 USD) |

- **Lỗi đã giải quyết:**
  * Điểm tổng thể tăng mạnh từ **74.09 lên 80.68 điểm**.
  * **Giải quyết hoàn toàn lỗi chuỗi liên kết nhiều bước (Incident Chaining) ở case H6:** Ở lần chạy 2, agent đã tự động gọi tiếp `get_incident_details(provider='github', incident_id='xr3myzj7ms51')` ở Step 2 đúng theo đường dẫn từ `h02n9572mx9y`. Vi phạm mechanical check ở H6 bị xóa bỏ hoàn toàn (violation cap được gỡ bỏ từ 60 lên 100), điểm tăng từ 60 lên 70.
  * **Xử lý triệt để yêu cầu ngoài luồng ở case H4:** Điểm tăng vọt từ **15 lên 75 (+60 điểm)** nhờ agent đã đáp ứng làm thơ về biển một cách sáng tạo, lịch sự mà tuân thủ đúng yêu cầu không gọi bất kỳ tool trạng thái nào.
  * **Xử lý chuẩn xác khi gặp timeout ở case E6:** Điểm tăng từ **65 lên 100 tuyệt đối (+35 điểm)** nhờ chỉ dẫn báo rõ thiếu dữ liệu thay vì suy đoán trạng thái.
  * Nhiều case khác tăng điểm mạnh: A1 (80 -> 100), A2 (85 -> 100), A3 (75 -> 95), A5 (55 -> 75).

- **Lỗi còn lại hoặc mới xuất hiện:**
  * **Lỗi còn lại ở case H1:** Mặc dù prompt đã bổ sung quy tắc incident chaining và đã giúp H6 thành công, ở H1 mô hình vẫn dừng lại sau step 1 (do đã gọi song song cả 2 provider GitHub và Cloudflare ở step 1 nên ngộ nhận là đã có đủ dữ liệu so sánh). Điều này cho thấy với mô hình nhỏ (`gpt-4o-mini`), khi câu hỏi yêu cầu đồng thời nhiều tác vụ (vừa tra 2 bên, vừa so sánh, vừa truy vết liên kết), mô hình vẫn có xác suất bỏ sót bước tra cứu sâu thứ 2 nếu không có cơ chế code enforcement chặt chẽ.
  * **Biến động ở case H2 (100 -> 50):** Do tính phân tán (variance) ngẫu nhiên của mô hình, ở lần 2 model chỉ gọi `get_incident_details` cho `t89g2q1800fr` mà quên gọi `get_status` để kiểm tra trạng thái hiện tại của Actions, dẫn đến bị phạt mechanical cap 50 điểm ở case này.

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.
* **Giới hạn của LLM Judge:** Judge cũng là một mô hình ngôn ngữ (ở đây là `openai/gpt-4o-mini`), có tính ngẫu nhiên (non-deterministic) và có thể có thiên lệch nhẹ khi tự đánh giá output của chính cùng họ mô hình. Điểm số chỉ mang tính ước lượng tương đối, cần đối chiếu với trace và mechanical check.
* **Giới hạn của Snapshot:** Dữ liệu snapshot được đóng băng tại thời điểm cụ thể (tháng 11/2026), chỉ chứa tối đa 50 incident gần nhất. Việc không thấy incident trong snapshot không đồng nghĩa trong thực tế không xảy ra sự cố.
* **Giới hạn của số lần chạy:** Đánh giá trên một lần chạy duy nhất (single run) chưa phản ánh toàn diện độ ổn định thống kê (variance) của mô hình; sự sụt giảm ở một vài case ngẫu nhiên như H2 minh họa rõ hiện tượng phương sai suy luận giữa các lượt chạy độc lập.
