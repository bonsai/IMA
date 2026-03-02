# SmartIME 技術スタック決定書

> **目的**: C# と Rust の役割分担を明確にし、DB・ML を含む全技術スタックを確定する。本ドキュメントは既存の ADR 群（ADR0001〜ADR0020）を踏まえ、ADR0021〜ADR0023 として追加された決定を統合した全体像である。

---

## 1. 設計哲学の再確認

SmartIME の技術スタックは、以下の三つの不変原則によって制約される。

| 原則 | 内容 |
|---|---|
| **Offline First** | 外部 API・クラウド依存を一切排除。全推論はデバイス内で完結する。 |
| **Sub-10ms Latency** | キーストロークへの追従を保証するため、分類・変換のエンドツーエンド遅延を 10ms 以内に抑える。 |
| **User Sovereignty** | 自動確定・強制変換を禁止。ユーザーが常に最終決定権を持つ。 |

---

## 2. 技術スタック全体図

```
┌─────────────────────────────────────────────────────────────────┐
│                      Platform Layer                             │
│                                                                 │
│  [C# WPF 設定UI]    [Windows TSF IME]  [Android IME Service]  │
│  (ADR0021)          C++ Bridge         Kotlin + JNI            │
│      │                    │                    │               │
│      └────────────────────┼────────────────────┘               │
│                           │ P/Invoke / JNI / WASM              │
└───────────────────────────┼─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                  Rust Core (smart_ime_core)                     │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Input Pipeline                                          │  │
│  │  Raw Romaji Buffer → Segmenter → Discriminator → Router  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌─────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │  ML / NLP Layer │  │  Dictionary Layer │  │  Storage     │  │
│  │                 │  │                  │  │  Layer       │  │
│  │  Vibrato        │  │  System Trie     │  │  SQLite      │  │
│  │  (Morphology)   │  │  (mmap/Zero-Copy)│  │  (Personal   │  │
│  │                 │  │                  │  │  Delta)      │  │
│  │  Tract + ONNX   │  │  Domain Plugin   │  │              │  │
│  │  (Inference)    │  │                  │  │  redb        │  │
│  │                 │  │  Personal Delta  │  │  (Cache)     │  │
│  │  N-gram Model   │  │  (redb/SQLite)   │  │              │  │
│  └─────────────────┘  └──────────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. レイヤー別スタック詳細

### 3.1 コアエンジン (Rust)

Rust はシステム全体の心臓部であり、全プラットフォームで共有される唯一の真実の源（Single Source of Truth）である。ADR0001 で確定済み。

| コンポーネント | 技術 | 根拠 |
|---|---|---|
| 入力バッファ | Rust (カスタム実装) | ゼロコピー・ロックフリー設計 |
| セグメンター | Rust (カスタム実装) | 境界検出アルゴリズム |
| 判別モデル (Discriminator) | N-gram + ルールベース (Rust) | 1-2ms 以内の超高速判別 |
| 処理モデル (Processor) | Tract + ONNX / Vibrato | 専門処理の分離 (ADR0014) |
| SIMD 最適化 | Rust `std::simd` / `packed_simd` | 並列文字処理 (ADR0019) |

### 3.2 形態素解析 (ML/NLP)

**採用: Vibrato** (ADR0023 にて決定)

Vibrato は MeCab の純 Rust 再実装であり、MeCab 互換辞書（IPADIC/UniDic）をそのまま利用できる。ベンチマークでは MeCab の 2 倍以上の速度を示しており、かつ C++ FFI を必要としないため、クロスプラットフォームビルドが大幅に簡素化される。

| 比較対象 | 言語 | トークン化速度 | WASM対応 | 採用可否 |
|---|---|---|---|---|
| **Vibrato** | **Rust (純正)** | **MeCabの2倍** | **検討中** | **✅ 採用** |
| Lindera | Rust (純正) | 22µs | ✅ (13MB) | 代替候補 |
| MeCrab | Rust (純正) | 高速 | 未確認 | 代替候補 |
| MeCab | C++ | 高速 | ✗ | ✗ |
| Sudachi | Java | 66µs (WASM) | ✅ | ✗ (非Rust) |

### 3.3 推論エンジン (ML)

**採用: Tract** (ADR0023 にて決定)

Tract は純 Rust 製の ONNX 推論エンジンであり、エッジデバイス向けに最適化されている。モデルを ONNX 形式でエクスポートすることで、Python 等で訓練したモデルを Rust から直接実行できる。

| 比較対象 | 言語 | 特徴 | 採用可否 |
|---|---|---|---|
| **Tract** | **Rust (純正)** | **軽量、ONNX、エッジ向け** | **✅ 採用** |
| ort (ONNX Runtime) | Rust bindings | 高機能、C++依存 | 将来の拡張候補 |
| candle | Rust (純正) | HuggingFace製、GPU対応 | Phase 3 以降の候補 |
| burn | Rust (純正) | 学習・推論両対応 | 将来の候補 |

### 3.4 データベース (DB)

**採用: SQLite (rusqlite) + redb** (ADR0022 にて決定)

Personal Delta の永続化には SQLite を、高速キャッシュには純 Rust 製 KV ストアの redb を採用する二層構造とする。

| 用途 | 採用DB | 根拠 |
|---|---|---|
| Personal Delta 永続化 | **SQLite (rusqlite)** | ACID、実績、クロスプラットフォーム、SQL対応 |
| 高速 KV キャッシュ | **redb** | 純Rust、高速読み書き、LMDB風 |
| System辞書 | mmap バイナリ (Trie) | ゼロコピー読み取り (ADR0017) |
| Domain辞書 | プラグイン形式バイナリ | 動的ロード (ADR0016) |

### 3.5 プラットフォームアダプター

| プラットフォーム | UI/アダプター | コア連携 | 根拠 |
|---|---|---|---|
| **Windows (IME)** | C++ / TSF | Rust DLL (直接リンク) | TSF は C++ COM が必須 (ADR0009) |
| **Windows (設定UI)** | **C# WPF** | P/Invoke (csbindgen) | **ADR0021 新規決定** |
| **Android** | Kotlin + IME Service | JNI → Rust NDK | Android IME API は Java/Kotlin (ADR0009) |
| **Browser** | Web Extension (MV3) | WASM | Service Worker でオフライン動作 (ADR0009) |

---

## 4. C# の役割と Rust との境界線

本プロジェクトにおける C# の役割は、**リアルタイム入力パスの外側**に厳密に限定される。

```
[リアルタイム入力パス] ← Rust のみ。C# は絶対に介入しない
    キーストローク → バッファ → 判別 → 変換 → 出力

