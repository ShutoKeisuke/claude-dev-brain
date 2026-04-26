# claude-dev-brain 構築プラン

## 目的

`claude-dev-brain` は、Claude Code 上で Web 開発アイデアの探索・評価・出力を支援する Plugin Marketplace 形式のリポジトリとして構築する。

最初のゴールは、以下の流れを小さく動かせる状態にすること。

```txt
/cdb:idea
↓
方向性を1問ずつヒアリング
↓
情報収集の観点整理
↓
Web開発案を3つ生成
↓
評価
↓
採用候補を ideas/ に保存
```

---

## 確定方針

### repo名

```txt
claude-dev-brain
```

### 配布形式

Claude Code Plugin / Marketplace 形式

### plugin名

```txt
cdb
```

### UX方針

- 基本は半自動
- 方向性ヒアリングは1問ずつ
- 自然文起動も狙う
- 明示コマンドとして `/cdb:idea` を用意する

### 初期ユースケース

Web開発案出し

### 出力方針

```txt
ideas/YYYY-MM-DD-title.md
```

各アイデアには status を持たせる。

```txt
status: idea | selected | archived
```

---

## 最終的な初期構成

```txt
claude-dev-brain/
  marketplace.json
  README.md

  plugins/
    cdb/
      plugin.json

      commands/
        cdb-idea.md

      agents/
        direction-interviewer.md
        market-researcher.md
        idea-generator.md
        idea-evaluator.md
        output-writer.md

      skills/
        idea-discovery/
          SKILL.md
          references/
            evaluation-criteria.md
            output-format.md

      templates/
        idea.md
        research.md
        selected-idea.md

  examples/
    idea-bank/
      ideas/
        .gitkeep

  docs/
    usage.md
    development-plan.md
```

---

# 構築ステップ

## Step 1: 最小Plugin構成を作る

### 目的

Claude Code Plugin として認識できる最小構成を作る。

### 作るもの

```txt
marketplace.json
plugins/cdb/plugin.json
README.md
```

### 完了条件

- repo構成がPlugin Marketplace形式になっている
- `cdb` plugin の名前・説明・バージョンが定義されている
- READMEに目的が書かれている

---

## Step 2: `/cdb:idea` コマンドを作る

### 目的

最初の入口を作る。

### 作るもの

```txt
plugins/cdb/commands/cdb-idea.md
```

### この段階の挙動

まだAgentやSkillは使わなくてよい。  
まずは `/cdb:idea` を実行したら、方向性ヒアリングを開始する指示だけ書く。

### 完了条件

- `/cdb:idea` の役割が明確
- 方向性ヒアリングへ誘導できる
- 自然文でも使われやすい説明になっている

---

## Step 3: `direction-interviewer` Agentを作る

### 目的

Web開発案を出す前に、方向性を擦り合わせる。

### 作るもの

```txt
plugins/cdb/agents/direction-interviewer.md
```

### 質問項目

最初は以下だけでよい。

```txt
1. 方向性
   - 海外にあるが日本に少ないサービス
   - トレンド起点
   - 低予算の個人開発
   - BtoB SaaS
   - 自由入力

2. 予算感

3. 開発期間

4. 個人開発かチーム開発か

5. 出力先
   - ideas/
   - docs/ideas/
   - その他
```

### UX方針

- 1問ずつ聞く
- 選択肢を出す
- 回答が曖昧でも仮説を置いて進める
- 聞きすぎない

### 完了条件

- 方向性ヒアリングが自然に進む
- 最低限の前提がまとまる
- 次の research に渡せる入力ができる

---

## Step 4: `market-researcher` Agentを作る

### 目的

本格的なWeb検索の前に、調査観点を整理する。

### 作るもの

```txt
plugins/cdb/agents/market-researcher.md
```

### 初期スコープ

最初は実調査よりも、以下の観点整理を優先する。

```txt
- 調査すべき市場
- 競合候補
- 海外事例
- 日本での未充足ニーズ
- トレンド確認ポイント
- 収益化の仮説
```

### 完了条件

- 案出しに使える調査メモが出る
- 競合/市場/トレンドの観点が整理される
- 後からWeb検索強化しやすい構造になっている

---

## Step 5: `idea-generator` Agentを作る

### 目的

ヒアリング結果と調査観点からWeb開発案を生成する。

### 作るもの

```txt
plugins/cdb/agents/idea-generator.md
```

### 初期出力数

```txt
3案
```

### 各案に含める情報

```txt
- サービス名
- 概要
- ターゲット
- 解決する課題
- 提供価値
- MVP機能
- 技術スタック案
- 収益化方法
- 日本での勝ち筋
- 懸念点
```

### 完了条件

- 3つのWeb開発案が出る
- それぞれ比較可能な粒度になっている
- 評価Agentに渡せる形式になっている

