# **2023/5/26 大幅更新！記法が変わりました！**
**以下のサンプルコードは新しい記法に修正済みです。お手数ですがコードの修正をお願いします。**

①文字定義の部分<br>
変更前：x, y, z = symbols('x y z')　※sympy.symbolsを使用<br>
変更後：x, y, z = symbols('x y z')　※tytan.symbolsを使用<br>
※これによりimport sympyが不要に<br>
※使い方はsympy.symbolsと同じ、なので引き続きsympy.symbolsを使ってもOK<br>

②コンパイルの部分<br>
変更前：QUBO, offset = qubo.Compile(H).get_qubo()<br>
変更後：**qubo, offset = Compile(H).get_qubo()**<br>
※quboが変数名かクラス名か紛らわしかったためquboクラスをカット<br>
※quboは変数名として使用してください<br>

# TYTAN（タイタン）
大規模QUBOアニーリングのためのSDKです。

**QUBO**を共通の入力形式とし、複数のサンプラーから選んでアニーリングできます。

QUBO化には、数式を記述する方法、QUBO行列を入力する方法、QUBO行列のcsvを読み込んで入力する方法があります。

## 問題サイズ
ローカル：1,000量子ビット程度

API経由：1,000-10万量子ビット程度

## インストール
更新が頻繁なためgithubからのインストールを推奨します。
```
pip install git+https://github.com/tytansdk/tytan
```

pipインストールはこちらです。
```
pip install tytan
```

## サンプルコード
基本的には定式化を行い、ソルバーと呼ばれる問題を解いてくれるプログラムにdict形式で入力をします。下記の例ではsympyを使って数式から入力をしています。数式以外にもcsvやnumpy matrixやpandas dataframeなど様々な入出力に対応しています。

sympy記法
```
#from tytan import symbols, Compile, sampler
from tytan import *

# 変数を定義
x, y, z = symbols('x y z')

#式を記述
expr = 3*x**2 + 2*x*y + 4*y**2 + z**2 + 2*x*z + 2*y*z

# Compileクラスを使用して、QUBOを取得
qubo, offset = Compile(expr).get_qubo()

# サンプラーを選択
solver = sampler.SASampler()

# 計算
result = solver.run(qubo, shots=100)
for r in result:
    print(r)
```

numpy記法
```
#from tytan import Compile, sampler
from tytan import *
import numpy as np

# QUBO行列を記述（上三角行列）
matrix = np.array([[3, 2, 2], [0, 4, 2], [0, 0, 2]])

# Compileクラスを使用して、QUBOを取得。
qubo, offset = Compile(matrix).get_qubo()

# サンプラーを選択
solver = sampler.SASampler()

# 計算
result = solver.run(qubo, shots=100)
for r in result:
    print(r)
```

pandas記法（csv読み込み）
```
#from tytan import Compile, sampler
from tytan import *
import pandas as pd

# QUBO行列を取得（csv, 上三角行列）
# header, index名を設定した場合は変数名が設定した名前になる。
csv_data = pd.read_csv('qubo.csv')

# Compileクラスを使用して、QUBOを取得
qubo, offset = Compile(csv_data).get_qubo()

# サンプラーを選択
solver = sampler.SASampler()

# 計算
result = solver.run(qubo, shots=100)
for r in result:
    print(r)
```

### 出力例
上記の例は入力に変数のラベルごと入力できるので、定式化した変数そのままで数値が戻ってきます。配列となっている答えは、「量子ビットの値」、「エネルギー（コスト）値」、「出現確率」の順で格納されています。

```
[{'z': 0, 'y': 0, 'x': 0}, 0.0, 8]
[{'z': 1, 'y': 0, 'x': 0}, 1.0, 15]
[{'z': 0, 'y': 0, 'x': 1}, 3.0, 12]
[{'z': 0, 'y': 1, 'x': 0}, 4.0, 11]
[{'z': 1, 'y': 0, 'x': 1}, 6.0, 17]
[{'z': 1, 'y': 1, 'x': 0}, 7.0, 12]
[{'z': 0, 'y': 1, 'x': 1}, 9.0, 16]
[{'z': 1, 'y': 1, 'x': 1}, 14.0, 9]
```


