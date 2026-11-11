# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Ngọc Sang

MSSV: 2A202603444

## 1. Case được chọn

- ID case: H2
- Câu trả lời của agent: 
    - Agent đã bác bỏ tiền đề sai, xác nhận GitHub Actions đang operational tại snapshot ngày 09/11/2026 lúc 03:50 UTC và incident t89g2q1800fr đã resolved.

    - Tuy nhiên, Agent chỉ nêu ngày tạo incident là 05/11, chưa nêu chính xác giờ tạo và chưa trình bày đầy đủ các mốc chuyển trạng thái investigating, monitoring và resolved.
- Điểm ở lần chạy 1: 96.91/100

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
{
  "event": "tool",
  "step": 2,
  "name": "get_incident_details",
  "arguments": {
    "provider": "github",
    "incident_id": "t89g2q1800fr"
  },
  "result": {
    "status": "resolved",
    "created_at": "2026-11-05T16:50:11.086Z",
    "updated_at": "2026-11-05T17:42:35.455Z",
    "updates": [
      {
        "status": "resolved",
        "created_at": "2026-11-05T17:42:35.425Z"
      },
      {
        "status": "monitoring",
        "created_at": "2026-11-05T17:04:46.807Z"
      },
      {
        "status": "investigating",
        "created_at": "2026-11-05T16:50:11.189Z"
      }
    ]
  }
}
```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Bổ sung các quy tắc trong SYSTEM_PROMPT tại file agent.py, cụ thể là quy trình kiểm tra trạng thái và quy tắc diễn giải thời gian
- Vì sao chọn thay đổi này: Vì Judge chỉ ra rằng câu trả lời lần đầu còn thiếu giờ tạo incident và diễn biến các trạng thái. Việc bổ sung quy tắc giúp Agent khai thác dữ liệu tool chính xác và trình bày timeline đầy đủ hơn, thay vì chỉ trả lời trạng thái cuối cùng.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 96.91/100	 | 97.95/100 |
| Điểm case đã chọn | 96/100 | 100/100  |
| Số model calls | 69 | 71 |
| Token, chi phí nếu có | 65,912 | 72,611 |

- Lỗi đã giải quyết:
     - Agent đã trình bày đầy đủ hơn các mốc thời gian của incident.
     - Phân biệt rõ incident lịch sử đã resolved với trạng thái hiện tại của GitHub Actions.
- Lỗi còn lại hoặc mới xuất hiện:
     - Case H5 giảm từ 96 xuống 92 điểm.
     - Một số case khác cũng có điểm dao động giữa hai lần chạy, ví dụ E1 giảm từ 91 xuống 85 và   E5 tăng từ 88 lên 100.

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.

- Giới hạn của LLM Judge: Điểm được chấm bởi LLM Judge, vì vậy kết quả có thể bị ảnh hưởng bởi cách diễn giải, mức độ nghiêm ngặt và sự biến thiên giữa các lần đánh giá. Điểm số không hoàn toàn đại diện cho chất lượng thực tế của Agent.

- Giới hạn của snapshot: 

     - Dữ liệu trạng thái được ghi nhận tại thời điểm 2026-11-09 03:50 UTC. Vì vậy, kết quả chỉ phản ánh trạng thái tại thời điểm snapshot, không đảm bảo GitHub hoặc Cloudflare vẫn giữ trạng thái đó ở thời điểm khác.

     - Ngoài ra, công cụ tìm kiếm incident chỉ kiểm tra 50 incident gần nhất. Việc không tìm thấy incident phù hợp không chứng minh rằng incident đó chưa từng tồn tại.

- Việc chỉ chạy một lần mỗi phiên bản: Mỗi phiên bản chỉ được chạy Eval một lần. Do đó, chưa đủ cơ sở để kết luận mức tăng điểm hoàn toàn đến từ thay đổi System Prompt. Kết quả còn có thể chịu ảnh hưởng từ sự biến thiên của model và Judge.