# sonic-guard — LLM-driven safe humanoid (GR00T SONIC × Cross-Layer supervision)

自然言語の指示で動くヒューマノイド（Unitree G1, MuJoCo, GR00T **SONIC**）に、
LTL→Büchi オートマトンによる **cross-layer 安全監視**を付けるプロジェクト。

- LLM は **1つだけ**（Ollama / gemma3:27b）: タスク計画と（RoboGuard 式の）LTL 生成を時分割で担当
- 安全保証は LLM ではなく **spot（形式手法）** が出す
- モーションは NVlabs/GR00T-WholeBodyControl の SONIC（キーボード WASD 相当の
  速度コマンドをプログラムから注入する）

## 現在地

- **Phase 0（いまここ）**: SONIC 公式デモの動作検証 → [PHASE0_SETUP.md](PHASE0_SETUP.md)
- Phase A: 頭脳側（LLM 計画 + LTL 生成 + Büchi 検証 + plan.json 出力）
- Phase B: SONIC へのコマンド注入アダプタ + 2D ナビゲータ
- Phase C: ランタイム実行監視レイヤー（第3レイヤー・新規性）
- Phase D: 実験ハーネス（safety rate 等の計測）

## 引用義務（論文化するとき）

- Wang et al., *Ensuring Safety in LLM-Driven Robotics: A Cross-Layer Sequence
  Supervision Mechanism*, IROS 2024.
- Ravichandran et al., *RoboGuard: Safety Guardrails for LLM-enabled Robots*,
  arXiv:2503.07885.
- NVIDIA GR00T-WholeBodyControl / GEAR-SONIC（リポジトリの CITATION.cff に従う）
