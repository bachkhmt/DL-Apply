# Giải thích notebook Fashion-MNIST

Notebook được phân tích: `Assignment1/A1_Foundations_FashionMNIST_merged.ipynb`

Notebook xây dựng một pipeline phân loại Fashion-MNIST bằng năm mô hình: Linear Classifier, MLP, CNN, Bidirectional GRU và Transformer.

## Code cell 1 — Cài đặt thư viện

```python
!pip install torch torchvision scikit-learn matplotlib pandas
```

Cell cài các thư viện cần thiết:

- `torch`, `torchvision`: xây dựng mô hình và tải Fashion-MNIST.
- `scikit-learn`: chia dữ liệu và tính các metric.
- `matplotlib`: vẽ biểu đồ.
- `pandas`: tạo và lưu bảng kết quả.

Dấu `!` dùng để chạy lệnh terminal bên trong Jupyter hoặc Google Colab. Mặc dù bình luận nói chỉ bỏ comment khi cần, lệnh hiện đang được bật nên sẽ gọi `pip` mỗi lần chạy cell.

## Code cell 2 — Import và thiết lập môi trường

Cell import các thư viện, đặt seed bằng `42` cho Python, NumPy và PyTorch, rồi chọn GPU nếu có:

```python
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")
```

Tên lớp và số lớp chưa được khai báo tại đây; chúng sẽ được lấy từ dữ liệu trong phần EDA. Việc đặt seed giúp các lần chạy có kết quả gần giống nhau. `torch.nn.functional as F` được import nhưng chưa được dùng trong notebook.

## Code cell 3 — Google Drive và thư mục đầu ra

Nếu chạy trên Colab, cell mount Google Drive và đặt thư mục gốc tại `MyDrive/CO3133_Assignments`. Nếu chạy local, nó sử dụng thư mục hiện tại.

Ba thư mục con được tạo:

- `figures`: lưu hình ảnh và biểu đồ.
- `checkpoints`: lưu trọng số mô hình.
- `results`: lưu bảng kết quả, lịch sử train và prediction.

## Code cell 4 — Thông tin tái lập thí nghiệm

Cell tạo `env_info`, chứa:

- Seed.
- Phiên bản Python.
- Phiên bản PyTorch và torchvision.
- Thiết bị CPU/GPU.
- Tên GPU.
- Thông tin hệ điều hành.

Thông tin này giúp người khác biết môi trường đã dùng để chạy thí nghiệm.

## Code cell 5 — Tải dữ liệu

Cell đầu tiên tải MNIST để debug, nhưng MNIST không được sử dụng trong kết quả chính.

Sau đó nó tải Fashion-MNIST:

- Tập train chính thức: 60.000 ảnh.
- Tập test chính thức: 10.000 ảnh.

Dữ liệu được chuyển sang NumPy:

- `train_images_all`: `(60000, 28, 28)`.
- `train_labels_all`: `(60000,)`.
- `test_images`: `(10000, 28, 28)`.
- `test_labels`: `(10000,)`.

Mỗi ảnh là ảnh xám `28×28`, có kiểu `uint8` và giá trị pixel từ 0 đến 255.

## Code cell 6 — Phân tích phân bố lớp

Cell lấy metadata lớp trực tiếp từ dữ liệu thay vì khai báo sẵn ở đầu notebook:

```python
class_ids, counts = np.unique(train_labels_all, return_counts=True)
CLASS_NAMES = list(fmnist_train_raw.classes)
NUM_CLASSES = len(class_ids)
```

- `class_ids`: các nhãn duy nhất thực sự xuất hiện trong tập train.
- `counts`: số mẫu ứng với từng nhãn.
- `CLASS_NAMES`: tên lớp lấy từ metadata của dataset `FashionMNIST`.
- `NUM_CLASSES`: số lượng nhãn duy nhất tìm thấy trong dữ liệu.

