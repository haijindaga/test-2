# Phase 0 — GR00T SONIC を Ubuntu で動かす（動作検証）

目的: **NVIDIA 公式の SONIC デモ（MuJoCo + キーボード WASD 操縦）が
この PC（RTX 4060 / Ubuntu 22.04）で動くことを確認する。**
ここが動けば、本プロジェクト（LLM＋安全レイヤー）の土台が確定します。

> ⚠️ このファイルのコマンドは、コピペで `_`（アンダースコア）が消える問題を
> 避けるため、**わざとグロブ（`*`）で書いてあります**。そのまま貼ってください。
> 「正しい名前に直す」必要はありません。

---

## 0. 事前チェック（1分）

```bash
nvidia-smi
df -h ~
docker --version
```

- `nvidia-smi` の右上 **Driver Version をメモ**（Docker ルートは 550 以上が必要）。
- 空き容量は **30GB 以上**を推奨（リポジトリ＋モデルが大きい）。
- `docker` が無くても OK（その場合はネイティブルートで進める）。

## 1. クローン（Git LFS 必須）

```bash
sudo apt update
sudo apt install -y git-lfs
git lfs install
cd ~/projects
git clone https://github.com/NVlabs/GR00T-WholeBodyControl.git
cd GR00T-WholeBodyControl
git lfs pull
```

> LFS を忘れると ONNX やメッシュが「小さなポインタファイル」になり後で謎エラーになります。
> `git lfs pull` は必ず実行。

## 2. MuJoCo シミュレータ環境を作る

```bash
bash install*/*mujoco*sim*.sh
```

（`install_scripts/install_mujoco_sim.sh` に展開されます。`.venv_sim` という
仮想環境が自動で作られます）

```bash
source .venv*/bin/activate
python check*.py
```

`check_environment.py` が「OK」系の出力を出すか確認。

## 3. モデル（チェックポイント）のダウンロード

```bash
source .venv*/bin/activate
python download*hf.py
```

- HuggingFace の `nvidia/GEAR-SONIC` から
  `model_encoder.onnx` / `model_decoder.onnx` / `observation_config.yaml` /
  `planner_sonic.onnx` が落ちてきます。
- **planner は WASD 操縦（Planner Mode）に必要**なので、`--no-planner` は付けない。
- もし認証エラーが出たら: HuggingFace アカウントでモデルページの
  ライセンス同意 → `pip install -U "huggingface_hub[cli]"` → `hf auth login`
  （トークンを貼る）→ 再実行。

## 4. デプロイスタック（ポリシー推論側）のビルド

**ルート A: ネイティブ（Docker 不要・まずこちらを試す）**

```bash
cd gear*deploy
bash scripts/*deps*.sh
source scripts/setup*.sh
just build
```

（`install_deps.sh` が TensorRT 10.13 等の依存を入れます。
`just` が無いと言われたら `sudo apt install -y just` か、
`bash scripts/*deps*.sh` の出力に従ってください）

**ルート B: Docker（ネイティブで TensorRT 関連が失敗した場合）**

```bash
cd gear*deploy
./docker/run-ros2-dev.sh
# コンテナ内で:
source scripts/setup*.sh
just build
```

（要: NVIDIA ドライバ 550+、docker、nvidia-container-toolkit）

## 5. 実行（端末2つ）

**端末1 — MuJoCo シミュレータ:**

```bash
cd ~/projects/GR00T-WholeBodyControl
source .venv*/bin/activate
python gear*/scripts/run*sim*loop.py
```

**端末2 — SONIC ポリシー＋キーボード:**

```bash
cd ~/projects/GR00T-WholeBodyControl
cd gear*deploy
source scripts/setup*.sh
bash deploy.sh --input-type keyboard sim
```

> ⚠️ 実行中は Ollama で大きいモデルを回さない（VRAM の取り合いになるため）。
> いつもの「計画フェーズと実行フェーズの時間分割」と同じ考え方です。

## 6. キー操作（確認すること）

| キー | 動作 |
|---|---|
| `ENTER` | モード切替（通常 ⇄ **Planner Mode**） |
| `W` / `S` | 前進 / 後退（Planner Mode） |
| `A` / `D` | 旋回 |
| `,` / `.` | 横歩き（ストレイフ） |
| `9` / `0` | 速度ダウン / アップ |
| `1`〜`8` | モーションセット内の切替（歩き方いろいろ） |
| `N` / `P` | モーションセットの巡回 |
| `T` | （通常モード）参照モーション再生 |
| `O` | **緊急停止** ← 将来、安全レイヤーがここに介入します |

**チェックリスト:**
- [ ] G1 が MuJoCo ビューアに立って表示される
- [ ] ENTER で Planner Mode に入り、W で前進する
- [ ] A/D で曲がる、`,` `.` で横歩きする
- [ ] 1〜8 / N / P で違うモーションになる
- [ ] O で止まる

## 7. 結果報告（これをコピーして埋めて返信）

```
Driver Version   : 
ルート            : ネイティブ / Docker
手順1 クローン    : OK / エラー(貼る)
手順2 sim環境     : OK / エラー(貼る)
手順3 モデルDL    : OK / エラー(貼る)
手順4 ビルド      : OK / エラー(貼る)
手順5 起動        : OK / エラー(貼る)
WASD歩行         : OK / NG(様子を書く)
体感             : スムーズ / カクつく / その他
```

エラーが出たら**最後の30行くらい**をそのまま貼ってくれれば、次の版で直します。
1〜2往復の修正は想定内です（特に手順4の TensorRT まわり）。
