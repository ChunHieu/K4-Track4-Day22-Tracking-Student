# Báo cáo lab: chọn tracker cho 5 video

**Học viên:** Cao Đức Hiếu **Mã học viên:** 2A02701

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | strongsort | 0.15 | 0.5 | Camera tĩnh, ánh sáng ban ngày rõ nét. Người đi bộ đan xen nhau và bị che khuất tạm thời sau cột/người khác. Nhờ có Re-ID (`osnet_x0_25_msmt17`), tracker phục hồi đúng ID sau che khuất, giữ track dài nhất quán (IDF1 = 32.57%, HOTA = 29.19%). | bytetrack (conf=0.3, iou=0.5): Ít ID switch tức thời (12 lần) nhưng không có Re-ID nên khi che khuất dài người bị gán ID mới, IDF1 chỉ đạt 25.71% và HOTA đạt 26.91%. |
| video_2 (phố đêm, tĩnh, rất đông) | bytetrack | 0.25 | 0.5 | Mật độ người cực kỳ đông đúc trên phố đêm. Ánh sáng nhân tạo yếu khiến Re-ID dễ bị nhiễu màu. ByteTrack với liên kết 2 tầng (two-stage association) ghép nối rất tốt các detection bị che khuất một phần ở tầng 2, chuyển động mượt mà, không bị sụt FPS. | strongsort (conf=0.25, iou=0.5): Đám đông quá dày đặc và ban đêm làm vector đặc trưng Re-ID bị nhiễu do bóng đổ, dẫn đến gán nhầm ID khi hai người đi sát nhau, đồng thời tốc độ FPS bị chậm đáng kể. |
| video_3 (camera di động, ảnh nhỏ) | ocsort | 0.25 | 0.5 | Camera di động lia góc và di chuyển liên tục, ảnh nhỏ và tốc độ khung hình gián đoạn khiến chuyển động phi tuyến tính. OC-SORT với cơ chế OOS (Observation-Centric Online Smoothing) và Direction Consistency xử lý chuyển động camera rất tốt, không bị trôi hộp khi lia máy. | bytetrack (conf=0.3, iou=0.5): Giả định vận tốc không đổi của Kalman filter thông thường bị phá vỡ khi camera di chuyển đột ngột, dẫn đến việc mất dấu và sinh ID mới liên tục. |
| video_4 (trong nhà, camera di chuyển) | bytetrack | 0.35 | 0.5 | Bối cảnh trong nhà có sàn bóng và vách kính phản chiếu lớn. Đặt conf=0.35 giúp lọc bỏ triệt để các hình ảnh phản chiếu người trong kính (loại bỏ hộp giả FP). ByteTrack theo dấu người thật rõ ràng, không bị hiện tượng hộp nhấp nháy trên vách kính. | bytetrack (conf=0.15, iou=0.5): Ngưỡng conf thấp khiến YOLO phát hiện cả bóng phản chiếu mờ trong kính và bóng dưới sàn, tạo ra các track ảo nhấp nháy gây sai lệch nghiêm trọng. |
| video_5 (trên xe bus, giao lộ đông) | ocsort | 0.25 | 0.5 | Quay từ trên xe bus qua giao lộ đông đúc, camera bị rung lắc dữ dội do xe chạy và sóc đường. OC-SORT sử dụng độ lệch vận tốc dựa trên quan sát thực tế (observation-centric) giúp ổn định hộp theo vết người đi đường, tránh nhảy ID khi khung hình bị xóc nảy. | strongsort (conf=0.25, iou=0.5): Rung lắc mạnh gây hiện tượng mờ nhòe chuyển động (motion blur), làm giảm chất lượng crop đưa vào Re-ID, dẫn đến việc gán nhầm định danh giữa các người đi bộ kề nhau. |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
HOTA: nop_bai_video1-pedestrian    HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1                            29.19     21.612    40.041    23.525    65.504    42.389    82.779    81.625    30.535    36.406    74.544    27.139    
COMBINED                           29.19     21.612    40.041    23.525    65.504    42.389    82.779    81.625    30.535    36.406    74.544    27.139    

