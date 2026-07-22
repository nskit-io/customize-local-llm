[🇬🇧 English](README.md) | [🇰🇷 한국어](README.ko.md)

# Hybrid LLM Architecture

**性格はローカルモデル、ファクトはクラウドモデル — プライバシー優先のAIキャラクターアーキテクチャ。**

> [NVatar](https://github.com/nskit-io/nvatar)プロジェクトより — ローカルハードウェアで完全に動作するAIアバターチャットシステム。

---

## なぜ作ったのか

LLMルーティングは存在します（RouteLLM、FrugalGPT、Semantic Router）。しかし既存の研究は**コスト**や**難易度**の最適化に集中しています — このパターンは代わりに**機能別分離**を扱います：

| レイヤー | 場所 | 理由 |
|---------|------|------|
| 性格＆会話 | **ローカル**（Gemma 26B） | プライバシー、レイテンシ、キャラクター一貫性 |
| ファクト＆検索 | **クラウド**（Claude via CSW） | 正確性、ツールアクセス、Web検索 |

## 問題

ローカルLLMは性格は得意だが**事実を捏造する**。クラウドLLMは正確だが雑談には**コスト過多**で、プライベートな会話が第三者に露出される。

**解決：難易度ではなく機能別に分離。**

## アーキテクチャ

```
ユーザーメッセージ
    │
    ▼
コンテキストルーター（Gemma、ローカル）
    │
    ├─ T1-T4, T8-T10 ──→ Gemma（ローカル）
    │   雑談、感情、アドバイス、   性格レイヤー
    │   意見、記憶、挨拶          低レイテンシ
    │
    └─ T5-T7 ──────────→ CSW → Claude（クラウド）
        事実、ニュース、推薦       知識レイヤー
                                 正確 + Web検索
```

## 10の会話タイプ

| コード | タイプ | ルーティング | Thinking |
|--------|--------|------------|----------|
| T1 | 雑談 | Gemma | OFF |
| T2 | 感情共有 | Gemma | OFF |
| T3 | アドバイス | Gemma | OFF |
| T4 | 深い議論 | Gemma | **ON** |
| T5 | 事実の質問 | **クラウド** | - |
| T6 | ニュース | **クラウド** | - |
| T7 | おすすめ | **クラウド** | - |
| T8 | 記憶参照 | Gemma | OFF |
| T9 | アバターQ | Gemma | OFF |
| T10 | 挨拶/別れ | Gemma | OFF |

**核心的洞察：** キャラクター会話の70-80%がT1-T4（性格レイヤー）。クラウドは事実の正確性が必要な20-30%にのみ使用。

## 「調べてみようか？」パターン

こっそりクラウドにルーティングせず、キャラクターが**許可を求める**：

```
ユーザー：「東京の天気どう？」
アバター：「調べてみようか？」
→ ユーザーが自然な肯定応答で確認
→ CSW WebSearch実行
アバター：「東京今18度だって！午後雨降るらしいから傘持ってって〜」
```

## なぜ大きなクラウドモデルを使わないのか？

| 懸念事項 | ローカル優先ハイブリッド | クラウドのみ |
|---------|----------------------|------------|
| プライバシー | 会話はローカルに保持 | 全データがクラウドへ |
| レイテンシ | ほとんどのメッセージが即座に応答 | すべて3-5秒 |
| コスト | 20%のメッセージのみクラウド | 100%クラウド |
| オフライン | インターネットなしでチャット可能 | オフライン不可 |

## 関連プロジェクト

- [**avatar-chat**](https://github.com/nskit-io/avatar-chat) — GemmaキャラクターAIプロンプトエンジニアリング
- [**chat-like-human-memory**](https://github.com/nskit-io/chat-like-human-memory) — 9D感情トラッキング + 性格進化 + 3層記憶
- [**vrm-studio**](https://github.com/nskit-io/vrm-studio) — Three.js + WebSocket 3D VRMアバターチャットルーム
- [**CSW**](https://github.com/nskit-io/csw) — クラウドレイヤーを動かすClaude Subscription Worker
- [**portable-ai-companion**](https://github.com/nskit-io/portable-ai-companion) — クロスアプリフランチャイズアーキテクチャ

## ライセンス

MIT License - [LICENSE](LICENSE)参照

---

## この研究をサポートしてください

NVatarは、ハイブリッドAIアーキテクチャの最先端を探求する独立R&Dプロジェクトです。個人創業者が外部資金なしで進めています。

**寄付・投資のお問い合わせ**
- Email: [nskit@nskit.io](mailto:nskit@nskit.io)
- Organization: [NSKit](https://nskit.io) by Neoulsoft Inc.

<p align="center">
  <a href="https://github.com/nskit-io">github.com/nskit-io</a>
</p>
