# 使用 darkflow 訓練自己的 YOLO
[YOLO 官網](https://pjreddie.com/darknet/yolo/)  
[darkflow](https://github.com/thtrieu/darkflow)  
[darknet](https://github.com/pjreddie/darknet)  

## 下載 darkflow
到 darkflow GitHub 下載或 clone

## 準備 datasets
1. 準備圖片(.jpg)及註解(.xml)
2. 在 darkflow-master 內創建 train 目錄
3. 將欲訓練的圖片放置 train/image，註解放置 train/annotation
4. 測試的圖片放至 darkflow-master/sample_img

## 準備 weights 檔及 cfg 檔
1. 到 [YOLO 官網](https://pjreddie.com/darknet/yolo/)下載欲訓練 YOLO 類型的 weights 檔及 cfg 檔，例如我要訓練 YOLO v2 就下載 YOLO v2 的 weights 及 cfg
2. 將 weights 檔放進 darkflow-master/bin，cfg 檔放進 darkflow-master/cfg
3. 複製一份 cfg 檔並重新命名，例如 yolov2_test.cfg
4. 開啟複製的 cfg 檔
5. 將檔最後一層 [region] 的 classes 改為預測種類的數量，例如我只要預測狗這個種類就將 classes 改為 1
6. 修改 anchors，可利用[kmeans](https://github.com/lars76/kmeans-anchor-boxes)計算自己 data 的 anchors
7. 修改最後一層 [convolutional] 的 filters，YOLO v2 的計算公式為 (classes+5)\*5，YOLO v3 的計算公式為 (classes+5)\*3，例如我用 YOLO v2 預測一個種類就改為 (1+5)\*5=30$
<font color="red">
原始的 cfg 檔必須要保留下來，之後用 weights 檔訓練時會比較原始的 cfg 檔及新的 cfg 檔
</font>

## 修改 label
修改 darkflow-master/labels.txt，將內容改為預測種類的 label，一行寫一個類別

## 開始訓練
1. 開啟 cmd 進入 darkflow-master
  ```cd ./darkflow-master```
2. 設定
  ```python setup.py build_ext --inplace```
3. 訓練
  - 使用 weights 檔訓練
    ```python ./flow --load bin/yolov2.weights --model cfg/yolov2_test.cfg --train --annotation train/annotation --dataset train/image```
  - 不使用 weights 檔訓練
    ```python ./flow --model cfg/yolov2_test.cfg --train --annotation train/annotation --dataset train/image```
  :::info
  常用參數
  ```--gpu 0.7```：設定 gpu 使用率為 0.7
  ```--epoch 30```：設定 epoch 為 30
  ```--batch 10```：設定 batch 為 10
  ```--lr 1e-3```：設定 learning rate 為 0.001
  其他參數可參閱 darkflow-master/darkflow/default.py
  :::
  :::success
  若要繼續訓練模型可使用
  ```python ./flow --load -1 --model cfg/yolov2_test.cfg --train --annotation train/annotation --dataset train/image```
  或從任意檢查點開始
  ```python ./flow --load [checkpoint] --model cfg/yolov2_test.cfg --train --annotation train/annotation --dataset train/image```
  :::
## 測試
```python ./flow --load -1 --model cfg/yolov2_test.cfg```  
測試結果會在 darkflow-master/sample_img/out  
若要觀察 bounding box 的位置可輸出 json 檔來查看  
```python ./flow --load -1 --model cfg/yolov2_test.cfg --json```  

## 可能遇到的錯誤
#### ```AssertionError: expect 202314760 bytes, found 203934260```
- 檢查是否有保留原 cfg 檔
#### ```sre_constants.error: bad character range k-a at position 364```
- 可能是 train/image 內又分不同目錄，圖片必須全都放在 train/image 內
#### ```ZeroDivisionError: division by zero```
- annotation 必須為 .xml
#### ```AttributeError: 'NoneType' object has no attribute 'shape'```
- 圖片路徑錯誤
- 檢查註解(xml檔)內的 <filename> 圖片名稱是否有副檔名(.jpg)
#### 測試後沒有輸出(沒有bounding box)
- 可能是訓練次數不夠，多跑幾個 epoch
- 可嘗試一次訓練一小批資料(第一次使用3~5筆)，擬和後加入另一小批資料(數量通常為原資料的一半)再訓練
