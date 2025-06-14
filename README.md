# BTC価格の極小・極大予測 Transformerモデル

## 概要 

このプロジェクトは、Transformerモデルとソフトラベリング技術を用いて、ビットコイン（BTC）の5分足価格データから、価格の**極小点**（買いシグナル候補）と**極大点**（売りシグナル候補）を予測するためのものです。

モデルは、**分類タスク**（極小/極大/その他）と**回帰タスク**（将来価格の予測）を同時に学習するマルチタスク学習フレームワークを採用しています。

## 主な特徴 

* **Transformerベースのモデル**: 時系列データ内の長期的な依存関係やパターンを捉えるために、Transformer Encoderアーキテクチャを利用します。
* **スコアベースのソフトラベリング**: 単純な「極小=1, 極大=2」といったハードラベルではなく、価格が周囲の価格と比べてどれだけ突出しているかの「度合い」をスコア化し、それを確率分布（ソフトラベル）としてモデルの学習ターゲットとすることで、より曖昧でノイズの多い金融データに対して柔軟な学習を目指します。
* **マルチタスク学習**: 分類損失（KLダイバージェンス）と回帰損失（MSE）を組み合わせることで、モデルがより豊かな特徴表現を学習することを促します。
* **詳細な評価とシミュレーション**: 標準的な分類メトリクスに加え、予測シグナルに基づいた損益（P&L）シミュレーションや、シグナル発生後の価格変動統計など、多角的な評価機能を提供します。


## セットアップと実行方法 🔧

### 1. 前提条件

* Python 3.8+
* PyTorch
* pandas
* NumPy
* scikit-learn
* Matplotlib
* tqdm

必要なライブラリは、以下のコマンドでインストールできます。


```bash
pip install torch pandas numpy scikit-learn matplotlib tqdm
```

## ディレクトリ構成
```
.
└── transformer
    ├── evaluation_results
    │   ├── best_transformer_model.pth
    │   ├── test_independent_eval_future_returns_epbest_model_file.csv (テスト期間のシグナル後指標)
    │   ├── test_independent_eval_future_returns_epbest_model_file.txt (テスト期間のシグナル後指標)
    │   └── test_independent_eval_metrics_epbest_model_file.txt (ハードラベリングに対する分類テスト結果)
    └── ev_transformer.ipynb（メインの実行ファイル）
```

## モデル構造 
モデルはPyTorchで実装されており、主に2つのクラスから構成されます。

- PositionalEncoding: Transformerが入力の順序情報を理解できるように、各時点のデータに位置情報を付加します。
- TransformerClassifierSoftLabel: メインのモデル。
  - 入力: 4つの特徴量（正規化されたclose, high, low, volume）を持つ時系列シーケンス。
  - 構造:
    1. 入力層: 4次元の特徴量をモデルの内部次元数（d_model）に変換します。
    2. Positional Encoding: 位置情報を付加します。
    3. Transformer Encoder: Multi-Head Self-Attention機構を用いて、シーケンス内の時間的パターンを学習します。
    4. 出力ヘッド: Encoderの最終出力（シーケンスの最後の時点のベクトル）を、2つの異なるヘッドに入力します。
        - 分類ヘッド: 3クラス（その他, 極小, 極大）の確率を予測します。
        - 回帰ヘッド: 1つの連続値を予測します（例: 将来の価格変動）。
    5. 損失関数: 分類タスクの KLダイバージェンス損失（ソフトラベル用）と、回帰タスクの 平均二乗誤差（MSE）損失を、重み付けして合計したものを最終的な損失として使用します。

## 予測結果の例
以下は、価格チャートと予測シグナルのプロット例です。緑の上向き三角が「極小（買い）」シグナル、赤の下向き三角が「極大（売り）」シグナルを示しています。

![極小・極大シグナルのプロット例](https://github.com/user-attachments/assets/cee5e84a-7ad6-4367-ad26-c3351462746f)
