# GPUインフラ / Kubernetes GPU 学習リンク集

AIインフラエンジニア向けの学習で紹介されたリンクを、後から見返してそのまま学習に使える形に整理した資料集です。
単なるリンク集ではなく、各リンクに「学べること」「読む価値・目的」「優先度」を付けています。

- 収録数: 17件（2026-11-25に11件、2026-11-26に6件を追加。ほかに既存項目へ統合したURLが1件）
- URL確認日: 2026-11-25（11件）/ 2026-11-26（追加7件）— いずれも実際にアクセスしてタイトルと内容を確認済み
- 出典の会話: サイドチャット「AIインフラエンジニア向け GPUサーバー 学習計画」（2026-11-20 / 2026-11-22紹介分）+ 2026-11-26にユーザーが直接指定したリンク7件
- 重複チェック: 初回追加時点でこのリポジトリには README.md のみで重複なし。2026-11-26追加分では、vLLM Quickstart の stable版（Online Serving節）が既存項目と同ページだったため、新規項目にせず既存項目へURLを統合した

## 優先度の凡例

- **高**: 学習順序の本線。先に読む／ハンズオンで参照する
- **中**: 本線の理解が進んだ段階で読む、または特定テーマのときに参照する
- **低**: 参考資料（今回は該当なし）

## おすすめの読む順番

1. GPUs on Kubernetes Explained (2026) で全体像を通す
2. Kubernetes公式 Schedule GPUs で答え合わせをする
3. AI Factory Operations Lab で GPUなしハンズオン（Build / Break / Diagnose）を回す
4. NVIDIA GPU Operator 公式手順と The New Stack の比較で、導入方式（Device Plugin単体か Operatorか）を判断できるようにする
5. vLLM Quickstart で推論サービングを動かし、GPU Monitoring で監視の型を身につける
6. RunPod の2件は、単体GPUサーバー運用に戻るときの参照用

---

## 1. Kubernetes × GPU：基礎とアーキテクチャ

### GPUs on Kubernetes Explained (2026): Device Plugin, MIG, Time-Slicing, DRA

- URL: https://www.ajayk.sh/blog/gpus-on-kubernetes-explained-2026/
- 種別: 解説ブログ（英語）
- 学べること: PodがGPUを掴むまでの流れ（Device Plugin が `nvidia.com/gpu` を広告し、Schedulerが整数として数える仕組み）、GPU Operatorが入れる部品一式、MIG / Time-Slicing / MPS の共有方式の違い、DRA（Dynamic Resource Allocation）、Kueueによる公平なキューイング
- 読む価値・目的: Kubernetes GPUの総合解説として会話で「まずこれ」とされた1本。2026年時点の内容まで追随しており、アーキテクチャの全体像を1本で通せる
- 優先度: **高**

### Schedule GPUs | Kubernetes（公式）

- URL: https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/
- 種別: 公式ドキュメント（英語）
- 学べること: Device Pluginの公式な仕組み、GPUは `limits` にだけ書くルール（requestsとlimitsは同値、requests単独は不可）、GPU種別が違うノードのラベル付けとスケジューリング、Node Feature Discoveryによる自動ラベル付け
- 読む価値・目的: 民間解説の「答え合わせ」用。試験や実務で迷ったら最終的にここに戻る
- 優先度: **高**

### Multi-Tenancy, Resource Quotas, and Cost Allocation（Kinda Technical）

- URL: https://kindatechnical.com/mlops-guide/multi-tenancy-resource-quotas-and-cost-allocation.html
- 種別: 実務ガイド（英語）
- 学べること: 複数MLチームでGPUクラスタを共有するときの Namespace 分離、ResourceQuota / LimitRange、PriorityClassによる優先度付きGPUスケジューリング、チーム別コスト配分（チャージバック）
- 読む価値・目的: 単一チームの構築ができるようになった次の段階「共有クラスタの運用設計」の資料
- 優先度: **中**
- 注意: 会話では「Kubernetes for ML: GPU Scheduling, Operators, and Autoscaling」というタイトルで紹介されましたが、2026-11-25に実際に開くと、このURLの実ページは上記のマルチテナンシー／クォータ／コスト配分の記事でした。内容はGPU共有運用に有用なため、実タイトルで収録しています

