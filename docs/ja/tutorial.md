# チュートリアル：ゼロからBeアプリを作る

このチュートリアルでは、be-semanticのワークフローに沿って、Todoアプリを最初から作る。
`example/` ディレクトリの完成形がゴール。

## 前提条件

- PHP 8.3+
- [Composer](https://getcomposer.org/)
- [asd (app-state-diagram)](https://github.com/alps-asd/app-state-diagram)

### asd のインストール

```bash
# Homebrew（推奨）
brew install alps-asd/asd/asd

# または Composer
composer global require koriym/app-state-diagram
```

インストール確認：
```bash
asd --version
```

---

## Step 1：ストーリーを書く

プロジェクトディレクトリを作り、ユーザーストーリーから始める。

```bash
mkdir my-todo
cd my-todo
mkdir -p story alps fake schema src/{Being,Final,Reason}
```

`story/main.md` を作成する：

```markdown
# アプリの概要
シンプルなTodoアプリ。タスクを作成・管理・完了できる。

## ユーザーストーリー

### タスク一覧を見る
ユーザーとして、すべてのTodoを一覧で見たい。
なぜなら、今何をすべきか把握したいから。

### タスクを作成する
ユーザーとして、新しいタスクをタイトルと任意のメモ付きで作成したい。
なぜなら、やることを記録したいから。

...（続きは example/story/main.md を参照）

## エンティティ
- **Todo**: id, title, memo（任意）, isCompleted, createdAt
```

**チェックポイント：** 各ストーリーに「なぜなら」節はあるか？エンティティと属性は明確か？

---

## Step 2：ALPSプロファイルを作る

ストーリーをもとに、アプリの語彙と状態遷移を定義する。

`alps/todo.json` を作成する（`example/alps/todo.json` を参照）。

ALPSの基本構造：
```json
{
  "$schema": "https://alps-io.github.io/schemas/alps.json",
  "alps": {
    "title": "Todo App",
    "descriptor": [
      // セマンティクスディスクリプタ（データ要素）
      {"id": "todoId", "title": "Todo ID"},
      {"id": "todoTitle", "title": "タイトル"},

      // 表現（複合ビュー）
      {"id": "TodoList", "descriptor": [...]},

      // 操作（遷移）
      {"id": "goTodoList", "type": "safe", "rt": "#TodoList"},
      {"id": "doCreateTodo", "type": "unsafe", "rt": "#TodoList"}
    ]
  }
}
```

### 状態遷移図を確認する

```bash
# バリデーション + HTML生成
asd alps/todo.json

# ブラウザで確認
open alps/todo.html
```

状態遷移図でアプリの全体像が見える。ステークホルダーへの説明にもそのまま使える。

**チェックポイント：** ストーリーの全アクションがALPSに反映されているか？

---

## Step 3：フェイクデータを生成する

ALPSを見ながら、AIに50件のリアルなフェイクデータを生成させる。

**プロンプト例：**
```
以下のALPSプロファイルに基づいて、Todoアプリの現実的なフェイクデータを50件生成してください。
実際のユーザーが日常的に使うような内容にしてください。

[alps/todo.json の内容を貼り付ける]

出力形式: JSON配列
```

生成したデータを `fake/data-50.json` に保存する。

`example/fake/data-50.json` も参考にできる。

**チェックポイント：** データは現実的か？ランダムではなく「実際のユーザーが書きそう」か？

---

## Step 4：フェイクデータを観察して合意する

50件のデータを眺めて、制約を発見する。

**観察すること：**

```bash
# タイトルの長さ分布を確認
cat fake/data-50.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
lengths = [len(d['todoTitle']) for d in data]
print(f'最短: {min(lengths)}, 最長: {max(lengths)}, 平均: {sum(lengths)/len(lengths):.1f}')
"

# memoがnullの件数を確認
cat fake/data-50.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
null_count = sum(1 for d in data if d.get('todoMemo') is None)
print(f'memo=null: {null_count}/{len(data)}件')
"
```

**発見したことをメモする：**
```
観察結果：
- todoTitle: 最長41文字 → maxLength: 80 に設定
- todoMemo: 50件中35件がnull → optional
- isCompleted: true/falseが混在 → boolean
- createdAt: 常に存在 → required
```

ステークホルダーと一緒にデータを確認し、合意を取る。

---

## Step 5：スキーマを書く

観察と合意から、JSONスキーマを書く。

`schema/todo-list.json`：
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["items"],
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "required": ["todoId", "todoTitle", "isCompleted", "createdAt"],
        "properties": {
          "todoTitle": {
            "type": "string",
            "maxLength": 80
          }
        }
      }
    }
  }
}
```

`example/schema/` の完成形も参照。

**ポイント：** `maxLength: 255` ではなく、観察した値から導くこと。

---

## Step 6：Beアプリを実装する

### プロジェクトのセットアップ

```bash
composer init \
  --name="yourname/my-todo" \
  --require="be-framework/be:0.x-dev" \
  --require="ray/di:^2.18" \
  --require-dev="phpunit/phpunit:^10.0"

composer install
```

### Being（状態）を実装する

スキーマの `required` → Being のコンストラクタ引数  
スキーマの `properties` → 型とバリデーション

```php
// src/Being/TodoList.php
namespace MyTodo\Being;

use Be\Framework\Being;

#[Being]
class TodoList
{
    public function __construct(
        /** @var TodoItem[] */
        public readonly array $items
    ) {}
}
```

`be-framework-demos/demos/` の実装例も参考に。

### テストを実行する

```bash
./vendor/bin/phpunit
```

---

## 完成チェックリスト

- [ ] `story/main.md` — 全機能のユーザーストーリー（「なぜなら」節あり）
- [ ] `alps/todo.json` — バリデーション済みALPSプロファイル
- [ ] `alps/todo.html` — `asd` で生成した状態遷移図
- [ ] `fake/data-50.json` — 50件のリアルなフェイクデータ
- [ ] 観察メモ — 各フィールドの制約の根拠
- [ ] `schema/todo-list.json` — 観察から導いたスキーマ
- [ ] `schema/todo-detail.json` — 観察から導いたスキーマ
- [ ] `src/` — Beアプリの実装
- [ ] テスト全グリーン

---

## 参考リンク

- [Be Framework](https://be-framework.github.io/)
- [ALPS仕様](https://alps.io/)
- [app-state-diagram](https://www.app-state-diagram.com/)
- [be-framework-demos](https://github.com/be-framework/demos)
- [Semantic-Exメソッド](https://koriym.github.io/blog/2025/08/10/semantic-method-en)