Cell kiểm tra số tên lớp có khớp với số nhãn tìm thấy hay không, rồi tạo bảng phân bố, tính tỷ lệ phần trăm, tỷ lệ mất cân bằng và vẽ biểu đồ cột.

Fashion-MNIST chuẩn có 6.000 ảnh cho mỗi lớp trong tập 60.000 ảnh, nên tỷ lệ giữa lớp lớn nhất và nhỏ nhất dự kiến bằng `1.0`.

## Code cell 7 — Hiển thị mẫu và tính mean/std

Phần đầu tìm ảnh đầu tiên của mỗi lớp và hiển thị thành lưới `2×5`. Hình được lưu với tên `01_eda_representative_samples.png`.

Phần sau chia giá trị pixel cho `255.0` để đưa về khoảng `[0, 1]`, rồi tính:

- `pixel_mean`: trung bình pixel.
- `pixel_std`: độ lệch chuẩn pixel.

Hai giá trị này được sử dụng để chuẩn hóa dữ liệu ở cell tiếp theo.

## Code cell 8 — Chia train, validation và test

Cell dùng `train_test_split` để chia 60.000 ảnh ban đầu theo tỷ lệ chuẩn 80/20:

- Train: 48.000 ảnh (80%).
- Validation: 12.000 ảnh (20%).
- Test chính thức: 10.000 ảnh (tập test độc lập của Fashion-MNIST).

Tham số:

```python
test_size=0.2,
stratify=train_labels_all
```

giúp duy trì tỷ lệ các lớp cân bằng trong cả train và validation. `random_state=SEED` giúp tái tạo đúng cách chia dữ liệu ở các lần chạy sau.

## Code cell 9 — Dataset, augmentation và DataLoader

`ArrayImageDataset` là Dataset tùy chỉnh. Với mỗi chỉ số, nó:

1. Chuyển NumPy array thành ảnh PIL grayscale.
2. Chuyển nhãn thành số nguyên.
3. Áp dụng transform.
4. Trả về cặp `(ảnh, nhãn)`.

Transform dành cho train gồm:

- Lật ngang (`RandomHorizontalFlip`) với xác suất 50% (bảo toàn tính đối xứng vật lý).
- Xoay nhẹ góc ảnh (`RandomRotation`) ngẫu nhiên trong khoảng $\pm 10^\circ$ (giả lập góc chụp nghiêng).
- Padding 2 pixel rồi crop ngẫu nhiên về `28×28` (`RandomCrop`).
- Thay đổi ngẫu nhiên cường độ sáng và độ tương phản $\pm 20\%$ (`ColorJitter(brightness=0.2, contrast=0.2)`).
- Chuyển ảnh thành tensor (`ToTensor`).
- Chuẩn hóa theo mean/std đã tính (`Normalize`).

Validation và test chỉ được chuyển sang tensor và chuẩn hóa, không dùng augmentation.

DataLoader sử dụng batch size 128 cho train và 256 cho validation/test. Train được shuffle, còn validation/test thì không.

Batch ảnh dự kiến có shape `[128, 1, 28, 28]`. Trên một số môi trường Windows/Jupyter, `num_workers=2` có thể gây lỗi tiến trình con; khi đó có thể đổi thành `num_workers=0`.

## Code cell 10 — Các hàm train và evaluation dùng chung

### `count_params(model)`

Đếm tổng số tham số có thể học (`requires_grad=True`) của model.

### `compute_class_metrics(y_true, y_pred, class_names)`

Hàm tính toán chi tiết từng chỉ số đánh giá cho từng class riêng biệt và tổng kết:
- **Từng class (từ 0 đến 9):**
  - **Accuracy (One-vs-Rest):** Tỷ lệ phân loại đúng của lớp đó so với toàn bộ các lớp còn lại.
  - **Precision:** Tỷ lệ số mẫu mô hình dự đoán là class $c$ mà đúng thực tế ($TP / (TP + FP)$).
  - **Recall:** Tỷ lệ số mẫu thực tế thuộc class $c$ mà mô hình phát hiện được ($TP / (TP + FN)$).
  - **F1-Score:** Trung bình điều hòa giữa Precision và Recall của class đó ($2 \cdot P \cdot R / (P + R)$).
  - **Support:** Tổng số mẫu thực tế thuộc class đó.
