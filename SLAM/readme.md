# 3D SLAM main

## usage:

- workon envPy2DL (virtual env python - the same with env that used in PSMNet code)
- dev on home PC:   
    `./main.py --dataset "/media/huyhv/My Passport/1.3DVision/2.Data/3DSlamData/" --city "aachen_dev"`  
`./main.py --dataset "/home/huyhv/3DSlamData/" --city "aachen_dev"`

- dev on cist PC:  
    `./main.py --dataset "/media/huynv/Data/14.ComputerVision/3.Data/3DSlamData/" --city "aachen_dev"`

open terminal to view log debug file:
`3D_SLAM_HustAIS/SLAM/.log$ tail -f log.log`

Using this command to test depth:  
`/3D_SLAM_HustAIS/depthEstimate$ ./objectPosFromDisparity.py --disp ../../../3.Data/3DSlamData/aachen_dev/disparityPSMNet/  aachen_000000_000019_leftImg8bit.png --leftColor ../../../3.Data/3DSlamData/aachen_dev/leftImg/aachen_000000_000019_leftImg8bit.png `

Aug 13, 2019: 
- Checked depth

## Some points to note:
- object's format: `[xmin, ymin, xmax, ymax]`
- landmarks in Camera coordinate format: [[X, Z, _object]...]
    - X: Tọa độ theo phương ngang (met)
    - Z: Chiều sâu tính từ gốc Camera 
- landmarks in Vehicle coordinate format: [[x_pv, y_pv, _object]...]
    - x_pv: Chiều sâu tính từ Vehicle coordinate
    - y_pv: Tọa độ theo chiều ngang ảnh (trong vehicle coordinate)

### Áp dụng bộ lọc:
- Đối tượng thuộc label: "traffic light", "pole"
- Số đỉnh của đối tượng từ file gtFine.json < 6
- depth < 30 
- Object nằm trong vùng từ: x = imgWidth/20 tới imgWidth*19/20 (giữa hai đường màu xanh)
- Diện tích của bounding box > 500 (px^2 - Tính theo pixel)

### Nội dung ảnh - tên:
- *_objects.png: Vẽ bouding box toàn bộ object
- *_landmarked_Cam.png: Chứa thông tin về diện tích và chiều sâu của toàn bộ objects
- *_landmarked_Vehicle: Chứa thông tin về tọa độ [X_pv, Y_pv] của các objects sau khi áp dụng bộ lọc.

g2o file format:  https://github.com/RainerKuemmerle/g2o/wiki/File-Format-SLAM-2D 


## 14/08/2019:
### Cây thư mục kết quả:
- Ảnh dãn landmark: ./log/landmarked/
- file g2o: g2o/{args.city}.g2o
### features:
- Chưa có phát hiện landmark trùng
- Cơ chế tạo file g2o chưa thể dùng nếu có landmark trùng.

## 23 Aug, 2019:
#TODO:
- Tạo thêm bộ dữ liệu dài hơn (stuttgart_01)
    - Chạy PSMNet:
        - Tạo bộ left, right trên pc thầy Hạ
        - Chuyển lên aisgpu2 để chạy PSMNet
        - Sửa code sửa đuôi file kết quả trên aisgpu2
        - Kéo kết quả về máy thầy Hạ
        - Kéo về máy Huy
    - Chạy object detection 
        - Đẩy dữ liệu lên aisgpu4 
        - Chạy object detection 
        - Kéo file *_detected.json về máy 

- Tăng bộ nhớ cache lên n frame
    - Update vị trí landmark trong cache
    - Gắn thêm chỉ số frame cho landmark, sau id -> []
    - update chỉ số frame nếu có landmark trùng
    - Nếu chỉ số frame > number_cache thì xóa.