### サンプルコード２
3ルーク問題は、3×3マスに3つのルーク（飛車）を互いに利きが及ばないように置く方法を探す問題です。2次元配列的な量子ビットの設定方法と、Auto_arrayクラスによる4種の結果可視化方法をご確認ください。
```
from tytan import symbols, Compile, sampler, Auto_array

#量子ビットを用意（まとめて指定）
q0_a, q0_b, q0_c = symbols('q0_a q0_b q0_c')
q1_a, q1_b, q1_c = symbols('q1_a q1_b q1_c')
q2_a, q2_b, q2_c = symbols('q2_a q2_b q2_c')

#各行に１つだけ１
H = 0
H += (q0_a + q0_b + q0_c - 1)**2
H += (q1_a + q1_b + q1_c - 1)**2
H += (q2_a + q2_b + q2_c - 1)**2

#各列に１つだけ１
H += (q0_a + q1_a + q2_a - 1)**2
H += (q0_b + q1_b + q2_b - 1)**2
H += (q0_c + q1_c + q2_c - 1)**2

#コンパイル
qubo, offset = Compile(H).get_qubo()

#サンプラー選択（乱数シード固定）
solver = sampler.SASampler(seed=0)

#サンプリング（100回）
result = solver.run(qubo, shots=100)

#すべての結果を確認
print('result')
for r in result:
    print(r)

#１つ目の結果を自動配列で確認（ndarray形式）
arr, subs = Auto_array(result[0]).get_ndarray('q{}_{}')
print('get_ndarray')
print(arr)
print(subs)

#１つ目の結果を自動配列で確認（DataFrame形式）
df, subs = Auto_array(result[0]).get_dframe('q{}_{}')
print('get_dframe')
print(df)

#１つ目の結果を自動配列で確認（image形式）
import matplotlib.pyplot as plt
img, subs = Auto_array(result[0]).get_image('q{}_{}')
print('get_image')
plt.imshow(img)
plt.yticks(range(len(subs[0])), subs[0])
plt.xticks(range(len(subs[1])), subs[1])
plt.show()
```

```
result
[{'q0_a': 0.0, 'q0_b': 0.0, 'q0_c': 1.0, 'q1_a': 0.0, 'q1_b': 1.0, 'q1_c': 0.0, 'q2_a': 1.0, 'q2_b': 0.0, 'q2_c': 0.0}, -6.0, 19]
[{'q0_a': 0.0, 'q0_b': 0.0, 'q0_c': 1.0, 'q1_a': 1.0, 'q1_b': 0.0, 'q1_c': 0.0, 'q2_a': 0.0, 'q2_b': 1.0, 'q2_c': 0.0}, -6.0, 13]
[{'q0_a': 0.0, 'q0_b': 1.0, 'q0_c': 0.0, 'q1_a': 0.0, 'q1_b': 0.0, 'q1_c': 1.0, 'q2_a': 1.0, 'q2_b': 0.0, 'q2_c': 0.0}, -6.0, 21]
[{'q0_a': 0.0, 'q0_b': 1.0, 'q0_c': 0.0, 'q1_a': 1.0, 'q1_b': 0.0, 'q1_c': 0.0, 'q2_a': 0.0, 'q2_b': 0.0, 'q2_c': 1.0}, -6.0, 19]
[{'q0_a': 1.0, 'q0_b': 0.0, 'q0_c': 0.0, 'q1_a': 0.0, 'q1_b': 0.0, 'q1_c': 1.0, 'q2_a': 0.0, 'q2_b': 1.0, 'q2_c': 0.0}, -6.0, 11]
[{'q0_a': 1.0, 'q0_b': 0.0, 'q0_c': 0.0, 'q1_a': 0.0, 'q1_b': 1.0, 'q1_c': 0.0, 'q2_a': 0.0, 'q2_b': 0.0, 'q2_c': 1.0}, -6.0, 17]
get_ndarray
[[0 0 1]
 [0 1 0]
 [1 0 0]]
[['0', '1', '2'], ['a', 'b', 'c']]
get_dframe
   a  b  c
0  0  0  1
1  0  1  0
2  1  0  0
get_image
```
<img src="https://github.com/tytansdk/tytan/blob/main/img/img-01.png" width="15%">



# サンプラー
サンプラーと呼ばれる計算をしてくれるプログラムが搭載されています。TYTANでは基本的には簡単なソルバーを用意はしていますが、ユーザーごとにソルバーを用意して簡単に繋げられるようにしています。ローカルのサンプラーのように手元のマシンにプログラムとして搭載する以外にも、APIのサンプラーでサーバーに接続する専用サンプラーなど様々な形態に対応できます。

ローカルサンプラー：
```
SASampler
GASampler
```

商用クラウドサンプラー：
```
ZekeSampler
NQSSampler
```

# 商用利用
TYTANは商用利用前提ですので、個人での利用はもちろん企業での活用を許可し、促進しています。

# コントリビューター
https://github.com/tytansdk/tytan/graphs/contributors