- **Hàng tổng kết cuối bảng (Macro Average):**
  - Tính trung bình số học không trọng số của Accuracy, Precision, Recall và F1-Score trên toàn bộ 10 lớp, kèm theo Overall Accuracy của toàn tập dữ liệu.

### `fit(model, dataloader, criterion, optimizer, device)`

Chạy một epoch huấn luyện trên tập train:
1. Đặt `model.train()`.
2. Duyệt qua từng batch trong DataLoader: reset gradient (`optimizer.zero_grad()`), tính forward (`outputs = model(images)`), tính loss, lan truyền ngược (`loss.backward()`) và cập nhật trọng số (`optimizer.step()`).
3. Thống kê loss và gom các nhãn dự đoán để tính loss trung bình cùng bộ 4 chỉ số: **Accuracy, Macro-Precision, Macro-Recall, và Macro-F1**.

### `evaluate(model, dataloader, criterion, device)`

Đánh giá mô hình trên tập validation hoặc test:
1. Sử dụng decorator `@torch.no_grad()` và đặt `model.eval()`.
2. Chỉ tính forward và loss, không tính gradient để tiết kiệm bộ nhớ và tăng tốc.
3. Trả về loss trung bình, Accuracy, Macro-Precision, Macro-Recall, Macro-F1, nhãn thật và nhãn dự đoán.

### `run_one_epoch(model, train_loader, val_loader, criterion, optimizer, device)`

Hàm điều phối trong một epoch: lần lượt gọi `fit()` cho tập train và `evaluate()` cho tập validation, trả về bộ chỉ số đầy đủ (loss, acc, prec, rec, f1) của cả 2 tập.

### `run_epoch(...)`

Wrapper tương thích ngược nhận tham số `optimizer` để tự động chuyển tiếp sang `fit()` (nếu có optimizer) hoặc `evaluate()` (nếu optimizer là None), giúp các cell đánh giá test và nạp lại checkpoint hoạt động liền mạch với đầy đủ các metric.

### `train_model(...)`

Hàm huấn luyện hoàn chỉnh một kiến trúc mạng:
- Sử dụng `CrossEntropyLoss` (nhận raw logits, không qua softmax).
- Optimizer Adam và scheduler `ReduceLROnPlateau` để giảm learning rate khi validation loss chững lại.
- Gọi `run_one_epoch()` trong từng epoch và in kết quả ra màn hình (không dùng thư viện thanh tiến trình tqdm để giữ code gọn gàng).
- Cơ chế Early Stopping với `patience=3` để dừng sớm khi val_loss không cải thiện, và nạp lại trọng số tốt nhất trước khi trả về.

### `measure_inference_time(...)`

Hàm chạy warm-up 2 batch đầu, sau đó đo thời gian inference thực tế trên toàn bộ test loader (đồng bộ hóa GPU bằng `torch.cuda.synchronize()`) và trả về độ trễ trung bình tính theo millisecond cho mỗi ảnh.

## Code cell 11 — Linear Classifier

Kiến trúc:

```text
Ảnh 1×28×28 → flatten 784 → Linear 10
```

Model biến mỗi ảnh thành vector 784 phần tử rồi dùng một lớp tuyến tính để tạo 10 logits. Đây là baseline đơn giản và không khai thác trực tiếp cấu trúc không gian của ảnh.

## Code cell 12 — MLP

Kiến trúc mặc định:

```text
784 → 256 → ReLU → Dropout
    → 128 → ReLU → Dropout
    → 10 logits
```

`ReLU` cung cấp tính phi tuyến. `Dropout(0.3)` tắt ngẫu nhiên 30% activation khi train để giảm overfitting.

