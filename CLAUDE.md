# CLAUDE.md

## Tổng quan
CHF-MIMIC-IV là bài tập lớn môn Data Mining: dự đoán nguy cơ tử vong (`target`) ở bệnh nhân suy tim (CHF) từ dữ liệu cận lâm sàng/bệnh nền dẫn xuất từ **MIMIC-IV**. Quy trình: KNNImputer điền thiếu → K-Means chia 2 cụm → huấn luyện **một mô hình CatBoost cho mỗi cụm** → suy luận bằng cách gán bệnh nhân mới vào cụm gần nhất rồi dùng mô hình của cụm đó.

## Cấu trúc
- `Code.ipynb` — pipeline đầy đủ (chia 80/20, KNNImputer, Elbow/Silhouette chọn 2 cụm, huấn luyện CatBoost).
- `retrain.py` — huấn luyện lại 2 mô hình từ một CSV mới, ghi `retrain_log.txt` và `catboost_model_cluster_{0,1}.cbm`.
- `Infer.py` — suy luận một dòng CSV: `python Infer.py <file.csv> <chỉ_số_dòng_từ_0>`.
- `Infer_GUI.py` (Tkinter) và `web_app.py` (Flask) — giao diện chạy suy luận trên file CSV mẫu.
- `catboost_model_cluster_*.cbm`, `cluster_*.csv`, `top_features_cluster_*.csv`, `data_80_imputed_no_id.csv` — mô hình và dữ liệu tham chiếu để gán cụm/điền thiếu.
- `final_data_first.csv`, `data_10_*.csv`, `data/` — dữ liệu gốc đã tiền xử lý (có cột `ID`).
- `.github/workflows/retrain_model.yaml` — khi push vào `data/**`, CI chạy `retrain.py` và tự commit lại log + model (dùng secret `GH_TOKEN`, git user JKLover0909).

## Môi trường
Python 3.8+ (CI dùng 3.8): `pandas numpy scikit-learn catboost flask` (xem `requirements.txt`). Các script đang trỏ đường dẫn dạng `BTL_Mining/...`, tức chạy từ thư mục cha của thư mục repo (đổi tên/clone thành `BTL_Mining`) — cần chỉnh nếu muốn chạy từ gốc repo.

## Quy tắc khi làm việc
- **Dữ liệu y tế:** các CSV có cột `ID` mang mã bệnh án thật của MIMIC-IV. Điều khoản PhysioNet (Data Use Agreement) hạn chế việc chia sẻ dữ liệu mức bệnh nhân công khai. Không thêm dữ liệu mới có `ID` vào repo public; ưu tiên dữ liệu đã ẩn danh/tổng hợp.
- Không sửa `*.cbm`, `retrain_log.txt` bằng tay — do CI sinh ra khi push `data/**`.
- Không commit `nohup.out`, `logout.txt`, `catboost_info/` (đã có trong `.gitignore`), cũng như token/khóa.
- Mô hình chỉ phục vụ học tập, không phải công cụ chẩn đoán lâm sàng; kết quả hiện có recall thấp ở lớp dương (xem `retrain_log.txt`/notebook).
