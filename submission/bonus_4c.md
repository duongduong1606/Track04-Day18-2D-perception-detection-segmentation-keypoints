# Bonus 4C — Kiểm tra `flip_idx` bằng validation lật gương

## Thiết lập

Hai model YOLO26n-pose được train 40 epoch, `imgsz=640`, cùng seed và cấu hình trên RTX 3060:

- **Giải phẫu:** lật ngang đồng thời hoán đổi từng cặp keypoint `left_*` ↔ `right_*`.
- **Đồng nhất:** giữ `flip_idx = [0, 1, ..., 11]`, tức nhãn trái/phải không đổi sau khi lật.

Tập val lật gương được tạo từ 53 ảnh val gốc bằng cách lật ảnh, đổi `x → 1 - x` và hoán đổi nhãn trái/phải theo quy ước giải phẫu.

## Kết quả

| Model | Pose mAP50-95 — val gốc | Pose mAP50-95 — val lật gương | Mức giảm tuyệt đối |
|---|---:|---:|---:|
| `flip_idx` giải phẫu | 0,401 | 0,297 | 0,104 |
| `flip_idx` đồng nhất | 0,421 | 0,283 | 0,138 |

## Nhận xét

Nếu chỉ nhìn val gốc, metric không những không phát hiện bug mà còn xếp model `flip_idx` đồng nhất cao hơn 0,020 mAP. Nguyên nhân là toàn bộ train (210/210) và val gốc (53/53) đều có hổ quay phải, nên metric chỉ đo trên một phía của phân phối triển khai. Model có nhãn augmentation sai vẫn có thể khớp tập val thiên lệch này.

Trên val lật gương, model đồng nhất giảm 0,138, nhiều hơn model giải phẫu (giảm 0,104), và kết quả cuối thấp hơn 0,014 mAP. Cả hai model đều giảm do ảnh quay trái là distribution shift, nhưng khoảng giảm lớn hơn của model identity cho thấy quy ước lật sai làm khả năng tổng quát hóa kém hơn.

Tập validation triển khai nên cân bằng hổ quay trái/phải, gồm cả ảnh lật có nhãn đúng và ảnh quay trái thật, đồng thời báo cáo metric theo từng nhóm hướng quay. Việc tách slice metric này ngăn mAP trung bình che mất lỗi hệ thống ở nhãn trái/phải.
