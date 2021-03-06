
# Depth Estimate from Stereo cameras
### Tham khảo:
* Depth estimation from stereo cameras (pdf)
### Disparity estimation 

- Nội dung của stereo matching là dựa trên việc tìm sự tương thích giữa hai ảnh đầu vào. 
Trong bài này, việc việc khớp gwiax hai điểm được xem xét bằng việc kiểm tra nxn pixel láng giềng N quanh cả 2 điểm đấy.
- Matching block được đặt cho mỗi pixel trong một hình ảnh. 
- Sự khác nhau trong vị trí của các ddierm trong các mặt phẳng ảnh là *disparity* của điểm đó.
- Matching block size là một thông số quan trọng ảnh hưởng tới kết quả của việc ước tính disparity. Block nhỏ hơn có thể tìm kiếm chi tiết tốt hơn, nhưng dễ có lỗi, trong khi block lớn hơn thì robust hơn nhưng lại bỏ qua các chi tiết nhỏ. 


### Depth from disparity
- Disparity không giống depth.
Bỗi quan hệ giữa disparity và depth dựa trên cấu hình camera 
- Công thức: `depth = baseline * focal / disparity`
![cong thuc](https://raw.githubusercontent.com/Huyhust13/3D_SLAM_HustAIS/master/figures/stereo_depth.png?token=AIFPE4M3KDRG46SAVAXI7O25GZ4LU) 


    - (fx,fy) - Focal length in pixels.  

        * fx=F/px  
        * fy=F/py  

    - F - Focal length in world units, typically expressed in millimeters.  
    - (px,py) — Size of the pixel in world units.

#### Ví dụ:
```
*To obtain depth, you need to convert disparity using the following formula:
depth = baseline * focal / disparity
For KITTI the baseline is 0.54m and the focal ~721 pixels. 
The relative disparity outputted by the model has to be scaled by 1242 which is the original image size.
The final formula is:
depth = 0.54 * 721 / (1242 * disp)*
```
<img align="center" src="https://raw.githubusercontent.com/Huyhust13/3D_SLAM_HustAIS/master/figures/vd_depthCitycapes.png">

Các vấn đề:
- Phải đọc được đúng định dạng dữ liệu từ disparity map. 
    Giá trị maximum của disparity thể hiện bằng D.
    Với PSMNet thì Maximum disparity là 192 ==> Quy định định dạng biến đọc từ opencv.
- Tìm được đúng thông số camera (đơn vị đo của thông số)


## Phương pháp
1. Dùng bộ dữ liệu stereo (ApolloDataset hoặc CityscapesDataset), chạy qua PSMNet để ra dispiraty map.
2. Từ dispiraty map, kết hợp với thông số camera để tính ra depth map
3. Depth map + object detected -> Dense point cloud hay gì???

- Mạng đã chạy được
- Dữ liệu camera:
    - Citicape có bộ dữ liệu camera:
    ```
    {
        "extrinsic": {
            "baseline": 0.209313, 
            "pitch": 0.038, 
            "roll": 0.0, 
            "x": 1.7, 
            "y": 0.1, 
            "yaw": -0.0195, 
            "z": 1.22
        }, 
        "intrinsic": {
            "fx": 2262.52, 
            "fy": 2265.3017905988554, 
            "u0": 1096.98, 
            "v0": 513.137
        }
    }
    ```
    Trong đó: *baseline (m), fx, fy (pixel)*. 

    fx, fy là gì???


## Các bộ dữ liệu:
1. ApoloScapesDataset: 
* [Link download](http://apolloscape.auto/stereo.html#to_metric_href)
* [git repository](https://github.com/ApolloScapeAuto/dataset-api/tree/master/stereo)

Dữ liệu:
- Ảnh stereo
- Ảnh disparity
- Ảnh depth

==> Tập Stereo_train_001 (4.4GB) có đầy đủ thông tin trên để tính depth, nhưng bộ dữ liệu này không có ảnh depth được tính sẵn.
intrinsic.txt
```
K = [2301.3147, 0, 1489.8536; 0, 2301.3147, 479.1750; 0, 0, 1]
```
==> focal = 2301.3147 (pixel)


2. Cityscapes 
* [Link download](https://www.cityscapes-dataset.com/downloads/)
* [git repository](https://github.com/mcordts/cityscapesScripts)

Dữ liệu:
- Có dữ liệu để train nhận dạng cột đèn giao thông
- Có disparity
- Có thông số camera
- Không có ảnh depth có sẵn

- Thông số camera:
    - disparity precomputed disparity depth maps. To obtain the disparity values, compute for each pixel p with p > 0: d = ( float(p) - 1. ) / 256., while a value p = 0 is an invalid measurement. Warning: the images are stored as 16-bit pngs, which is non-standard and not supported by all libraries.


------------
## Một số opensource
### [PSMNet](https://github.com/JiaRenChang/PSMNet)
- Thử chạy file submission.py với dữ liệu đã train sẵn *.tar
- Hi vọng có thể sử dụng mô hình đã train, chế biến lại code submission.py để  dùng luôn.

10/7/2019:
- Code submission.py đang chạy trên cuda, gặp một số lỗi khi chạy trên cpu

--> Code này cần CUDA

Train PSMNet:
- KITTY15: bộ ảnh thật, dataset street view từ xe ô tô. Chứa 200 ảnh stereo khớp với ảnh disparity ground-truth thưa dùng LiDAR và 200 ảnh test khác không có ảnh LiDAR. Kích thước ảnh 376*1240

```
The PSMNet architecture was implemented using PyTorch. All models were end-to-end trained with Adam
(β1 = 0:9; β2 = 0:999). We performed color normalization on the entire dataset for data preprocessing. During
training, images were randomly cropped to size H = 256
and W = 512. The maximum disparity (D) was set to 192.
We trained our models from scratch using the Scene Flow
dataset with a constant learning rate of 0.001 for 10 epochs.
For Scene Flow, the trained model was directly used for
testing. For KITTI, we used the model trained with Scene
Flow data after fine-tuning on the KITTI training set for 300
epochs. The learning rate of this fine-tuning began at 0.001
for the first 200 epochs and 0.0001 for the remaining 100
epochs. The batch size was set to 12 for the training on four
nNvidia Titan-Xp GPUs (each of 3). The training process
took about 13 hours for Scene Flow dataset and 5 hours for
KITTI datasets. Moreover, we prolonged the training process to 1000 epochs to obtain the final model and the test
results for KITTI submission.
```

- Dữ liệu để sử dụng với model pre-trained cần có H và W chia hết cho 32.
`If you use your own dataset, the width and height of image should be divisible by 32.`

- Ảnh disparity map tương ứng với ảnh trái.
    `For the pixel (x; y) in the
left image, if its corresponding point is found at (x − d; y)
in the right image, then the depth of this pixel is calculated
by fB d , where f is the camera's focal length and B is the
distance between two camera centers.`


### [DenseDepth](https://github.com/ialhashim/DenseDepth)
12/7/2019: 
DenseDepth từ [link này](https://github.com/ialhashim/DenseDepth).

Tập trung vào Depth môi trường đường phố ngoài trời.
Sử dụng model đã train sẵn: [KITTI](https://s3-eu-west-1.amazonaws.com/densedepth/kitti.h5)

Chạy `python test.py` với model kitti.h5 và ảnh từ KITTI dataset (phải resize về 348x1248)

---
### Thắc mắc:
- Input: 
    - Model train sẵn
    - Ảnh
- Output: 
    - Chiều sâu???

Vì sao chỉ 1 ảnh lại có thể ra đc chiều sâu???

### Cài cuda và sử dụng trên venv
1. Cài cuda  
https://medium.com/@kapilvarshney/how-to-setup-ubuntu-16-04-with-cuda-gpu-and-other-requirements-for-deep-learning-f547db75f227

Kich hoat cuda:
```
# env cuda
export PATH=$PATH:/usr/local/cuda-10.1/bin
export CUDADIR=/usr/local/cuda-10.1
```

2. Tạo môi trường venv để chạy laị code submission.py
3. Cài các gói cần thiết: 
`pip install -r requirements.txt`
- Kiểm chứng kết quả đo???

---------
## Các bước dự kiến (update: 24/07/2019)
1. Chạy lại file submission.py chạy mạng PSMNet để ra disparity map.
    - Đã ok. Kết quả tạo ra trong 21 file trong thư mục disparity/tmp_1024x512 (resize từ 2024x1024 <-> giảm 2 lần mỗi chiều)
2. Tính depth và tọa độ từ ảnh disparity.
    - Đã tính được với ảnh disparity Cityscape (file depthEstimate/depthFromDisparity với source PSMNet/disparity/tmp_1024x512) nhưng kq có vẻ chưa đúng lắm. Chuyển sang phần 3 để kiểm tra ảnh depth.
    - Usage:
    `./depthFromDisparity.py --image ../PSMNet/disparity/tmp_1024x512/berlin_000002_000019_leftImg8bit.png`
    - Ảnh depth trông có vẻ khá ổn (khi xét sự tương quan giữa các đối tượng trong ảnh với nhau). Còn giá trị depth thì có vẻ chưa chính xác lắm
    - Các yếu tố có thể ảnh hưởng tới độ chính xác khi đọc depth:
        - Thông số camera: baseline, **focal**
        - Định dạng ảnh disparity khi lưu -> _Đã xác thực, không sai ở phần này.
        - Các hệ số chuyển đổi...

    - **Xác thực định dạng:**
        - Ở file submission.py lưu disparity với lệnh:
            - `skimage.io.imsave(dispmap,(img*256).astype('uint16'))`
            - Giá trị mỗi pixel dc nhân lên 256 lần
            - Định dạng uint16
        - Đọc file 
            - Chế độ đọc trong opencv có
            `CV_LOAD_IMAGE_ANYDEPTH - If set, return 16-bit/32-bit image when the input has the corresponding depth, otherwise convert it to 8-bit.`
            -> Đọc bằng ANYDEPTH sẽ đọc đúng định dạng unit16. Sau đó chia cho 256 là dc gía trị đúng của disparity.
        - **Nhưng:**
            - Ảnh (1024x512) là đã downscale xuống 1 nửa, do đó disprity (độ chênh lệch pixel) sẽ bằng 1 nửa so với ảnh gốc (2048x1024). Do đó khi tính phải x2 sau đó mới chia cho 256 --> kq tương đồng với disparity mà dataset cho.
3. Kiểm tra độ chinh xác khi tính depth (ảnh hưởng từ disparity, định dạng dữ liệu, thông số camera):
    - Dùng bộ apollo có ảnh depth (down xuống mỗi cạnh 4 lần)
        - Kết quả khi chạy ảnh apolo down ảnh xuống còn 832x672 thì kq tệ. Dạo quanh các diễn đàn thì thấy họ dùng bộ apolo để train lại chứ k dùng pre-trained model của KITTY.
    - Khi so sánh giữa đọc depth từ disparity của cityscape cho sẵn với ảnh disparity tính được thì depth từ ảnh sẵn có vẻ bằng 1 nửa so với ảnh tính ra và có vẻ hợp lý hơn.
4. Kết nối với object detection:
 - Lấy depth khi có bouding box:
    - Ví dụ chương trình 3D object detection sử dụng Intel realsense lấy depth của object bằng cách lấy depth của tâm bounding box.
    [Link](https://github.com/PINTO0309/MobileNet-SSD-RealSense/blob/master/MultiStickSSDwithRealSense_OpenVINO_NCS2.py)
    ```python
    # 0:=RealSense Mode, 1:=USB Camera Mode
    if camera_mode == 0:
        meters = depth_dist.get_distance(box_left+int((box_right-box_left)/2), box_top+int((box_bottom-box_top)/2))
        label_text = LABEL[int(class_id)] + " (" + str(percentage) + "%)"+ " {:.2f}".format(meters) + " meters away"
    ```
3. Nối thông từ chạy ảnh stereo ra depth.

Work-flow:
<img align="center" src="https://raw.githubusercontent.com/Huyhust13/3D_SLAM_HustAIS/master/figures/work-fl.png">

ver1.0:
<img align="center" src="https://raw.githubusercontent.com/Huyhust13/3D_SLAM_HustAIS/master/figures/workflowTestDepth.png">
--------------
## Triển khai:

1. 

2. 

Update 24/7/2019:
- Chạy dữ liệu CityScapes với PSMNet thiếu dung lượng GPU.
    - Khắc phục: 
        - Giảm size ảnh đầu vào bằng np.resize --> disparity không đúng
    - Giải pháp:
        - Mượn gpu để chạy
        - Tìm cách để chạy được infer với gpu ram thấp
        - Chọn mạng khác

25/7/2019:
- Chạy trên gpu (6Gb) của AIS cũng bị `out of memory`.
    


25/07/2019:
- Chạy mô hình pytorch trên nhiều gpu --> chưa được
- resize skimage --> mạng k nhận do sai đầu vào
- dùng opencv --> khác với skimage
- log --> nếu dùng >> trong bash shell để ghi file thì phải chạy xong vòng for mới hiện ra console (không realtime trong vòng for)
    - Dùng thư viện logging thì tạo ra nhiều loại log k cần thiết

26/7/2019:
- Chốt dùng log yaml config -> vừa ghi log ra console vừa ghi vào file
- Sau khi resize thì ra kq không ổn -> Test lại crop thì vẫn không được --> đã fix được nhưng k biết nguyên nhân :(

- resize Cityscape về 1024x512:
    - kq khá ok (file tmp_1024x512)
    - dùng ram gpu ~ 4.5gb, time ~1.9s/ảnh - aisgpu2

- resize Cityscape về 768x384:
    - kq? tam chap nhan
    - ram gpu ~2.7gb, time ~1.09s/anh - aisgpu2

- Môi trường: 
    - cuda 8.0
    - torch 0.4.0
    - torchvision 0.2.0
    ??? Trên home pc khong chay duoc (ubuntu 18.04 cuda 10 -> kq lỗi, cuda 8 -> k chạy đc (đơ luôn máy))

- Tiếp theo: tính depth, tọa độ.

01/08/2019:
- Kiếm được ảnh KITTY vừa có stereo và vừa có depth
- Chạy PSMNet xong, nhưng chưa tìm được thông số camera để kiểm chứng.

02/08/2019:
- Tạm dùng `For KITTI the baseline is 0.54m and the focal ~721 pixels` làm thông số Stereo KITTY 
- Chạy readDepth: `./readDepth.py --image ../data/KITTY/depth/depth/0000000009.png `
- Chạy depth from disparity: `./depthFromDisparity.py --dataset KITTY --image ../PSMNet/disparity/0000000009.png`
- Chưa tìm được thông số ảnh depth để so sánh.

03/08/2019:

- Found KITTY parrams: 
    - Saved as uint16 PNG images.
    - The depth per pixel can be computed in meter by converting uint16 to float and dividing it by 256.0: 
    `depth(x,y) = float(I(x,y)/256.0)`
    [Link here](http://www.cvlibs.net:3000/ageiger/rob_devkit/src/master/depth)

    - Extrinsic: 
        - baseline = 0.54m.
        - depth from lidar > depth from Stereo: 0.27m
        - [link here](http://www.cvlibs.net/datasets/kitti/setup.php)
- How to compare depth (from dataset and from PSMNet)??? 

Compare:
(_Depth from dataset should be greater than depth from PSMNet 0.27m_)

| Position | Depth From dataset | Depth from PSMNet |
|---|---|---|
| (445:220) | 13.9140625 | 13.6835584843 |
| (455:220) | 14.1796875 | 13.8355136036 |
| (393:166) | 8.8046875 | 8.72088896666 |
| (709:183) | 77.55078125 | 56.7279681275 |
| (857:173) | 30.82421875 | 28.4937221269 |
| (613:163) | 29.94921875 | 27.9347085202 |
|  |  |  |

### 16/8/2019:
- Vấn đề xảy ra sau khi sử dụng bộ cityscapes:
    - Ảnh quá thưa
    - Nếu sử dụng tập videoDemo (ảnh liên tiếp) thì không có thông tin về odom, right image
- Check lại tập Apollo:
    - Bộ "road02_ins" có: 
        - colorImage: chứa stereo camera (camera 5 và 6)
        - Label: Chứa thông tin object detection.
        **Đặc biệt ở file này chứa file json có thông tin bouding box object nhưng không có pole, traffic light... Tuy nhiên xem kĩ lại thì trong ảnh _*bin.png_ có chứa thông tin object detection có cả pole, traff light chứa trong giá trị pixel (81-83). Có thể trích xuất từ đây ra boudingbox**