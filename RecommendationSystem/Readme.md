Recommendation System
=================

# 1. Mô tả bài toán
Trong các hệ thống kinh doanh, thương mại thì lợi nhuận đến chủ yếu từ việc khách hàng mua các sản phẩm. Khách hàng thường chỉ sử dụng một vài sản phẩm nhất định và mục đích của công ty là cần kích thích nhu cầu mua sắm, sử dụng của khách hàng bằng cách gợi ý cho người dùng những sản phẩm mà họ chưa mua nhưng có khả năng là họ quan tâm.

# 2. Giới thiệu Dataset
Bộ dữ liệu sử dụng trong bài toán này là [MovieLens](https://grouplens.org/datasets/movielens/). Bộ dữ liệu có nhiều quy mô khác nhau, nhưng vì tài nguyên và thời gian tính toán còn hạn chế nên hiện tại chỉ thực nghiệm với bộ dữ liệu [ml-100k](https://grouplens.org/datasets/movielens/100k/) và [ml-1m](https://grouplens.org/datasets/movielens/1m/).
Các bộ dữ liệu khác nhau sẽ có một số thông tin khác nhau nhưng hầu hết đều chứa thông tin về **user** (người dùng), **item** (bộ phim), **rating** (số sao mà người dùng đánh giá cho bộ phim).
Mô tả sơ bộ về dữ liệu: 

* User: id, tuổi, giới tính, nghề nghiệp, zipcode 
* Item: id, tên phim, ngày phát hành, thể loại
* Rating: user_id, item_id, rating, timestamp


# 3. Các phương pháp giải quyết
Các phương pháp đề xuất:

## 3.1. Content based:
* Ý tưởng chính: người dùng U thích item I1 thì sẽ thích những item I2, I3,... giống với I1, nhưng sự giống nhau giữa các item được so sánh dựa trên những **đặc trưng** từ chính **đặc điểm của item** (như thể loại, diễn viên,...). Do đó phương pháp này có tên là Content based
* Dữ liệu training: Ma trận có kích thước (num_items, num_features). Trong bài toán này, mỗi item có feature là vector mã hóa cho thể loại của item. Đó là vector có số chiều bằng số thể loại, phần tử có giá trị 1 nếu item thuộc thể loại tương ứng, ngược lại có giá trị 0. Một item có thể thuộc nhiều thể loại.
* Qúa trình huấn luyện:
    * Tính độ tương đồng giữa 2 item bất kì bằng độ đo cosine
* Quá trình dự đoán:
    * Input: user U và item I
    * Tìm tập các item X mà user U đã từng rate (có trong training data)
    * Output: Rating dự đoán user U sẽ rate cho item I được tính bằng tổ hợp các rating của các item trong X với trọng số là độ tương đồng giữa item trong X với item I
    * Ví dụ: Cần dự đoán rating của user U cho item I. Tập hợp các (item, rating) mà U đã rate là [(I1, 5), (I2, 4), (I3, 1)] và độ tương đồng giữa 3 item trên với item I là [0.2, 0.3, 0.9]. Khi đó rating dự đoán của (U,I) là (5 * 0.2 + 4 * 0.3 + 1 * 0.9) / (0.2 + 0.3 + 0.9) = 2.21...
## 3.2. Collaborative Filtering:
* Có 2 hướng tiếp cận là User-User và Item-Item
### 3.2.1. User-User CF
* Ý tưởng chính: Nếu người dùng U1 và U2 có sự giống nhau trong cách rating (cùng thích hoặc cùng ghét 1 số item khác, điều này thể hiện U1 và U2 có cùng *gu*) thì nếu U2 thích (ghét) item I thì U1 cũng thích (ghét) item I.
* Quá trình huấn luyện:
    * Tính độ tương đồng giữa 2 user bất kì. Chú ý: Feature vector của mỗi user chính là vector rating của user đó. Do độ dài của vector là số item có trong training data nên với bộ dữ liệu lớn thì không thể tính toán ngay được toàn bộ sự tương đồng của các cặp user. Khi đó ta chỉ tính sự tương đồng giữa 2 user khi cần thiết trong lúc dự đoán và lưu lại để dùng lại khi cần
* Quá trình dự đoán:
    * Input: user U và item I
    * Tìm tập user X đã từng rate cho item I
    * Output: rating dự đoán của user U cho item I được tính bằng tổ hợp các rating của các user X cho item I với trọng số là độ tương đồng giữa user trong tập X và user U
    
### 3.2.2. Item-Item CF    
* Tương tự User-User nhưng thay vì đi tìm các user có cùng *gu* với user U đã từng rate cho item I thì ta đi tìm các item có sự tương đồng với item I mà đã được rate bởi U (sự tương đồng được tính bởi độ tương đồng giữa 2 vector rating của 2 item)
* Cách này rất giống Content based nhưng khác biệt ở chỗ là sự tương đồng giữa các item được so sánh trên 2 vector rating của 2 item
* Ý tưởng chính: Nếu user U thích (ghét) item I1 và item I2 giống *gu* với item I1 (cùng được thích/ghét bởi 1 số user khác) thì U cũng thích (ghét) item I2
# 4. Kết quả thực nghiệm
## 4.1. Kết quả thống kê
* Cấu hình máy: core I3 380M, ram 8GB
* Dataset ml-100k
    * Training size: 90570
    * Test size: 9430
    * Thời gian dự đoán và lỗi RMSE đo trên tập test:

|              | Time (seconds) | RMSE     | 
|--------------|:--------------:|----------|
|Content Based | 22.4684        | 1.096987 |
|User-User CF  | 705.7224       | 1.035468 |
|Item-Item     | 631.0948       | 1.020564 |

* Dataset ml-100m:
    * Training size: 800167
    * Test size: 1000 (do không đủ thời gian, size đầy đủ là 200042)
    * Thời gian dự đoán và lỗi RMSE đo trên tập test:

|              | Time (seconds) | RMSE     | 
|--------------|:--------------:|----------|
|Content Based | 5.5095         | 1.024142 |
|User-User CF  | 1051.3008      | 1.009764 |
|Item-Item     | 490.8624       | 1.020594 |

## 4.2. Nhận xét về kết quả thực nghiệm
* Phương pháp content based có thời gian dự đoán khá nhanh bởi vì quá trình huấn luyện đã tính được sự tương đồng giữa các cặp item (số feature ít hơn nhiều so với việc sử dụng các phương pháp khác (số lượng thể loại so với số user/item))
* Phương pháp Collaborative Filtering nhìn chung cho kết quả chính xác hơn phương pháp Content based

# 5. Một số nhận xét
## 5.1. Đặc điểm của các phương pháp
* Content based
    * Ưu điểm:
        * Có khả năng gợi ý cho user mới chỉ rate 1 vài item
        * Có khả năng gợi ý cho user có *gu* khác biệt
        * Có khả năng gợi ý các item mới hoặc không phổ biến
        * Không cần dữ liệu từ user khác
    * Nhược điểm:
        * Xây dựng feature vector cho mỗi item là khó khăn
        * Không gợi ý được các item nằm ngoài mối quan tâm mà user đã thể hiện với hệ thống (chỉ gợi ý được các item giống với các item mà user đã rate, nhưng mỗi user đều có thể có nhiều thể loại quan tâm khác nữa mà họ chưa thể hiện)
    
* Collaborative Filtering
    * Đặc điểm: Item-Item CF thường cho kết quả tốt hơn User-User bởi vì dễ tìm được các item có sự tương đồng hơn so với việc tìm các user có *gu* tương đồng
    * Ưu điểm:
        * Xây dựng feature vector là dễ dàng (lấy từ rating matrix)
        * Khám phá được các mối quan tâm tiềm ẩn của user dựa trên sự đánh giá của các user khác
    * Nhược điểm:
        * Không gợi ý được các item chưa được rate (item không phổ biến)
        * Không gợi ý được cho các user có *gu* khác biệt
        * Ma trận rating (num_users x num_items) rất lớn nên tốn thời gian tính toán

## 5.2. Một số gợi ý cải tiến
* Trong phương pháp CF thì ma trận rating cần được normalized (trừ rating trung bình) để tính việc tính sự tương đồng bằng cosine là đúng đắn (đã thử cài đặt nhưng kết quả lại tồi hơn, có thể do code sai ở đâu đó)
* Trong tập dataset ml-20m có chứa dữ liệu về feature của mỗi item (chứa giá trị điểm cho các tag), feature này có thể tốt hơn so với feature hiện tại của Content Based (vector mã hóa thể loại) nhưng số chiều lớn hơn nhiều