MLP mạnh hơn Linear Classifier nhưng vẫn flatten ảnh nên làm mất cấu trúc không gian 2D.

## Code cell 13 — CNN

Phần trích xuất đặc trưng có cấu trúc:

```text
1×28×28
→ Conv 32 → BatchNorm → ReLU → MaxPool
→ 32×14×14
→ Conv 64 → BatchNorm → ReLU → MaxPool
→ 64×7×7
→ Conv 128 → BatchNorm → ReLU
```

Phần phân loại gồm Adaptive Average Pooling, Flatten, Dropout và Linear.

- Convolution học cạnh, họa tiết và hình dạng.
- BatchNorm giúp quá trình train ổn định.
- MaxPool giảm kích thước không gian.
- Adaptive Average Pooling biến mỗi feature map thành một giá trị.

CNN có inductive bias phù hợp với ảnh và thường hoạt động tốt trên Fashion-MNIST.

## Code cell 14 — Bidirectional GRU

Ảnh được xem như chuỗi 28 timestep; mỗi timestep là một hàng gồm 28 pixel.

```python
x = x.squeeze(1)
```

Shape đổi từ `(N, 1, 28, 28)` thành `(N, 28, 28)`.

GRU hai chiều đọc các hàng từ trên xuống dưới và từ dưới lên trên. Với `hidden_size=128`, đầu ra có chiều 256. Cell lấy đầu ra ở timestep cuối rồi đưa qua Dropout và Linear để phân loại.

Lưu ý: với bidirectional GRU, lấy riêng `out[:, -1, :]` không tổng hợp hai hướng một cách hoàn toàn đối xứng. Dùng hidden state cuối của từng hướng thường rõ ràng hơn.

## Code cell 15 — Patch Transformer

Ảnh `28×28` được chia thành các patch `4×4`, tạo thành `7×7 = 49` patch. Mỗi patch có 16 pixel.

Quy trình:

1. `unfold` tách ảnh thành patch.
2. Flatten mỗi patch thành vector 16 phần tử.
3. Chiếu mỗi vector từ 16 chiều lên `d_model=64`.
4. Thêm token học được `[CLS]`.
5. Cộng positional embedding.
6. Đưa token qua ba Transformer Encoder layer.
7. Lấy biểu diễn của `[CLS]`.
8. Dùng Linear để tạo 10 logits.

Positional embedding giúp model biết vị trí của các patch trong ảnh.

## Code cell 16 — Huấn luyện cả năm model

`MODEL_BUILDERS` ánh xạ tên model tới hàm khởi tạo. Cấu hình chung gồm:

- Tối đa 15 epoch.
- Learning rate `0.001`.
- Early-stopping patience bằng 3.
- Một seed cho mỗi model.

Với mỗi model, cell:

1. Đặt seed.
2. Khởi tạo model.
3. Đếm tham số.
4. Train model.
5. Đánh giá trên test.
6. Đo thời gian inference.
7. Lưu model, prediction và history.
8. Lưu checkpoint.
9. Thêm metric vào bảng tổng kết.

`results_df` được sắp xếp theo test accuracy giảm dần.

Về phương pháp luận, notebook đánh giá cả năm model trên test rồi dùng kết quả test để xếp hạng. Quy trình nghiêm ngặt hơn là chọn kiến trúc và hyperparameter bằng validation, sau đó chỉ đánh giá test một lần cho model đã chọn.

## Code cell 17 — Nạp lại checkpoint

Hàm `load_and_evaluate`:

1. Đọc checkpoint.
2. Tạo lại đúng kiến trúc.
3. Nạp `state_dict`.
4. Đánh giá lại trên test.
5. In accuracy và macro-F1.

Dòng cuối chọn model đứng đầu `results_df` để kiểm tra rằng checkpoint đã lưu có thể tái tạo kết quả.

## Code cell 18 — So sánh định lượng

Cell in bảng kết quả và vẽ ba biểu đồ:

1. Accuracy và Macro-F1.
2. Số tham số có thể học.
3. Thời gian inference trên mỗi mẫu.

Trục số tham số sử dụng logarithmic scale. Hình được lưu thành `12_model_comparison.png`.

Accuracy và macro-F1 hiện được vẽ tại cùng vị trí, nên hai cột bị chồng lên nhau thay vì đứng cạnh nhau.

## Code cell 19 — Training/validation curves

Với mỗi model, cell vẽ:

- `train_loss`.
- `val_loss`.

Biểu đồ giúp nhận biết:

- Overfitting: train loss giảm nhưng validation loss tăng.
- Underfitting: cả hai loss vẫn cao.
- Early stopping xảy ra sau bao nhiêu epoch.

Hình được lưu thành `12_training_curves.png`.

## Code cell 20 — Confusion matrices

Cell tính ma trận nhầm lẫn cho từng model bằng `confusion_matrix(y_true, y_pred)`. Ma trận cho biết lớp thật nào thường bị model dự đoán nhầm thành lớp nào.

Đoạn sau có vấn đề:

```python
ax.set_xticks(range(NUM_CLASSES)); ax.set_xticks([])
ax.set_yticks(range(NUM_CLASSES)); ax.set_yticks([])
```

Lệnh thứ hai xóa các tick vừa tạo. Có thể sửa thành:

```python
ax.set_xticks(range(NUM_CLASSES))
ax.set_yticks(range(NUM_CLASSES))
ax.set_xticklabels(CLASS_NAMES, rotation=90)
ax.set_yticklabels(CLASS_NAMES)
```

## Code cell 21 — Ví dụ dự đoán đúng và sai

Cell chọn model có test macro-F1 cao nhất, rồi tìm:

- `correct_idx`: vị trí dự đoán đúng.
- `wrong_idx`: vị trí dự đoán sai.

Hàm `show_examples` lấy ngẫu nhiên tối đa tám ảnh. Tiêu đề của mỗi ảnh hiển thị:

- `T`: nhãn thật.
- `P`: nhãn dự đoán.

Hai hình được lưu thành `14_qualitative_correct.png` và `14_qualitative_incorrect.png`.

Vì test loader không shuffle, thứ tự trong `y_true` và `y_pred` khớp với `test_images`.

## Code cell 22 — Tóm tắt khả năng tái lập

Cell in lại:

- Seed.
- Phiên bản thư viện.
- Thiết bị.
- Cách chia dữ liệu.
- Quy tắc chọn checkpoint.
- Danh sách checkpoint đã tạo.

Thông tin này hỗ trợ tái lập thí nghiệm.

## Code cell 23 — Lưu kết quả

Cell lưu:

1. Bảng kết quả thành CSV.
2. Training histories thành pickle.
3. Trọng số của từng model.
4. Nhãn thật và prediction test.

Cell này có xung đột định dạng checkpoint. Cell 16 lưu checkpoint dưới dạng dictionary:

```python
{
    "state_dict": ...,
    "seed": ...,
    "params": ...,
    "class_names": ...,
    "model_name": ...
}
```

Nhưng cell 23 ghi đè cùng file bằng raw `state_dict`:

```python
torch.save(d["model"].state_dict(), ckpt_path)
```

Sau khi chạy cell 23, `load_and_evaluate` ở cell 17 sẽ lỗi vì nó vẫn tìm `ckpt["state_dict"]` và `ckpt["seed"]`. Nên giữ cùng một định dạng hoặc lưu raw state dict với tên file khác.

## Các vấn đề còn cần lưu ý

1. Code cell 20 tự xóa tick của confusion matrix.
2. Code cell 23 ghi đè checkpoint bằng định dạng không tương thích với code cell 17.

Hai vấn đề này không liên quan đến cách lấy thông tin lớp trong EDA, nhưng ảnh hưởng đến phần hiển thị kết quả và khả năng nạp lại model.
