# be-semantic

技術的制約からではなく、**意味**からアプリケーションを設計する方法論。

**be-semantic** は、ユーザーの意図と実装を意味論（セマンティクス）でつなぐドメインファーストの設計ワークフロー。[Be Framework](https://github.com/be-framework/be)、[ALPS](https://alps.io/)、データ駆動スキーマ設計を一貫したプロセスに統合する。

## 問題

多くのAPI設計は技術から始まる：
> 「RESTful APIが必要だ。エンドポイントとスキーマを設計しよう。」

この結果、スキーマはデータベーステーブルやフレームワークの慣習を反映したものになり、本来のドメインを表現しない。

**be-semantic は逆から始める：**
> 「ユーザーは何をしたいのか？それはどういう*意味*なのか？」

## ワークフロー

```
ストーリー → ALPS → Fake → 合意 → スキーマ → Be実装
```

| ステップ | 内容 | 目的 |
|---------|------|------|
| **ストーリー** | 自然言語のユーザーストーリー | 設計をユーザーの意図に根ざす |
| **ALPS** | アプリの語彙と状態遷移 | 「何ができるか」を定義する（「どう実装するか」ではなく） |
| **Fake** | 50件以上のリアルなフェイクデータ生成 | 制約を決める前に現実を観察する |
| **合意** | フェイクデータをステークホルダーと確認 | 共通理解をアラインする |
| **スキーマ** | 観察から導いたJSONスキーマ | 合意した現実を制約として記録する |
| **Be実装** | BeフレームワークのBeing/Finalとして実装 | 技術ではなくドメインを映すコード |

## 核心となる洞察

> 制約は抽象的なルールから規定するのではなく、観察した現実から導くべきだ。

50件のフェイクデータを生成して眺めると、発見がある：
- 実際に誰かが書く最長のタイトルは41文字だった（「VARCHARだから255」ではなく）
- 常に存在するフィールドと、ほとんど使われないフィールドがある
- 考えもしなかったエッジケースが自然に現れる

スキーマは*推測*ではなく、*観察の記録*になる。

## クイックスタート

```bash
git clone https://github.com/koriym/be-semantic.git
cd be-semantic
```

`example/` ディレクトリに、このワークフローで作ったTodoアプリの完全な例がある：

- [`example/story/`](example/story/) — ユーザーストーリー
- [`example/alps/`](example/alps/) — ALPSプロファイル
- [`example/fake/`](example/fake/) — 50件のフェイクデータ
- [`example/schema/`](example/schema/) — 導出したJSONスキーマ

## ドキュメント

- [**チュートリアル**](docs/ja/tutorial.md) — ゼロからBeアプリを作る（ここから始める）
- [コンセプト](docs/ja/concept.md) — 哲学と動機
- [ワークフロー](docs/ja/workflow.md) — ステップバイステップガイド
- [考察](docs/ja/insights.md) — 観察と振り返り

English documentation:
- [Concept](docs/en/concept.md)
- [Workflow](docs/en/workflow.md)
- [Insights](docs/en/insights.md)

## 関連リンク

- [Be Framework](https://github.com/be-framework/be)
- [ALPS仕様](https://alps.io/)
- [Semantic-Exメソッド](https://koriym.github.io/blog/2025/08/10/semantic-method-en)
- [app-state-diagram](https://github.com/koriym/app-state-diagram)
