# AdrIndex

This directory contains Architecture Decision Records (ADRs) for the SmartIME project.

| ID | Title / Status | Summary (JP) |
|----|-------|--------|
| [Adr0001](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0001CoreLanguageChoice.md) | **Core Language Choice**<br>`Accepted` | コアにRustを採用。Win(C++)、Android(Kotlin/JNI)、ブラウザ(WASM)で共通ロジックを共有し、メモリ安全かつ高性能な基盤を実現する。 |
| [Adr0002](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0002PostDetectionInput.md) | **Post-Detection Input Model**<br>`Accepted` | 「打つ前にモードを決める」常識を捨て、全入力をローマ字で受理。確定後にAIが意図（CLI/日本語等）を判定し最適に変換する事後判定モデル。 |
| [Adr0003](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0003OfflineConstraint.md) | **Offline Constraint**<br>`Accepted` | プライバシー、速度、可用性の観点から完全オフラインを厳守。モデルサイズを30-50MBに抑え、10ms以内の低レイテンシ推論をCPUのみで達成。 |
| [Adr0004](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0004MorphologicalEngineSelection.md) | **Morphological Engine Selection**<br>`Proposed` | 日本語構造解析にMeCabを採用。単なる変換機ではなく、助詞や文末の出現確率を数値化し、日本語らしさを評価する「確率評価器」として活用。 |
| [Adr0005](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0005AssistanceModes.md) | **Assistance Modes**<br>`Proposed` | ユーザーに主導権を渡す3モード（Manual, Assisted, Auto）を実装。過剰な自動確定を防ぎ、システムが思考の邪魔をしない段階的支援を行う。 |
| [Adr0006](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0006TypoTolerance.md) | **Typo Tolerance**<br>`Proposed` | 編集距離とキー配列に基づき1-2文字の誤字を許容。母音脱落や子音重複を「間違い」ではなく「意図」として汲み取り、思考を止めない入力を実現。 |
| [Adr0007](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0007CommandDiscovery.md) | **Command Discovery**<br>`Proposed` | CLI特有の構造（ハイフン、パス、予約語等）をパターン認識で特定。自然文混在時でもコマンド部分を誤って日本語変換しないための判別ロジック。 |
| [Adr0008](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0008LocalLearning.md) | **Local Learning**<br>`Proposed` | ユーザー個別の修正履歴や語彙を「Personal Delta」としてローカル学習。使い込むほどに個人の癖に適応し、自分専用の効率的な入力器へと深化。 |
| [Adr0009](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0009PlatformAdapters.md) | **Platform Adapters**<br>`Accepted` | OS固有制約に対応するためWin(TSF)、Android(IME Service)、Browser(Ext/WASM)の各アダプターを構築。共通Rustコアとブリッジ統合。 |
| [Adr0010](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0010LightweightInferenceStrategy.md) | **Lightweight Inference Strategy**<br>`Proposed` | 超低レイテンシ推論のため量子化された統計モデルを採用。30-50MBの軽量バイナリで、ハイスペックなハードなしに高精度な意図判定をCPUで実行。 |
| [Adr0011](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0011DataStructuresForOfflineSearch.md) | **Data Structures for Offline Search**<br>`Proposed` | TrieによるO(k)高速検索とBloom Filterによる存在判定を導入。辞書の肥大化に依存しない反応速度と、無駄な解析の早期遮断により打鍵に追従。 |
| [Adr0012](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0012PhasedImplementationStrategy.md) | **Phased Implementation Strategy**<br>`Accepted` | 実装を3段階（判別変換、文脈補正、意味補完学習）に分離。開発リスクを抑えつつ、まずは「事後判定」によるモードレスな快適さを早期実現。 |
| [Adr0013](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0013LocalSlmAsIntentClassifier.md) | **Local SLM as Intent Classifier**<br>`Accepted` | 局所言語モデルを「意図ルーター」として採用。入力の「日本語らしさ」をリアルタイムで確率評価し、CLIとの混在をシームレスに自動処理。 |
| [Adr0014](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0014DiscriminatorProcessorSeparation.md) | **Discriminator-Processor Separation**<br>`Accepted` | 高速な「判別機（Disc）」と専門的な「処理機（Proc）」に分離。判別機でブロックを切り分け、各専門処理機が変換・補正・補完などを効率的に分担。 |
| [Adr0015](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0015HybridDictionaryAndModelStrategy.md) | **Hybrid Dictionary and Model Strategy**<br>`Proposed` | 静的辞書(Trie)と動的推論(SLM)のハイブリッド。既知の語彙は高速検索し、未知語や誤字はモデルで補うことで、絶対的速度と柔軟な解釈を両立。 |
| [Adr0016](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0016LayeredDictionaryPartitioning.md) | **Layered Dictionary Partitioning**<br>`Accepted` | 辞書をSystem(静的)、Domain(専門)、Personal Delta(個人)の3層に分割。更新性とプライバシーを両立し、個人データを安全な秘密の庭として隔離。 |
| [Adr0017](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0017MemoryMappedZeroCopyStorage.md) | **Memory-Mapped Zero-Copy Storage**<br>`Accepted` | mmapによるゼロコピー・アクセスを導入。起動時のデシリアライズを排除し、必要なページだけを動的にロード。極限のメモリ節約と高速起動を実現。 |
| [Adr0018](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0018CorpusMetabolismMechanism.md) | **Corpus Metabolism Mechanism**<br>`Proposed` | 語彙の「新陳代謝」サイクルを実装。短期記憶から長期記憶への定着、および低頻度語のパージを自律的に行い、常に最適化された辞書状態を維持。 |
| [Adr0019](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0019HighPerformanceAndPersonalizationOptimization.md) | **High Performance and Personalization Optimization**<br>`Proposed` | Personal Deltaの動的同期、Rustによる全プラットフォーム抽象レイヤーの統一、SIMD/mmapによる極限の高速化を導入し、遅延ゼロのパーソナライズ体験を実現。 |
| [Adr0020](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/Adr0020ClipboardOneClickPaste.md) | **Clipboard One-Click Paste**<br>`Proposed` | クリップボード内の画像を1ボタン（またはCtrl+V）で即座にNDLOCR-Lite Webへ貼り付け、OCR処理を開始する「摩擦ゼロ」の入力体験を定義。 |

