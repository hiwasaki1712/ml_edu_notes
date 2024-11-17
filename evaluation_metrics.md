# 分類タスクでの評価指標
分類タスクの評価指標は正解率以外にもあり、見比べることで分類モデルの性能の特徴がわかる。

## 混同行列 Confusion Matrix

データの正解ラベルと予測ラベルとを対応させた表。  
どのクラス間で正解・不正解があるのか、モデル性能を定量的に可視化できる。

2値分類の場合は以下。
| 　 | 予測 Positive | 予測 Negative |
| :--- | :---: | :---: |
| 正例 True | TP | FN |
| 負例 False | FP | TN |

多クラス分類の場合は、Nクラスとして「正解クラス」対「予測クラス」の $N \times N$ 表になる。  
下図は、3クラス分類のときの例
![confusion_matrix_penguins](https://github.com/user-attachments/assets/68cddaec-a17f-4082-9d69-5cb743363146)


## 2値分類での指標の定義
* 正解率: ${\rm Accuracy} = \frac{TP+TN}{TP+FN+FP+TN}$  
  * 全サンプルの中で、正しく予測できた割合
* ${\rm Precision} = \frac{TP}{TP+FP}$  
  * 「正」と予測した中で、本当に正例だった割合  
  * この値が高いほど、負例が紛れ込むのを抑えられる。  
* ${\rm Recall} = \frac{TP}{TP+FN}$
  * 正例ラベルの中で、正しく「正」と予測できた割合。真陽性率(TPR)  
  * この値が高いほど、正例の取りこぼしを抑えられる。  
* ${\rm F1\ score} = \frac{2}{Precision^{-1}+Recall^{-1}}$  
  * precision と recall の調和平均  
  * 下の表のクラスごとの値は、2値分類としてみた場合に相当。
* 真陽性率 (TPR; true positive rate): ${\rm TPR} = {\rm Recall}$
  * 正例ラベルの中で、正しく「正」と予測できた割合  
* 儀陽性率 (FPR; false positive rate): ${\rm FPR} = \frac{FP}{FP+TN}$
  * 負例ラベルの中で、間違って「正」と予測された割合  


## 多クラス分類の評価
[各クラス平均の取り方](https://scikit-learn.org/1.5/modules/model_evaluation.html#from-binary-to-multiclass-and-multilabel)が複数ある。
クラス間にサンプル数の偏りがあると、平均の取り方でスコアが変わる。顕著な不均衡データでは特に注意が必要。  
不均衡データで正解率だけ見るのはダメ！  
各クラスのサンプル数を揃えていれば値が同じになるので気にしなくてよい。

* Micro 平均:
  * クラスを分けず、全サンプルを対等にして計算  
  * サンプル数の少ないクラスが反映されにくいので、不均衡データでは使えない  
  * micro precision, micro recall, micro F1 は micro accuracy と等しい
* Macro 平均:
  * 各クラスごとにスコアを計算し、その平均をとる  
  * 全クラスを対等に扱い、クラス間の不均衡の影響を受けない
  * macro recall は macro accuracy に等しい
* Weighted 平均:
  * 各クラスごとにスコアを計算し、そのクラスのサンプル数で重みづけして平均する
  * Macro 平均と似ているが、クラス間に重みをつける点が違う  
  * weighted recall は micro recall = accuracy と等しい

Scikit-learn での定義と解説は[こちら](https://scikit-learn.org/1.5/modules/model_evaluation.html#multiclass-and-multilabel-classification)

## [ROC (reciever operating characteristic) 曲線](https://scikit-learn.org/1.5/modules/model_evaluation.html#roc-metrics)
分類モデルが予測した確率をラベルに変換するときの閾値を変化させたときの真陽性率 (TPR) と儀陽性率 (FPR) の関係を図示したもの。
縦軸に TPR 、横軸に FPR をとる。  
![roc_penguins](https://github.com/user-attachments/assets/299d63be-2732-4f9a-afb0-8a8c4dc08b61)

### ROC曲線の読み方
* 左上の点が理想的で、TPR=1、FPR=0の全て正しく答えられた状態  
* 右下の点が最悪で、TPR=0、FPR=1で全て間違える状態
* 左下の点は、全て「負」と答える状態
* 右上の点は、全て「正」と答える状態
* 左下から右上をつなぐ直線が chance level で、ランダムに答えるモデルだとこの線上にのる。
このとき AUC=0.5 となる
* 曲線の下側の面積 (AUC; area under the curve) が1に近いほど理想に近づく

### 多クラスの場合
TPRとFPRを定義する方法が以下の2通りある。多クラス分類のROC曲線を描くときは、どちらか選択する。
* One-vs-One
  * クラスの組み合わせペアを全通り計算
  * Macro 平均を使えば、クラスの不均衡の影響を受けない
* One-vs-the-Rest
  * ある1クラスと、それ以外の全クラスを1ペアとして計算
  * クラスが不均衡だと、「残り (rest)」グループの構成に反映されるため、macro 平均を使っても影響を受ける
