# O-CNN
## 下載官方O-CNN
(https://github.com/Microsoft/O-CNN)  
官方 caffe 編譯說明: http://caffe.berkeleyvision.org/installation.html#compilation  
官方 O-CNN 編譯說明: https://github.com/microsoft/O-CNN/blob/master/docs/installation.md

## 將.off轉成.points
下載virtual scanner: https://github.com/wang-ps/O-CNN/tree/master/virtual_scanner  
將資料夾內的.off轉成.points  
`./virtualscanner ~/vip3d`  
.off原檔在benchmark中可下載: http://163.22.20.110/benchmark/index.html  

## 將.points轉成.octree
先製作一個.txt內容是.points的絕對路徑  
利用執行檔octree將檔案轉成.octree檔並存在新的資料夾  
`./octree --filenames vip3d.txt --depth 5 --axis z --output_path ~/vip3d_octree/`  

## 建立lmdb
建立一個txt，將octree檔案做分類，依照種類數目從0開始編號  
0-扇角花金龜  
1-灰瘤象  
2-斑蝥  
3-巨扁鍬甲  
4-獨角仙  
將檔案以7:3的比例製作兩個lmdb分別是test與train  
`./convert_octree_data ~/vip3d_octree/ ~/vip3dtrain.txt ~/vip3d_Train`  

## Training
`ocnn_M40_5.caffemodel`以利用ModelNet40訓練過的模型作為pretrain model  
修改solver檔，將學習率改為0.00001  
`base_lr: 0.00001`  
修改prototxt，將train與test的資料來源路徑做改變  
`source: "/home/viplab/vip3d_train/`  
`source: "/home/viplab/vip3d_test/`  
修改prototxt中最後一層的全連接層名稱與output數目  
`name: "ip2"` -> `name: "re-ip2"`  
`num_output: 40` -> `num_output: 5` 
並執行以下指令  
`/home/viplab/ocnn/caffe/build/tools/caffe train -solver solver_M40_5.prototxt --weights="M40_5.caffemodel"`  

## 驗證
將需要驗證的3D模型依上述方法建立lmdb，並將prototxt中test資料集來源改成此lmdb  
`source: "/home/viplab/netv2_test/"`    
將test中的batch_size改為1  
`batch_size: 1`  
利用剛剛訓練好的模型執行以下指令，且由於一個模型總共會生成12個octree檔所以迭代12次  
`/home/viplab//ocnn/caffe/build/tools/caffe test --model=M40_5.prototxt --gpu=0 --weights=/home/viplab/ocnn/caffe/examples/o-cnn/CAFFE/M40_5__iter_160000.caffemodel --iterations=12`  
