# 3. 解決方法 (Solution / Implementation)

## 解決の鍵: Rustコアによる「事後判別（Post-Detection）」
「悩み」を「未来」に変えるための、具体的かつ段階的な実装アプローチです。

### 3.1 技術スタック (Technical Stack)
複数の環境で「全く同じ確率判断ロジック」を共有し、どのデバイスでも同じ使い心地を実現するために Rust をコアに採用します。

#### Core Engine
- **Language**: **Rust**
  - 高性能、メモリ安全、決定論的なレイテンシ。
  - Windows (C++) / Android (JNI) / Browser (WASM) とのシームレスな連携。

#### Platform Adapters
- **Windows**: **TSF (Text Services Framework) + C++ Bridge**
  - Windows APIのネイティブ統合（IME APIの難易度への対応）。
- **Android**: **Kotlin + Android IME Service + Rust NDK**
  - Java/Kotlin層でのIMEサービス提供と、RustコアへのJNIブリッジ。
- **Browser**: **WebAssembly (WASM) + Rust**
  - ブラウザ拡張機能およびPWAでの動作。Service Worker上でのオフライン実行。

#### NLP & Inference (Offline)
- **Morphological Analysis**: **MeCab**
  - カスタムRust FFIを介した高速な形態素解析と助詞・文末判定。
- **Classification & Correction**: **Small Quantized Models**
  - 30MB〜50MB程度の軽量統計モデル（N-gram, Logistic Regression）。
- **Ambiguity Handling**: **Trie & Bloom Filters**
  - 高速な前方一致検索と、編集距離（Levenshtein <= 2）に基づく誤字補完。

### 3.2 判定アルゴリズム (Intent Routing)
1. **Raw Romaji Buffer**: すべての入力を一旦バッファに蓄積。
2. **Post-Classification**: Enter押下時、以下の特徴量から意図を分類。
   - `git`, `npm` 等のコマンド語彙（CLI）
   - `-m`, `--help`, `/` 等の記号パターン（CLI）
   - 助詞 (`ha`, `ga`, `wo`) や文末の出現確率（日本語自然文）
3. **Smart Correction**: 編集距離（Levenshtein）とキーボード配列に基づき、誤字を補完。

### 3.3 段階的ブレイクダウン (Implementation Roadmap)
- **Phase 1: Basic Intent Router**
  - ローマ字入力 → CLIか日本語かの自動判別 → 基本変換。
- **Phase 2: Contextual Assistance**
  - 文脈メモリの実進。直前の入力に応じた確率補正、打ち間違い補正の導入。
- **Phase 3: Semantic Completion**
  - MeCab等による深い解析と意味的な補完。ユーザーの癖（Personal Delta）の学習。

### 3.4 性能目標 (Performance Target)
- **Latency**: < 10ms (Commit to Result)
- **Memory**: < 50MB
- **Binary Size**: < 30MB (Core exclusion of dictionary)

### 3.5 詳細な技術決定 (ADRs)
具体的な実装の詳細は、以下の **ADRs (Architecture Decision Records)** を参照してください。
- [ADR Index (設計決定記録)](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/AdrIndex.md)