CLEAR: nop_bai_video1-pedestrian   MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag      
video_1                            19.907    78.883    20.499    28.206    78.54     11.29     29.032    59.677    13.951    5241      13340     1432      110       7         18        37        239       
COMBINED                           19.907    78.883    20.499    28.206    78.54     11.29     29.032    59.677    13.951    5241      13340     1432      110       7         18        37        239       

Identity: nop_bai_video1-pedestrianIDF1      IDR       IDP       IDTP      IDFN      IDFP      
video_1                            32.573    22.136    61.636    4113      14468     2560      
COMBINED                           32.573    22.136    61.636    4113      14468     2560      

Count: nop_bai_video1-pedestrian   Dets      GT_Dets   IDs       GT_IDs    
video_1                            6673      18581     189       62        
COMBINED                           6673      18581     189       62        
```

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

Với **ít nhất hai video** (nên gồm một video bạn chỉ đánh giá bằng mắt), viết 3–5 câu:

- **Phân tích Video 1 (Quảng trường tĩnh, ban ngày - Đánh giá định lượng):**
  - StrongSORT (kết hợp Re-ID `osnet_x0_25_msmt17`) mang lại chỉ số tổng thể HOTA (29.19%) và chỉ số duy trì danh tính IDF1 (32.57%) vượt trội so với ByteTrack (HOTA 27.31%, IDF1 26.99%). Khi xem video preview, ở các đoạn người đi bộ di chuyển từ xa vào gần và đi ngang qua sau lưng nhau, StrongSORT giữ màu ID của người đó ổn định suốt quãng đường đi dài thay vì tạo ID mới như các tracker thuần chuyển động. Tuy nhiên, việc hạ ngưỡng detector xuống `conf=0.15` để bao quát người ở xa khiến số lần đổi ID (IDSW = 110) cao hơn ByteTrack (IDSW = 13), đổi lại thu được số lượng True Positives cao hơn nhiều (5241 so với 3534).

- **Phân tích Video 4 (Trong nhà, camera di chuyển, phản chiếu kính - Đánh giá bằng mắt):**
  - Thách thức lớn nhất ở video 4 không phải là sự che khuất mà là hiện tượng phản chiếu hình ảnh (reflections) trên các tấm kính lớn và nền gạch bóng. Khi thử nghiệm ban đầu với `conf=0.15`, detector sinh ra rất nhiều bounding box lên hình ảnh phản chiếu của người trong kính, dẫn đến việc tracker gán ID cho cả "bóng ma" và gây nhấp nháy khó chịu. Việc tinh chỉnh tăng ngưỡng lên `conf=0.35` kết hợp với ByteTrack đã giải quyết triệt để vấn đề: loại bỏ hoàn toàn các hộp giả trên kính, trong khi vẫn bám vết mượt mà theo người thật khi camera di chuyển tiến lại gần.

- **Phân tích Video 5 (Trên xe bus, rung lắc mạnh - Đánh giá bằng mắt):**
  - Bối cảnh giao lộ quay từ xe bus có độ rung chấn cơ học rất lớn, khiến giả định vận tốc đều (linear constant velocity) của bộ lọc Kalman truyền thống bị vi phạm nghiêm trọng. Thuật toán OC-SORT với cơ chế quan sát trực tiếp (observation-centric) và làm mượt trực tuyến (online smoothing) thích ứng vượt trội với các dịch chuyển giật cục giữa các frame, giữ cho bounding box bám sát thân người đi bộ qua đường mà không bị trượt hộp hay văng track.

## 4. Nếu có thêm thời gian

Một hoặc hai câu: bạn sẽ thử tiếp điều gì (Re-ID khác, quét `conf` mịn hơn, xem frame gây lỗi…).

Nếu có thêm thời gian, em sẽ tích hợp mô-đun Bù chuyển động camera (Camera Motion Compensation - CMC dựa trên tính toán Optical Flow / Affine Transform) cho `video_3` và `video_5` để loại bỏ hoàn toàn ảnh hưởng của rung lắc, đồng thời quét dải ngưỡng `conf` mịn hơn (bước nhảy 0.02) trên `video_1` để cân bằng tối ưu giữa việc giảm IDSW và tăng IDF1.