---

## 2. GPU Operator / Device Plugin：導入と設計判断

### Installing the NVIDIA GPU Operator（NVIDIA公式）

- URL: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html
- 種別: 公式ドキュメント（英語）
- 学べること: GPU Operatorの前提条件、Helmでの導入手順、導入後の確認（Pod状態、ClusterPolicy の ready 確認、`nvidia.com/gpu` の Allocatable 確認）、CUDAサンプルとJupyterでの動作確認、主要なカスタマイズオプション（CDI有効化など）
- 読む価値・目的: ハンズオンで実際に手を動かすときの手順書そのもの。方式B（ドライバ／Toolkitをホスト先行導入し、Operator側で無効化する構成）を組むときも、オプションの意味はここで確認する
- 優先度: **高**

### GPU Orchestration in Kubernetes: Device Plugin or GPU Operator?（The New Stack）

- URL: https://thenewstack.io/gpu-orchestration-in-kubernetes-device-plugin-or-gpu-operator/
- 種別: 解説記事（英語）
- 学べること: Device Plugin単体とGPU Operatorの設計思想の違い、それぞれが向く場面（小規模・管理済みノードイメージならPlugin単体、大規模・本番・異種混在ならOperator）、機能比較表
- 読む価値・目的: 「Plugin単体でいいのか、Operatorが要るのか」という設計判断の根拠を持つための資料。k3s + GPU Operatorで詰まった経験（containerd設定の所有権争い）を一般論として理解し直すのにも役立つ
- 優先度: **高**

### NVIDIA GPU Operator（Leet's CUDA）

- URL: https://www.leetscuda.com/learn/gpu-operator
- 種別: 解説ページ（英語）
- 学べること: GPU Operatorが各GPUノードへ展開するDaemonSet群の図（driver / container-toolkit / device-plugin / gpu-feature-discovery / dcgm-exporter）、Operatorパターンの考え方、GPU Feature Discoveryが付けるノードラベル、MIGプロファイルの例、DCGMメトリクスの基本
- 読む価値・目的: Operatorの「中身の図」が一番分かりやすい入門解説。公式ドキュメントを読む前の下地作りに向く
- 優先度: **中**

### Using NVIDIA vGPU — NVIDIA GPU Operator（公式、2026-11-26追加）

- URL: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/install-gpu-operator-vgpu.html
- 種別: 公式ドキュメント（英語）
- 学べること: Kubernetes上で NVIDIA vGPU を使うためのGPU Operator導入手順。vGPUホストドライバやライセンス（CLS / DLS、クライアント設定トークン）などの前提条件、ゲストドライバのコンテナイメージのビルドとプライベートレジストリへのpush、ライセンス情報のSecret化、HelmでのOperator導入まで
- 読む価値・目的: GPU共有の3方式（MIG＝物理で切る、vGPU＝仮想で作る、Time-Slicing＝時間で貸す）のうち、vGPUを実際にKubernetesで構成する段になったら読む手順書。ライセンスとドライバイメージの自前ビルドが前提になる点が、通常のOperator導入との大きな違い
- 優先度: **中**

---

## 3. GPUインフラ運用：監視

### GPU Monitoring for ML: nvidia-smi, DCGM, and Production Observability Guide（Spheron）

- URL: https://www.spheron.network/blog/gpu-monitoring-for-ml/
- 種別: 実務ガイド（英語）
- 学べること: `nvidia-smi` の実用コマンド（`dmon` / `pmon` / `--query-gpu` のCSV出力）、PyTorch側のメモリプロファイリング、DCGM Exporter → Prometheus → Grafana の本番監視構成、主要DCGMメトリクスとアラート閾値（VRAM 90%超、温度83℃超、XIDエラーなど）
- 読む価値・目的: 「nvidia-smiは入口、本番はDCGM」という運用フェーズ全体像がコマンド付きでまとまっている。NCA-AIIOの運用分野（DCGM、XID、ECC）の実務側の裏付けにもなる
- 優先度: **高**

---

## 4. GPUサーバー単体運用：RunPod

### Storage options - RunPod Documentation（公式）

