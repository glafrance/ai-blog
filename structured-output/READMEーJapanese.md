# OpenAI vs Gemini vs Claude API：TypeScriptで構造化データを返す

このシリーズの最初の記事では、きれいな TypeScript ワークスペースを作成し、OpenAI、Gemini、Claude の API を使って、とてもシンプルな最初のスクリプトを実行しました。

**注意：TypeScript 環境のセットアップ、OpenAI・Gemini・Claude の開発者アカウント作成、API キーの取得などがまだ済んでいない場合は、以下の記事を参照してください。**

[Quick Guide to AI APIs](https://gravizot.com/quick-guide-to-ai-apis/)

最初のスクリプトの目的は、意図的にかなり基本的なものでした。

```text
プロンプトを送信する → テキスト応答を受け取る → 結果を表示する
```

これは最初に取り組む場所として適切です。ローカルの TypeScript 環境が動作していること、API キーが正しく設定されていること、そして各プロバイダーの SDK がリモートの AI サービスと正常に通信できることを確認できるからです。

この 2 回目の記事では、そこから一段階進みます。

モデルに単なる文や段落を返してもらうのではなく、**構造化出力**を返してもらいます。

構造化出力とは、モデルの応答が、既知のフィールド名や値の型を持つ JSON オブジェクトのように、特定のデータ形状に従うことを意味します。

この記事では、レスポンスは自由な文章ではありません。TypeScript コードが調べたり、検証したり、利用したりできるオブジェクトであるべきです。

今回の新しい目標は、次のようなものです。

```text
元テキストを送信する → 特定の JSON 形式を要求する → 結果を検証する → データを安全に使う
```

これは重要なステップです。実際のアプリケーションでは、生成された文章の段落以上のものが必要になることが多いからです。プログラムでは、信頼して直接利用できるデータが必要になることがあります。

たとえば、AI モデルに光合成を段落で説明してもらう代わりに、小さなレッスンカードを作成してもらうことができます。そのレッスンカードには、次のような項目を含めます。

* タイトル
* 短い要約
* 重要語句のリスト
* クイズ問題
* 次に学ぶための提案

このようなレスポンスは、アプリケーションで表示したり、保存したり、検索したり、変換したり、システムの別の部分へ送ったりしやすくなります。

## 構造化出力が重要な理由

通常のテキストは柔軟ですが、予測しにくい場合もあります。

モデルがすばらしい説明を返したとしても、プログラム側では、タイトルがどこで終わるのか、要約がどこから始まるのか、重要語句がいくつ含まれているのかが分からない場合があります。

構造化出力は、この問題を解決する助けになります。

次のように依頼する代わりに、

```text
このトピックを説明してください。
```

次のように、より具体的に依頼できます。

```text
次の正確な形を持つオブジェクトを返してください。
title
summary
keyTerms
quizQuestion
nextStep
```

これでモデルが完璧になるわけではありませんが、出力をコードで扱いやすくなります。

## Zod による実行時バリデーションを追加する

この記事では、**Zod** も使用します。Zod は TypeScript と相性のよいバリデーションライブラリです。

TypeScript の型は、コードを書いているときには役に立ちますが、実行時に予期しないデータから自動的に守ってくれるわけではありません。

ここでは API レスポンスがプログラムの実行中に返ってくるため、この点が重要です。

この記事では、スキーマには関連する 2 つの役割があります。

1 つ目は、AI モデルに返してほしい形を説明することです。

2 つ目は、返ってきたデータを信頼する前に、TypeScript コード側でチェックできるようにすることです。

この 2 つは関連していますが、まったく同じものではありません。

API は、モデルの応答を導いたり制約したりするためにスキーマを使います。一方、私たちのアプリケーションは、最終的な結果を実行時に検証するためにスキーマを使います。

言い換えると、次のようになります。

```text
TypeScript はコードを書いているときに役立つ。
Zod はコードが実行されているときに役立つ。
```

この記事では、`LessonCardSchema` を定義し、各 API から返してほしいオブジェクトを説明するために使います。

## 同じ目標、異なる API スタイル

OpenAI、Gemini、Claude はすべて、このような構造化出力のワークフローに使用できます。ただし、同じ考え方をまったく同じ形で表現するわけではありません。

これがこの記事の主な学習目標の 1 つです。

単にスクリプトを動かすことだけが目的ではありません。同じ問題に対して、それぞれのプロバイダーがどのように考えているのかを理解することも目的です。

大まかには、次のようになります。

| プロバイダー | スクリプトが送信するもの | スクリプトが受け取るもの | バリデーション / パースが行われる場所 |
|---|---|---|---|
| **OpenAI** | `zodTextFormat(...)` によって変換された Zod スキーマ | `response.output_parsed` 上のパース済みオブジェクト | 主に OpenAI SDK のヘルパーを通して行われる |
| **Gemini** | Zod スキーマから生成された JSON Schema | `response.text` 上の JSON テキスト | `JSON.parse(...)` と `LessonCardSchema.parse(...)` によって、明示的に自分のコード内で行う |
| **Claude** | `zodOutputFormat(...)` によって変換された Zod スキーマ | `message.parsed_output` 上のパース済みオブジェクト | 主に Anthropic SDK のヘルパーを通して行われる |

構文は異なりますが、考え方は似ています。

```text
形を定義する → その形で返すようモデルに依頼する → 結果を検証する
```

これは最初の記事から見れば小さな一歩ですが、意味のある一歩です。

このパターンを理解すると、より実用的なスクリプトを作れるようになります。

* テキストからフィールドを抽出する
* クイズ問題を生成する
* 記事を再利用可能なオブジェクトに要約する
* フラッシュカードを作成する
* ユーザー入力を分類する
* データベースやフロントエンドアプリケーション向けにデータを準備する

それでも、スクリプトは初心者向けに保ちます。

各スクリプトでは、同じ元テキスト、同じ基本スキーマ、同じ目標を使います。つまり、光合成の短い説明から、簡単な教育用レッスンカードを作成します。

## 💰 アカウント要件とコストに関する注意

このワークスペースでコードを実行する前に、各 AI プロバイダーの開発者アカウントを作成し、課金設定を確認してください。

プラットフォームの URL は時間とともに変更される可能性があるため、正しいオンボーディング用ポータルを見つける一番簡単な方法は、次のダッシュボード名で検索することです。

| プロバイダー | 検索する語句 | 目的 |
|---|---|---|
| **OpenAI** | `OpenAI API Platform Dashboard` | 開発者アカウントを登録し、API クレジットを確保する。 |
| **Google Gemini** | `Google AI Studio` | Gemini の開発者ワークスペースを初期化する。Gemini にはプロトタイピング用の無料枠がある場合がありますが、Google Cloud の課金に接続しない限り、レート制限が適用される場合があります。 |
| **Anthropic Claude** | `Anthropic Console` | 開発者組織を作成し、API クレジットを購入する。 |

---

## ⚠️ コスト管理のベストプラクティス

プログラムからの各リクエストは、リモートの AI サービスにトークンを送信します。これらのリクエストは、API アカウントに実際のコストとして請求される可能性があります。

自動化を実行する前に、次の安全策で自分を守りましょう。

### ハード使用上限を設定する

各プロバイダーの課金ダッシュボードにログインし、たとえば **$5.00** のような厳しめの日次または月次の予算上限を設定します。

これにより、暴走したスクリプトがクレジットを使い切ったり、予期しない請求を発生させたりするのを防ぐ助けになります。

### サンドボックスのループでは本番向けモデルを避ける

単純なテスト、デバッグ、CI ループでは、可能であれば安価で高速なモデルを使用します。

例：

```text
gemini-3.5-flash
```

大規模なエンタープライズ向けモデルは、対象を絞った本番用途に残しておきます。

### ループに上限を設ける

これらのスクリプトの周囲にカスタム自動化を書く場合は、必ず厳密なループ上限を使ってください。

例：

```ts
for (let i = 0; i < 5; i++) {
  // 安全な上限付きループ
}
```

これにより、無限の非同期ループが API エンドポイントにリクエストを送り続けることを防ぐ助けになります。

---

## ⚖️ 法的免責事項

> このガイドのソースコードと実装手順は、教育、情報提供、プロトタイピングのみを目的としています。
>
> 著者は、これらの資料を明示または黙示の保証なしに **「現状のまま」** 提供します。
>
> このワークスペース内のコードを実行することにより、あなたは、自分の開発者アカウント、API キーの安全性、金銭的責任、コスト、および API キーやインフラストラクチャに関連して発生するデータ利用料金について、すべて自己責任であることを認めるものとします。
>
> 著者は、金銭的損失、予期しないクレジット使用、アカウント停止、請求額の急増を含む、直接的、間接的、偶発的、特別、または結果的損害について責任を負いません。
>
> 責任を持って使用し、自動化を実行する前にハードな課金上限を設定してください。

## 📁 プロジェクト構造を確認する

TypeScript 環境をセットアップする記事の手順に従った場合、次のようなフォルダー構造になっているはずです。

`ai-quickstarts/src`

**注意：TypeScript 環境のセットアップ、OpenAI・Gemini・Claude の開発者アカウント作成、API キーの取得などがまだ済んでいない場合は、以下の記事を参照してください。**

[Quick Guide to AI APIs](https://gravizot.com/quick-guide-to-ai-apis/)

## 📦 zod パッケージをインストールする

Zod は、期待するレスポンスの形を定義し、検証するために使います。TypeScript はコードを書いているときに役立ちますが、Zod はプログラムの実行中に実際のデータをチェックするために役立ちます。

`package.json` ファイルがあるディレクトリにいることを確認し、次のコマンドを実行します。

```bash
npm install zod@latest
```

## 新しい API スクリプト用のファイルを作成する

次のスクリプトファイルを用意します。

```text
ai-quickstarts/
├── src/
│   ├── openai-lesson-card.ts
│   ├── gemini-lesson-card.ts
│   └── claude-lesson-card.ts
```

前の記事で作成したスクリプトがあっても問題ありません。

`src` ディレクトリ内にこれら 3 つのファイルを作成し、以下のプロバイダー別のクイックスタートコードを貼り付けます。

- `src/openai-lesson-card.ts`
- `src/gemini-lesson-card.ts`
- `src/claude-lesson-card.ts`

**注意：macOS のターミナルや Linux を使用している場合は、`src` ディレクトリに移動したあと、次のコマンドでファイルを作成できます。**

```bash
touch openai-lesson-card.ts
touch gemini-lesson-card.ts
touch claude-lesson-card.ts
```

## 共通のレッスンカードスキーマ

3 つのスクリプトはすべて、同じ基本的なレスポンス形状を使用します。

その形状を `LessonCardSchema` と呼びます。

このスキーマでは、有効なレッスンカードに次の項目が含まれる必要があると定義します。

* `title`: 文字列
* `summary`: 文字列
* `keyTerms`: 用語 / 定義オブジェクトの配列
* `quizQuestion`: 問題と答えを持つオブジェクト
* `nextStep`: 文字列

3 つのプロバイダーすべてで同じスキーマを使用すると、比較しやすくなります。

API の構文はプロバイダーごとに変わりますが、目標は同じです。

```text
このスキーマに一致する 1 つのレッスンカードオブジェクトを返す。
```

## 各ファイルに次の内容を追加する

## 📄 ファイル 1: `src/openai-lesson-card.ts`

```ts
import OpenAI from "openai";
import { zodTextFormat } from "openai/helpers/zod";
import { z } from "zod";

const client = new OpenAI();

const LessonCardSchema = z.object({
  title: z.string(),
  summary: z.string(),
  keyTerms: z.array(
    z.object({
      term: z.string(),
      definition: z.string(),
    })
  ),
  quizQuestion: z.object({
    question: z.string(),
    answer: z.string(),
  }),
  nextStep: z.string(),
});

type LessonCard = z.infer<typeof LessonCardSchema>;

const sourceText = `
光合成は、植物が自分の食べ物を作るために使う仕組みです。
植物は空気中の二酸化炭素と、土の中の水を取り込みます。
太陽の光のエネルギーを使って、それらを糖と酸素に変えます。
糖は植物が成長する助けになり、酸素は空気中に放出されます。
`;

async function main() {
  try {
    const response = await client.responses.parse({
      model: "gpt-5.5",
      instructions:
        "あなたは初心者にもわかりやすく教える理科の先生です。小学生向けの短いレッスンカードを作成してください。",
      input: sourceText,
      text: {
        format: zodTextFormat(LessonCardSchema, "lesson_card"),
      },
    });

    const lessonCard: LessonCard | null = response.output_parsed;

    if (!lessonCard) {
      throw new Error("OpenAI はパース済みのレッスンカードを返しませんでした。");
    }

    console.log("OpenAI のレッスンカード:");
    console.dir(lessonCard, { depth: null });
  } catch (error) {
    console.error("OpenAI 実行エラー:", error);
  }
}

main();
```

OpenAI のスクリプトでは、Responses API と `responses.parse(...)` を使います。

重要な点は、手動で `JSON.parse(...)` を呼び出していないことです。

その代わりに、Zod スキーマを OpenAI のヘルパー関数に渡します。

```ts
zodTextFormat(LessonCardSchema, "lesson_card")
```

このヘルパーは、OpenAI SDK が必要とする構造化出力フォーマットを与えます。

モデルがスキーマに一致するレスポンスを返すと、SDK は次の場所にパース済みの結果を返します。

```ts
response.output_parsed
```

そのため、OpenAI 版の流れは非常に直接的です。

```text
Zod スキーマ → OpenAI の構造化出力フォーマット → パース済み TypeScript オブジェクト
```

## 📄 ファイル 2: `src/gemini-lesson-card.ts`

```ts
import { GoogleGenAI } from "@google/genai";
import * as z from "zod";

const ai = new GoogleGenAI({});

const LessonCardSchema = z.object({
  title: z.string(),
  summary: z.string(),
  keyTerms: z.array(
    z.object({
      term: z.string(),
      definition: z.string(),
    })
  ),
  quizQuestion: z.object({
    question: z.string(),
    answer: z.string(),
  }),
  nextStep: z.string(),
});

type LessonCard = z.infer<typeof LessonCardSchema>;

const sourceText = `
光合成は、植物が自分の食べ物を作るために使う仕組みです。
植物は空気中の二酸化炭素と、土の中の水を取り込みます。
太陽の光のエネルギーを使って、それらを糖と酸素に変えます。
糖は植物が成長する助けになり、酸素は空気中に放出されます。
`;

async function main() {
  try {
    const prompt = `
あなたは初心者にもわかりやすく教える理科の先生です。
小学生向けの短いレッスンカードを作成してください。

元テキスト:
${sourceText}
`;

    const response = await ai.models.generateContent({
      model: "gemini-3.5-flash",
      contents: prompt,
      config: {
        responseMimeType: "application/json",
        responseJsonSchema: z.toJSONSchema(LessonCardSchema),
      },
    });

    if (!response.text) {
      throw new Error("Gemini はテキストを返しませんでした。");
    }

    const lessonCard: LessonCard = LessonCardSchema.parse(
      JSON.parse(response.text)
    );

    console.log("Gemini のレッスンカード:");
    console.dir(lessonCard, { depth: null });
  } catch (error) {
    console.error("Gemini 実行エラー:", error);
  }
}

main();
```

## Gemini の構造化出力アプローチ

Gemini のスクリプトは少し異なります。

Gemini は JSON Schema 形式のレスポンスフォーマットを受け取るため、Zod スキーマを JSON Schema に変換します。

```ts
z.toJSONSchema(LessonCardSchema)
```

次に、Gemini に JSON 出力が欲しいことを伝えます。

```ts
responseMimeType: "application/json"
```

Gemini からレスポンスが返ってきたあとも、プログラム側から見るとレスポンスはまだテキストです。

```ts
response.text
```

そのため、2 つの明示的なステップを行います。

```ts
JSON.parse(response.text)
```

これにより、JSON 文字列を JavaScript オブジェクトに変換します。

その後、Zod でオブジェクトを検証します。

```ts
LessonCardSchema.parse(...)
```

したがって、Gemini 版の流れは次のようになります。

```text
Zod スキーマ → Gemini 用の JSON Schema → JSON テキストレスポンス → JSON.parse → Zod バリデーション
```

注意：この例では、シリーズの最初の記事で使ったスタイルに合わせるため、Gemini の `generateContent` API を使用しています。Google の現在のドキュメントでは、最新の Gemini 機能やモデルにアクセスするために、より新しい Interactions API も推奨されています。このスクリプトが将来動作しなくなった場合は、まず最新の Gemini 構造化出力ドキュメントを確認してください。

## 📄 ファイル 3: `src/claude-lesson-card.ts`

```ts
import Anthropic from "@anthropic-ai/sdk";
import { zodOutputFormat } from "@anthropic-ai/sdk/helpers/zod";
import { z } from "zod/v4";

const client = new Anthropic();

const LessonCardSchema = z.object({
  title: z.string(),
  summary: z.string(),
  keyTerms: z.array(
    z.object({
      term: z.string(),
      definition: z.string(),
    })
  ),
  quizQuestion: z.object({
    question: z.string(),
    answer: z.string(),
  }),
  nextStep: z.string(),
});

type LessonCard = z.infer<typeof LessonCardSchema>;

const sourceText = `
光合成は、植物が自分の食べ物を作るために使う仕組みです。
植物は空気中の二酸化炭素と、土の中の水を取り込みます。
太陽の光のエネルギーを使って、それらを糖と酸素に変えます。
糖は植物が成長する助けになり、酸素は空気中に放出されます。
`;

async function main() {
  try {
    const message = await client.messages.parse({
      model: "claude-sonnet-4-6",
      max_tokens: 1000,
      system:
        "あなたは初心者にもわかりやすく教える理科の先生です。小学生向けの短いレッスンカードを作成してください。",
      messages: [
        {
          role: "user",
          content: sourceText,
        },
      ],
      output_config: {
        format: zodOutputFormat(LessonCardSchema),
      },
    });

    const lessonCard: LessonCard | null | undefined = message.parsed_output;

    if (!lessonCard) {
      throw new Error("Claude はパース済みのレッスンカードを返しませんでした。");
    }

    console.log("Claude のレッスンカード:");
    console.dir(lessonCard, { depth: null });
  } catch (error) {
    console.error("Claude 実行エラー:", error);
  }
}

main();

```

## Claude の構造化出力アプローチ

Claude のスクリプトでは、Anthropic の Messages API と構造化出力サポートを使います。

OpenAI のスクリプトと同じように、Claude 版でも手動で `JSON.parse(...)` は呼び出しません。

その代わりに、Zod スキーマを Anthropic のヘルパーに渡します。

```ts
zodOutputFormat(LessonCardSchema)
```

そのフォーマットは、次のように送信されます。

```ts
output_config: {
  format: zodOutputFormat(LessonCardSchema),
}
```

Claude が有効な構造化レスポンスを返すと、SDK は次の場所にパース済みのオブジェクトを返します。

```ts
message.parsed_output
```

したがって、Claude 版の流れは次のようになります。

```text
Zod スキーマ → Claude の出力フォーマット → パース済み TypeScript オブジェクト
```

Claude では、レスポンスの最大サイズを制御する `max_tokens` の値を明示的に指定する必要もあります。

## package.json を開き、これらのスクリプトを実行する行を追加する

```json
{
  "scripts": {
    "start:openai-lesson-card": "tsx src/openai-lesson-card.ts",
    "start:gemini-lesson-card": "tsx src/gemini-lesson-card.ts",
    "start:claude-lesson-card": "tsx src/claude-lesson-card.ts"
  }
}
```

**注意：前の記事で作成したスクリプト項目がすでにあっても問題ありません。**

## 🚀 ステップ 7：スクリプトを実行する

API キーの環境変数が読み込まれており、`package.json` にスクリプトが設定されている状態で、各プロバイダーのテストを個別に実行します。

```bash
# OpenAI Responses API のスクリプトを実行
npm run start:openai-lesson-card

# Google Gemini Gen AI のスクリプトを実行
npm run start:gemini-lesson-card

# Claude Anthropic Messages のスクリプトを実行
npm run start:claude-lesson-card
```

**注意：私の環境では、Gemini のスクリプトは OpenAI や Claude のスクリプトと比べてかなり実行に時間がかかりました。結果は環境によって異なる場合があります。**

**注意：** AI API と SDK はすばやく変化します。この記事の公開後に、モデル名、ヘルパーの import、構造化出力の構文が変わる可能性があります。

スクリプトが失敗した場合は、まず次の点を確認してください。

* プロバイダーのドキュメントにある現在のモデル名
* インストールされている SDK のバージョン
* プロバイダーの現在の構造化出力の例
* ターミナルに表示された正確なエラーメッセージ

スクリプトとエラーを AI アシスタントに貼り付けてデバッグの助けを得ることもできますが、必ず現在の公式ドキュメントと照らし合わせて確認してください。

以下は私が受け取ったレスポンスです。あなたの結果は異なる可能性があります。

**OpenAI のレッスンカード:**

```ts
{
  title: '光合成：植物が食べ物を作るしくみ',
  summary: '植物は、葉で太陽の光を受け、空気中の二酸化炭素と土から吸い上げた水を使って「糖」を作ります。このとき、いっしょに酸素もできて、空気中に出されます。糖は植物が大きく育つためのエネルギーになります。',
  keyTerms: [
    { term: '光合成', definition: '植物が光の力を使って、自分の食べ物である糖を作るしくみ。' },
    { term: '二酸化炭素', definition: '空気中にある気体で、植物が光合成に使うもの。' },
    { term: '糖', definition: '植物の成長に使われる食べ物のもと。' },
    { term: '酸素', definition: '光合成でできて、空気中に出される気体。' }
  ],
  quizQuestion: { question: '植物が光合成で作る、成長の助けになるものは何でしょう？', answer: '糖です。' },
  nextStep: '身近な葉っぱを観察して、植物がどこで光を受けているか考えてみましょう。'
}
```

**Gemini のレッスンカード:**

```ts
{
  title: '植物のパワー！光合成（こうごうせい）のひみつ',
  summary: '植物は、太陽の光、水、そして二酸化炭素（にさんかたんそ）を使って、自分で栄養（糖）を作っています。このときに、私たちが呼吸に使う酸素（さんそ）も作って空気中に出してくれています。',
  keyTerms: [
    {
      term: '光合成（こうごうせい）',
      definition: '植物が太陽の光のパワーを使って、自分で栄養（糖）を作る仕組みのこと。'
    },
    {
      term: '二酸化炭素（にさんかたんそ）',
      definition: '植物が光合成をするときに、空気中から取り込む気体のこと。'
    },
    {
      term: '酸素（さんそ）',
      definition: '光合成のときに作られて、空気中に出される気体。人間や動物が息をするために必要です。'
    }
  ],
  quizQuestion: {
    question: '植物が光合成をするときに、エネルギーとして使うものは何でしょう？',
    answer: '太陽の光（たいようのひかり）'
  },
  nextStep: 'おうちの近くや公園で、太陽の光を浴びている植物を観察してみようすい葉っぱを観察してみましょう。よく観察（かんさつ）してみましょう。みようしてみよう'
}
```

**Claude のレッスンカード:**

```ts
{
  title: '光合成ってなに？',
  summary: '植物は太陽の光を使って、二酸化炭素と水から自分の食べ物（糖）を作ります。このしくみを「光合成」といいます。そのときに酸素もできて、空気中に出されます。',
  keyTerms: [
    { term: '光合成', definition: '植物が太陽の光を使って食べ物を作るしくみ' },
    { term: '二酸化炭素', definition: '植物が空気中から取り込むガス' },
    { term: '糖（とう）', definition: '植物が成長するためのエネルギーになる甘い物質' },
    { term: '酸素', definition: '光合成のときにできて、空気中に出されるガス。わたしたちが呼吸するのに必要' }
  ],
  quizQuestion: {
    question: '植物が光合成をするために必要なものを3つ答えましょう！',
    answer: '太陽の光・二酸化炭素・水　の3つです！'
  },
  nextStep: '次は「植物の根・茎・葉のはたらき」を学んで、水や栄養がどのように運ばれるか調べてみよう！'
}
```

各 API は異なるレスポンスを返しますが、3 つのレスポンスすべてが、要求した構造化出力の形に従うはずです。

これがこの記事の主な学びです。

構造化出力は、レスポンスの**形**を制御する助けになります。

ただし、生成されたすべての詳細が真実であること、完全であること、または元テキストに完全に基づいていることを自動的に保証するわけではありません。

たとえば、モデルが追加の用語を含めたり、異なるクイズ問題を選んだり、要約の表現を変えたりする場合があります。

そのため、構造化出力はアプリケーションデータにとって便利ですが、通常のアプリケーションとしての判断、テスト、検証も引き続き必要です。

## JSON モードと構造化出力

次の 2 つの考え方を分けて理解することも重要です。

```text
JSON モードとは、有効な JSON を返すこと。
構造化出力とは、特定のスキーマに従う JSON を返すこと。
```

有効な JSON であることだけでは十分ではありません。

たとえば、これは有効な JSON です。

```json
{
  "message": "植物は太陽の光を使って食べ物を作ります。"
}
```

しかし、これはプログラムが期待している形ではありません。

私たちのプログラムが期待しているフィールドは、次のようなものです。

```text
title
summary
keyTerms
quizQuestion
nextStep
```

構造化出力のほうが強力です。アプリケーションが必要とする特定のオブジェクト形状にレスポンスを合わせようとするからです。

## 3 つの API の違い

主な違いは目標ではありません。

3 つの API すべてで目標は同じです。

```text
スキーマに一致するレッスンカードオブジェクトを作成する。
```

違いは、それぞれの API がその目標にどのように到達するかです。

### OpenAI

OpenAI は、非常に統合された構造化出力の流れを提供します。

Zod スキーマを定義し、それを OpenAI のヘルパーに渡し、`response.output_parsed` からパース済みオブジェクトを読み取ります。

### Gemini

Gemini では、JSON との境界がより見えやすくなります。

Gemini に JSON を要求し、生成された JSON をテキストとして読み取り、手動でパースし、その後 Zod で検証します。

コードは少し増えますが、パースとバリデーションのステップが見えやすいため、学習には役立ちます。

### Claude

この例では、Claude のアプローチは OpenAI に近いです。

`output_config` を通してスキーマベースの出力フォーマットを渡し、SDK は `message.parsed_output` 上にパース済みオブジェクトを返します。

Claude では `max_tokens` も必要です。そのため、OpenAI と Gemini の例と比べると、リクエストに追加の必須設定が 1 つあります。

## 最後に

このワークスペースは、シンプルな API 学習と実験を目的としています。

これらのスクリプトを実行する前に、次の点を確認してください。

- API キーが設定されていることを確認する。
- ハードな課金上限を設定する。
- テスト用のループは小さく保つ。
- シークレットをコードに直接書かない。
- サンドボックス作業では、低コストのモデルを使う。