---

## Step 6: `idea-evaluator` Agentを作る

### 目的

生成された案を評価し、採用候補を選びやすくする。

### 作るもの

```txt
plugins/cdb/agents/idea-evaluator.md
plugins/cdb/skills/idea-discovery/references/evaluation-criteria.md
```

### 評価軸

```txt
- 市場性
- 実装難易度
- 収益化しやすさ
- 個人開発適性
- 日本での勝ち筋
- 初期集客のしやすさ
- 継続運用コスト
```

### 完了条件

- 各案にスコアが付く
- 採用候補が1つ以上提示される
- なぜその案が良いのか説明される

---

## Step 7: `output-writer` Agentを作る

### 目的

採用候補をファイルとして保存しやすい形式に整える。

### 作るもの

```txt
plugins/cdb/agents/output-writer.md
plugins/cdb/templates/idea.md
plugins/cdb/templates/selected-idea.md
```

### 出力先

初期値は以下。

```txt
ideas/YYYY-MM-DD-title.md
```

### Markdown形式

```md
---
title: ""
status: selected
created_at: YYYY-MM-DD
source: cdb
---

# 概要

# ターゲット

# 解決する課題

# MVP

# 技術スタック

# 収益化

# 評価

# 次のアクション
```

### 完了条件

- 採用候補をMarkdown化できる
- outputの責務が他Agentから分離されている
- 後から保存先や形式を変更しやすい

---

## Step 8: `idea-discovery` Skillを作る

### 目的

Web開発案出しに関する共通知識・手順・評価基準をSkillとしてまとめる。

### 作るもの

```txt
plugins/cdb/skills/idea-discovery/SKILL.md
plugins/cdb/skills/idea-discovery/references/output-format.md
```

### 役割

Skillは主役ではなく、AgentやCommandから参照される共通知識として扱う。

### 含める内容

```txt
- Web開発案出しの基本プロセス
- 良いアイデアの条件
- MVPに落とす基準
- 評価観点
- 出力フォーマット
```

### 完了条件

- 自然文で「開発案を出して」と言った時に利用されやすい
- Agent間で判断基準がブレにくい
- 後からreferencesを増やせる

---

## Step 9: 一連の流れを統合する

### 目的

`/cdb:idea` から一連のワークフローを実行できるようにする。

### 流れ

```txt
/cdb:idea
↓
direction-interviewer
↓
market-researcher
↓
idea-generator
↓
idea-evaluator
↓
output-writer
```

### 完了条件

- 最初から最後まで一通り動く
- 3案生成される
- 1案以上を保存用Markdownにできる
- 不要に質問しすぎない

---

## Step 10: 実利用テスト

### 目的

実際の使い方に近い形で検証する。

### テストパターン

#### パターン1: 開発repoで使う

```txt
1. 新規開発repoを作る
2. cdb pluginを導入する
3. /cdb:idea を実行
4. docs/ideas/ or ideas/ に出力する
```

#### パターン2: idea bank repoで使う

```txt
1. idea-bank repoを作る
2. cdb pluginを導入する
3. 複数の案を生成
4. ideas/ に蓄積する
```

### 完了条件

- どちらのパターンでも違和感なく使える
- 出力先だけ変えれば運用できる
- Plugin側の設計を大きく変えずに済む

---

## Step 11: READMEと使い方を整える

### 目的

後から自分でも使いやすい状態にする。

### 作るもの

```txt
README.md
docs/usage.md
docs/development-plan.md
```

### READMEに書くこと

```txt
- claude-dev-brain の目的
- Pluginとしての導入方法
- /cdb:idea の使い方
- 想定ユースケース
- ディレクトリ構成
```

### 完了条件

- 半年後に見ても使い方がわかる
- 他のrepoに導入しやすい
- 今後の拡張方針がわかる

---

# 優先順位

## Phase 1: 最小で動かす

```txt
Step 1
Step 2
Step 3
Step 5
```

まずは、調査や保存を簡略化してもよいので、  
`/cdb:idea` から方向性ヒアリング → 3案生成まで動かす。

## Phase 2: 評価と保存

```txt
Step 6
Step 7
```

案の評価とMarkdown出力を追加する。

## Phase 3: 調査とSkill整備

```txt
Step 4
Step 8
```

market-researcher と idea-discovery Skill を整える。

## Phase 4: Pluginとして仕上げる

```txt
Step 9
Step 10
Step 11
```

一連の統合、実利用テスト、README整備を行う。

---

# 最初にやること

次にやるべきことはこれ。

```txt
Step 1: 最小Plugin構成を作る
```

作成対象。

```txt
marketplace.json
plugins/cdb/plugin.json
README.md
```

この3つを作って、まずは `claude-dev-brain` を Plugin Marketplace 形式のrepoとして成立させる。