[設定・管理パス] ← C# が担当
    設定UI → P/Invoke → Rust Core DLL → 設定永続化
```

C# が担当する具体的なコンポーネントは以下の通りである。

| コンポーネント | 技術 | 説明 |
|---|---|---|
| 設定パネル | C# WPF | Manual/Assisted/Auto モード切替、閾値調整 |
| 辞書管理 | C# WPF | Domain辞書のインポート・エクスポート |
| Personal Delta ビューア | C# WPF | 学習履歴の可視化・削除・リセット |
| インストーラー | C# / WiX | Rust DLL と C# UI を含む統合インストーラー |
| デバッグ・ログビューア | C# WPF | 開発・デバッグ用の入力ログ可視化ツール |

### C# ↔ Rust インターフェース設計

```csharp
// csbindgen が自動生成する C# 側コード例
[DllImport("smart_ime_core.dll")]
public static extern void set_mode(AssistanceMode mode);

[DllImport("smart_ime_core.dll")]
public static extern void reset_personal_delta();

[DllImport("smart_ime_core.dll")]
public static extern IntPtr get_stats_json();
```

```rust
// Rust 側の公開 API
#[no_mangle]
pub extern "C" fn set_mode(mode: AssistanceMode) { ... }

#[no_mangle]
pub extern "C" fn reset_personal_delta() { ... }

#[no_mangle]
pub extern "C" fn get_stats_json() -> *mut c_char { ... }
```

---

## 5. 実装フェーズとスタック適用範囲

| フェーズ | 目標 | 使用スタック |
|---|---|---|
| **Phase 1** | 基本的な意図ルーター (CLI vs JP) | Rust Core, ルールベース判別, TSF/Kotlin Adapter |
| **Phase 2** | 文脈補正・誤字補正 | + Vibrato, N-gram (Tract), redb キャッシュ |
| **Phase 3** | 意味補完・個人適応学習 | + SQLite (Personal Delta), Corpus Metabolism |
| **Phase 4** | 設定UI・管理ツール | + C# WPF, csbindgen, インストーラー |

---

## 6. 性能目標と制約のまとめ

| 指標 | 目標値 | 担保する技術 |
|---|---|---|
| 分類レイテンシ | < 10ms | Rust, SIMD, ルールベース Layer 1 |
| メモリ占有 | < 50MB | mmap, Bloom Filter, 量子化モデル |
| モデルサイズ | < 30MB (辞書除く) | N-gram + 量子化 Tiny Transformer |
| 起動時間 | < 500ms | mmap ゼロコピー辞書ロード |
| 学習更新 | < 1ms | redb KV ストア, ロックフリー設計 |

---

## 7. ADR 対応表

| ADR ID | タイトル | ステータス | 本ドキュメントとの関連 |
|---|---|---|---|
| ADR0001 | Core Language Choice (Rust) | Accepted | §3.1 コアエンジン |
| ADR0003 | Offline Constraint | Accepted | §1 設計哲学 |
| ADR0004 | Morphological Engine (MeCab) | Proposed | §3.2 → Vibrato に更新 |
| ADR0008 | Local Learning (Personal Delta) | Proposed | §3.4 DB |
| ADR0009 | Platform Adapters | Accepted | §3.5 プラットフォーム |
| ADR0010 | Lightweight Inference | Proposed | §3.3 推論エンジン |
| ADR0013 | Local SLM as Intent Classifier | Accepted | §3.3 推論エンジン |
| ADR0014 | Discriminator-Processor Separation | Accepted | §3.1 コアエンジン |
| ADR0016 | Layered Dictionary Partitioning | Accepted | §3.4 DB |
| ADR0017 | Memory-Mapped Zero-Copy Storage | Accepted | §3.4 DB |
| ADR0018 | Corpus Metabolism | Proposed | §5 Phase 3 |
| ADR0019 | High Performance Optimization | Proposed | §3.1 SIMD |
| **ADR0021** | **C# UI for Configuration** | **Proposed** | **§3.5, §4 C#の役割** |
| **ADR0022** | **Database for Personal Delta** | **Proposed** | **§3.4 DB** |
| **ADR0023** | **ML Strategy for Classification** | **Proposed** | **§3.2, §3.3 ML** |
