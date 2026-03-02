# SmartIME (Post-Detection IME)

「今どっちモードだっけ？」を過去にする。
入力は状態ではなく、確率の流れ。

## 📖 プロジェクト構造

SmartIMEのデザインと設計を以下の3つのフェーズで展開しています。

### 1. [悩みと現状 (Pain & Current Status)](file:///c:/Users/dance/zone/vibe/IME3/doc/1_Pain/PainAndCurrentStatus.md)
なぜ既存のIMEでは不十分なのか。ユーザーが抱える「モード切替」「多言語混在」「誤字」の苦しみ。
- **[異分野からの着想 (Pollination Ideas)](file:///c:/Users/dance/zone/vibe/IME3/doc/1_Pain/PollinationIdeas.md)**

### 2. [目指す未来 (Desired Future / Vision)](file:///c:/Users/dance/zone/vibe/IME3/doc/2_Future/DesiredFuture.md)
IMEが「意味を理解するルーター」になった後の世界。状態からの解放と、誰もがフロー状態で入力できる未来。
- **[哲学と美学 (Philosophy & Aesthetics)](file:///c:/Users/dance/zone/vibe/IME3/doc/2_Future/PhilosophyAndAesthetics.md)**
- **[辞書と世界 (Dictionary & The World)](file:///c:/Users/dance/zone/vibe/IME3/doc/2_Future/DictionaryAndWorld.md)**

### 3. [解決方法 (Solution / Implementation)](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/SolutionArchitecture.md)
Rust/WASMを用いた具体的な解決策、技術スタック、および実装ロードマップ。
- **[局所言語モデル戦略 (Local SLM Strategy)](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/LocalSlmStrategy.md)**
- **[判別と処理の分離 (Dual Model Architecture)](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/DualModelArchitecture.md)**
- **[辞書システム (Dictionary Architecture)](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/Dictionary.md)**

---

## 🛠 詳細な技術情報
- **[ADR Index (設計決定記録)](file:///c:/Users/dance/zone/vibe/IME3/doc/3_Solution/ADR/AdrIndex.md)**: 意思決定の歴史。

Developed with focus on Performance, Privacy, and Human Creativity.