- URL: https://docs.runpod.io/pods/storage/types
- 種別: 公式ドキュメント（英語）
- 学べること: RunPodの4種ストレージ（Container disk / Volume disk / Network volume / Global volume）の永続性・マウント先・料金・向き不向きの比較。Stopで消えるもの、Terminateで消えるもの、Podと独立して残るものの違い
- 読む価値・目的: 単体GPUサーバー運用の最初に押さえる「データが消える／課金が続く」事故防止の基本。Kubernetesの PV / PVC の考え方との対比でも理解が深まる
- 優先度: **中**

### Network volumes - RunPod Documentation（公式）

- URL: https://docs.runpod.io/storage/network-volumes
- 種別: 公式ドキュメント（英語）
- 学べること: Network Volumeの作り方とPodへの付け方（デプロイ時にしか付けられない制約）、Serverless / Instant Clustersでの使い方、S3互換API、ボリューム間の移行方法と料金
- 読む価値・目的: モデルやデータセットをPodを作り直しても残すための実務手順。RunPodで学ぶときの永続化の本線
- 優先度: **中**

---

## 5. vLLM / LLM推論サービング

### Quickstart - vLLM（公式）

- URL: https://docs.vllm.ai/en/v0.24.0/getting_started/quickstart/
- URL（stable版、2026-11-26追加）: https://docs.vllm.ai/en/stable/getting_started/quickstart/#online-serving
- 種別: 公式ドキュメント（英語）
- 学べること: vLLMのインストール、オフラインのバッチ推論（`LLM` / `SamplingParams`）、OpenAI互換APIサーバーの起動（`vllm serve`）、Completions / Chat Completionsの叩き方、Attentionバックエンドの選び方。stable版の `#online-serving` は Online Serving 節への直接リンク
- 読む価値・目的: 推論サービングのハンズオン入口。RunPodやKubernetes上でvLLMを動かすときの原点になる手順
- 優先度: **高**
- 注意: 最初のURLは v0.24.0 に固定された版、2本目は常に最新を指す stable版です。同じページのため、2026-11-26登録分では新規項目にせずここへ統合しました。手順の再現には版固定、最新仕様の確認には stable を使うこと

### vLLM Production Stack（vllm-project、2026-11-26追加）

- URL: https://github.com/vllm-project/production-stack#step-by-step-tutorials
- 種別: GitHubリポジトリ / チュートリアル集（英語）
- 学べること: vLLMをKubernetes上で本番運用するためのリファレンス構成。Helmでのデプロイ、Step-by-Stepチュートリアル（主要クラウドへの展開、最小構成、カスタマイズ、モデルの読み込み、KVキャッシュオフロード＝LMCache）、リクエストルーター（KVキャッシュ再利用を狙う振り分け）、Prometheus + Grafana のダッシュボード（TTFT、実行中／待ちリクエスト数、GPU KV使用率など）
- 読む価値・目的: 単体の `vllm serve` から一歩進み、「Kubernetes上で複数台のvLLMを本番サービングする」段階の標準形を学ぶ資料。今のハンズオン（k3s + vLLM単体）の次の行き先に当たる
- 優先度: **高**

### Benchmark CLI - vLLM（公式、2026-11-26追加）

- URL: https://docs.vllm.ai/en/stable/benchmarking/cli/#model-forward-benchmark
- 種別: 公式ドキュメント（英語）
- 学べること: `vllm bench` による性能計測。サービングベンチマーク（`vllm bench serve`）での TTFT / TPOT / ITL / スループットの測り方と指標の定義、結果の保存と可視化、カスタムデータセットでの計測、モデル単体の forward 計測（`vllm bench forward`、アンカー指定先）
- 読む価値・目的: 「動いた」の次に必要な「測れるようになる」ための公式手順。AI Factory Operations Lab の負荷計測テーマや、GCEハンズオンの仕上げ（vLLMの応答と性能の確認）と直結する
- 優先度: **高**

### Nano-vLLM（GeeeekExplorer、2026-11-26追加）