| [Adr0021](Adr0021_CSharpWpfUiForConfiguration.md) | **C# UI for Configuration**<br>`Proposed` | Windows向け設定UIにC# (WPF)を採用。RustコアとはFFI (P/Invoke)で連携し、UI開発の生産性とパフォーマンスを両立する。 |
| [Adr0022](Adr0022_DatabaseSelectionForPersonalDelta.md) | **Database for Personal Delta**<br>`Proposed` | 個人学習データ(Personal Delta)の永続化にSQLite、高速キャッシュに純Rust製KVストア(redb)を併用するハイブリッド戦略を採用。 |
| [Adr0023](Adr0023_MlStrategyForIntentClassification.md) | **ML Strategy for Classification**<br>`Proposed` | 意図分類のML戦略として、形態素解析に純Rust製・高速な「Vibrato」、推論エンジンに純Rust製ONNXランタイム「Tract」を採用する。 |
| [Adr0024](Adr0024_WordPrediction.md) | **Word and Phrase Prediction**<br>`Proposed` | Personal DeltaのN-gramとTrieを活用したリアルタイム単語・フレーズ予測。ゴーストテキストで非侵襲的に提案し、Tabキーで確定。自動確定は禁止。 |
| [Adr0025](Adr0025_LatentIntentInference.md) | **Latent Intent Inference**<br>`Proposed` | 確定テキストからURLや住所・日付などのエンティティを検出し、「次に取りうる行動」を隠れ意図として推論・提示する。パターンマッチング主体で軽量実装。 |
| [Adr0026](Adr0026_SpeakerClustering.md) | **Speaker Clustering**<br>`Proposed` | アクティブなアプリケーション（VSCode/Slack/Outlookなど）を文脈として検出し、用途別の統計ペルソナ（重み付けレイヤー）を自動切替。完全オフラインで話者スタイルに適応。 |
| [Adr0027](Adr0027_AsrInspiredPipeline.md) | **ASR-Inspired Text Input Pipeline**<br>`Proposed` | 音声認識（ASR）の「信号受理→言語識別→デコード→リスコアリング」パイプラインをキーボード入力に転用。ローマ字バッファ→判別→処理→補正の4段構造を正式に定義。 |