- URL: https://github.com/GeeeekExplorer/nano-vllm
- 種別: GitHubリポジトリ（英語、約1,200行のPython実装）
- 学べること: vLLMをゼロから軽量に再実装したコードを読むことで、推論エンジンの中身（Prefix caching、Tensor Parallelism、Torch compilation、CUDA graph など）が最小構成でどう実現されているか
- 読む価値・目的: ブラックボックスとして使う段階から、エンジン内部の仕組みをコードで追う段階へ進むための教材。LLM内部の学習（KVキャッシュ、PagedAttention）と実装をつなぐ位置づけ
- 優先度: **中**

---

## 6. AIインフラ総合ハンズオン（2026-11-25 追加）

### AI Factory Operations Lab

- URL: https://ld-singh.github.io/ai-factory-ops-lab/
- 種別: ハンズオン教材サイト（英語、GitHub: ld-singh/ai-factory-ops-lab）
- 学べること: GPUを持たずに学ぶGPUインフラ運用。kind + KWOK の疑似GPUフリートで「GPU PodがPendingになる原因診断」、KAI Scheduler / Volcano のキューとgang scheduling、HAMiによるGPU共有、Slurm（疑似GRES）、Prometheus / Grafana + 疑似DCGMでの監視とアラート、vLLM指標（TTFT / goodput / KVキャッシュ）でのSLO設計、ノードのプロビジョニングから廃止までのライフサイクル。最終章のみ実GPU（1台レンタル、$5-10程度）で実機検証する構成
- 読む価値・目的: 「Build（構築）→ Break（わざと壊す）→ Diagnose（診断）→ Prove（証拠を残す）」という運用者の型で学べるのが特徴。GCEハンズオンで実際に詰まったPending診断や監視のテーマが、課金なしのシミュレーションで先に練習できる。NCA-AIIOの弱点（Slurm、BCM式の運用ライフサイクル）の補強にも直結する
- 優先度: **高**
- 備考: 共有時のURLには `?utm_source=chatgpt.com` が付いていましたが、資料集にはパラメータなしの正規URLで収録しています（2026-11-25に両方とも同じページが開くことを確認済み）

---

## 7. CUDA / GPUプログラミング基礎（2026-11-26追加）

### CUDA Programming Guide（NVIDIA公式）

- URL: https://docs.nvidia.com/cuda/cuda-programming-guide/index.html
- 種別: 公式ドキュメント（英語）
- 学べること: CUDAのプログラミングモデル全体像。Part 1は言語に依存しないモデル解説（開発者でなくてもGPUの実行の仕組みが分かる）、Part 2でC++ / PythonでのGPUプログラミング基礎、Part 3以降でマルチGPU、CUDA Graphs、Unified Memoryなどの高度な機能とリファレンス
- 読む価値・目的: インフラ運用が主軸でも、「コンテナの中で何が実行されているか」の土台になる公式の原典。全部を読む資料ではなく、CUDAカーネルやメモリの話で疑問が出たときに引く辞書として置く
- 優先度: **中**

---

## 8. 実事例：大規模LLM推論サービング（2026-11-26追加）

### Launching a trillion-parameter model at Sakana AI（Modal）

- URL: https://modal.com/blog/sakana-case-study
- 種別: 事例ブログ（英語、2026-10-23公開）
- 学べること: 約1兆パラメータのMoEモデル（Sakana Namazu）を本番APIとしてサービングした最適化の実例。NVFP4量子化（日本語性能を保つための選択的量子化とキャリブレーション）、投機的デコーディング用のドラフトモデルの学習（生トークンIDで学習させる工夫）、CPU-GPU同期ストールの除去などのカーネルレベルの性能改善、KV-awareルーターによるTTFT改善、オンデマンドGPUでの容量計画
- 読む価値・目的: 量子化、投機的デコーディング、KVキャッシュ、テンソル並列といった学習済みの概念が、本番の推論基盤でどう効くかを具体例で結びつける読み物。vLLM運用の先にある「大規模サービングの最適化」の姿が見える
- 優先度: **中**

---

## メンテナンスメモ

- リンク切れを見つけたら、このファイルの該当項目に「確認日と状態」を追記してから差し替える
- 新しいリンクを追加するときは、カテゴリ・学べること・読む価値／目的・優先度・URL確認日をセットで書く
