# モダンな TypeScript AI ワークスペースのセットアップ

シンプルな 1 ファイルの quickstart script を使って、**OpenAI**、**Google Gemini**、**Anthropic Claude** の API をテストするための、クリーンでモダンな TypeScript workspace です。

このガイドでは、次の内容を扱います。

- Node.js + TypeScript プロジェクト の作成
- モダンな ESM モジュール の設定
- 各 provider SDK のインストール
- APIキーを環境変数として設定する
- OpenAI、Gemini、Claude 用の個別 クイックスタート用スクリプト の実行
- 各 API が リクエストとレスポンス をどのように構成しているかの理解

---

## 💰 アカウント要件と費用に関する注意事項

この ワークスペース で コード を実行する前に、各 AIプロバイダーの開発者アカウントを作成し、請求設定してください。

プラットフォーム URL は時間とともに変わる可能性があるため、正しい オンボーディングポータル を見つける最も簡単な方法は、次の ダッシュボード 名で検索することです。

| プロバイダー | 検索する語句 | 目的 |
|---|---|---|
| **OpenAI** | `OpenAI API Platform Dashboard` | developer account を登録し、事前購入型の API credits を確保します。 |
| **Google Gemini** | `Google AI Studio` | Gemini developer workspace を初期化します。Gemini は prototyping 用の free tier を提供している場合がありますが、Google Cloud billing に接続しない限り rate limits が適用されます。 |
| **Anthropic Claude** | `Anthropic Console` | developer organization を作成し、事前購入型の API credits を購入します。 |

---

## ⚠️ 費用管理のベストプラクティス

プログラムから送信されるリクエストはすべて、リモートのAIサービスにトークンを送信します。これらのリクエストは、APIアカウントに請求される実際の費用を発生させる可能性があります。

自動処理を実行する前に、次の対策で自分を守ってください。

### ハード使用上限を設定する

各 プロバイダーの請求ダッシュボード にログインし、**1000円** などの厳しめの日次または月次 予算上限 を設定してください。

これにより、暴走したスクリプトがクレジットを使い切ったり、予期しない請求を発生させたりするのを防ぎやすくなります。

### サンドボックスのループでは本番用モデルを避ける

簡単なテスト、デバッグ、CIループでは、可能な限り安価で高速なモデルを使用してください。

例:

```text
gemini-3.5-flash
```

大規模なエンタープライズ向けモデルは、対象を絞った本番用途のために取っておきましょう。

### ループを保護する

これらのスクリプトを使ってカスタム自動処理を書く場合は、必ず厳密なループ上限を設定してください。

例:

```ts
for (let i = 0; i < 5; i++) {
  // 安全な上限付きループ
}
```

これにより、非同期の無限ループがAPIエンドポイントを継続的に呼び出し続けるのを防ぎやすくなります。

---

## ⚖️ 法的免責事項

このガイドのソースコードと実装手順は、教育、情報提供、試作を目的として提供されています。

著者はこれらの資料を 「現状のまま」 提供しており、明示または黙示を問わず、いかなる保証も行いません。

このワークスペースのコードを実行することにより、開発者アカウント、APIキーの安全管理、金銭的責任、費用、およびAPIキーやインフラを通じて発生するデータ使用料について、あなた自身が単独で責任を負うことを認めるものとします。

著者は、金銭的損失、予期しないクレジット使用、アカウント停止、請求額の急増を含む、直接的、間接的、付随的、特別、または結果的な損害について責任を負いません。

責任を持って使用し、自動処理を実行する前にハード請求上限を設定してください。

---

## 概要

OpenAI、Gemini、Claude用のコードを書く前に、クリーンなTypeScript開発環境をセットアップします。

TypeScriptは、コンパイルまたは変換の手順なしでは、コンピューター上で直接実行できません。このクイックスタートをシンプルに保つため、このワークスペースでは tsx を使用します。これにより、JavaScriptファイルを手動でビルドしなくても、.tsスクリプトをすぐに実行できます。

---

## 🛠️ 第1ステップ：Node.jsワークスペースを初期化する

ターミナルを開き、新しいプロジェクトディレクトリを作成して、その中に移動します。

```bash
mkdir ai-quickstarts && cd ai-quickstarts
npm init -y
```

これにより、プロジェクトルートにデフォルトの `package.json` ファイルが作成されます。

---

## ⚙️ 第2ステップ：モダンなESMモジュールを設定する

デフォルトでは、Node.jsは古い **CommonJS** モジュールシステムを使用します。

モダンなAI SDKやTypeScriptコードでは通常、`import` と `export` 構文を使用するため、プロジェクトを **ECMAScript Modules**、つまり **ESM** を使うように設定します。

生成された `package.json` ファイルを開き、ファイル全体を次の内容に置き換えます。

```json
{
  "name": "ai-quickstarts",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start:openai": "tsx src/openai-test.ts",
    "start:gemini": "tsx src/gemini-test.ts",
    "start:claude": "tsx src/claude-test.ts"
  }
}
```

---

## 📦 第3ステップ：SDKとTypeScriptユーティリティをインストールする

コアAI SDKをインストールします。

```bash
npm install openai@latest @google/genai@latest @anthropic-ai/sdk@latest
```

開発用ツールをインストールします。

```bash
npm install -D typescript@latest tsx@latest @types/node@latest
```

### tsxとは？

**tsx** は **TypeScript Execute** の略です。

`.ts` ファイルを手動で別のJavaScript出力フォルダにコンパイルする代わりに、`tsx`は TypeScriptファイルを直接実行します。TypeScript専用の構文はメモリ上で取り除かれます。

これにより、API実験用の作業フローを高速でシンプルに保つことができます。

---

## 📄 第4ステップ：`tsconfig.json`を作成する

TypeScript設定ファイルを生成します。

```bash
npx tsc --init
```

これにより、プロジェクトルートに `tsconfig.json` ファイルが作成されます。

デフォルトの内容全体を、次のクリーンな基本設定に置き換えます。

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

---

## 📁 第5ステップ：プロジェクト構成を作成する

実行可能なTypeScriptファイル用に、専用の `src` フォルダを作成します。

プロジェクトは次のような構成にします。

```text
ai-quickstarts/
├── src/
│   ├── openai-test.ts
│   ├── gemini-test.ts
│   └── claude-test.ts
├── package.json
└── tsconfig.json
```

`src` ディレクトリにこれら3つのファイルを作成し、以下の各プロバイダー用クイックスタートコードをそれぞれのファイルに貼り付けます。

- `src/openai-test.ts`
- `src/gemini-test.ts`
- `src/claude-test.ts`

---

## 📄 ファイル1： `src/openai-test.ts`

この例では、OpenAI の **Responses API** を使用します。

`output_text` ヘルパープロパティを使って、モデルの出力を取り出します。

```ts
import OpenAI from "openai";

// process.env.OPENAI_API_KEY を自動的に読み取ります
const client = new OpenAI();

async function main() {
  try {
    const response = await client.responses.create({
      model: "gpt-5.5",
      input: "ユニコーンについての一文のベッドタイムストーリーを書いてください。"
    });

    // modernなResponses APIは、output_textを使って文字列トークンをまとめます
    console.log("OpenAIのレスポンス:");
    console.log(response.output_text);
  } catch (error) {
    console.error("OpenAI実行エラー:", error);
  }
}

main();
```

---

## 📄 ファイル2： `src/gemini-test.ts`

この例では、モダンなGoogle Gen AI SDKである `@google/genai` を使用します。

生成されたテキストには `response.text` からアクセスします。

```ts
import { GoogleGenAI } from "@google/genai";

// process.env.GEMINI_API_KEY を自動的に読み取ります
const ai = new GoogleGenAI({});

async function main() {
  try {
    const response = await ai.models.generateContent({
      model: "gemini-3.5-flash",
      contents: "ユニコーンについての一文のベッドタイムストーリーを書いてください。"
    });

    // 設計されたプロトタイプゲッターが、内部のJSON配列を動的に解析します
    console.log("Geminiのレスポンス:");
    console.log(response.text);
  } catch (error) {
    console.error("Gemini実行エラー:", error);
  }
}

main();
```

---

## 📄 ファイル3： `src/claude-test.ts`

この例では、Anthropicの構造化された **Messages API** を使用します。

Claudeのリクエストには、明示的な `max_tokens` の値が必要です。レスポンスの内容はコンテンツブロックの配列として返されるため、このスクリプトでは最初のブロックを確認してから、そのテキストを表示します。

```ts
import Anthropic from "@anthropic-ai/sdk";

// process.env.ANTHROPIC_API_KEY を自動的に読み取ります
const client = new Anthropic();

async function main() {
  try {
    const message = await client.messages.create({
      model: "claude-opus-4-8",
      max_tokens: 1000, // Claudeでは、この明示的な安全用パラメーターが必要です
      messages: [
        {
          role: "user",
          content: "ユニコーンについての一文のベッドタイムストーリーを書いてください。"
        }
      ]
    });

    // Claudeは、テキスト、レイアウトブロック、ツール実行を交互に返す場合があります。
    // テキストを取得するために、0番目の要素から始まる配列ブロックの中を確認します。
    console.log("Claudeのレスポンス:");
    if (message.content[0].type === "text") {
      console.log(message.content[0].text);
    }
  } catch (error) {
    console.error("Claude実行エラー::", error);
  }
}

main();
```

---

## 🔑 第6ステップ：環境変数を設定する

3つのSDKはすべて、OS環境からAPI認証情報を読み取ることができます。

つまり、シークレットキーをコードに直接ハードコードする必要はありません。

**実際には、APIキーやその他の機密情報を、コードファイルやgitなどのソース管理にコミットされる可能性があるファイルにハードコードしてはいけません。**

これら3つのAPIそれぞれについてAPIキーを作成します。APIキーの作成手順については、各APIの開発者向けドキュメントを参照してください。

APIキーを作成するときは、コンピューター上のどこかにコピーして保存しておく必要があります。作成後は再び表示できないためです。

スクリプトを実行する前に、現在開いているターミナルセッションで次のコマンドを実行します。

```bash
export OPENAI_API_KEY="sk-proj-..."
export GEMINI_API_KEY="AIzaSy..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## ターミナルで直接 `export KEY="value"` を実行する方法は一時的です。

ターミナルタブを閉じたり、コンピューターを再起動したりすると、これらの変数は消えます。

毎回APIキーを再度exportしなくてよいように、ターミナルのプロファイル設定に保存します。

---

## Mac/Linuxユーザー向け

ほとんどのモダンなターミナルは **Bash** または **Zsh** を使用します。

適切なプロファイルファイルを開きます。

```bash
# ターミナルがBashを使用している場合:
nano ~/.bash_profile

# ターミナルがmodern macOSのデフォルトであるZshを使用している場合:
nano ~/.zshrc
```

ファイル の一番下までスクロールし、次を貼り付けます。

```bash
# AIプロバイダーのAPIキー
export OPENAI_API_KEY="sk-proj-..."
export GEMINI_API_KEY="AIzaSy..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

Nano を保存して終了します。

- Ctrl + O を押す
- Enter を押す
- Ctrl + X を押す

変更をすぐに適用します。

```bash
source ~/.bash_profile  # or source ~/.zshrc
```

---

## 💻 その他の環境設定方法

シェルやOSによって、環境変数の保存方法は異なります。

重要なのは、次の正確な変数名がグローバルに利用可能であることです。

```text
OPENAI_API_KEY
GEMINI_API_KEY
ANTHROPIC_API_KEY
```

| 環境 | 設定方法の例 |
|---|---|---|---|
| **Windows PowerShell** | `$PROFILE` を使って、永続的なプロファイルスクリプトに変数を追加します。 |
| **Windows Command Prompt** | `setx` を使って、変数をシステムレジストリに永続保存します。 |
| **VS Code / WebStorm** | これらの変数名を、IDEのRun/Debug Configurationテンプレートに設定します。 |

### Windows PowerShellの例

```powershell
$env:OPENAI_API_KEY="sk-proj-..."
$env:GEMINI_API_KEY="AIzaSy..."
$env:ANTHROPIC_API_KEY="sk-ant-..."
```

### Windows Command Promptの例

```cmd
setx OPENAI_API_KEY "sk-proj-..."
setx GEMINI_API_KEY "AIzaSy..."
setx ANTHROPIC_API_KEY "sk-ant-..."
```

> **注:** これらは例にすぎません。OS、シェル、エディター、作業フローによっては、別の設定方法が必要になる場合があります。

---

## 🚀 第7ステップ：スクリプトを実行する

変数が読み込まれ、スクリプトが `package.json` にマッピングされたら、各プロバイダーのテストを個別に実行できます。

```bash
# OpenAI Responses API スクリプトを実行する
npm run start:openai

# Google Gemini Gen AI スクリプトを実行する
npm run start:gemini

# Claude Messages スクリプトを実行する
npm run start:claude
```

注：これら3つのスクリプトは同じプロンプトを使用しますが、APIごとに異なるLLM（Large Language Model、大規模言語モデル）を使うため、生成される結果は異なります。

** ユニコーンが登場する一文の寝る前のお話を書いてください。**

以下は私が受け取ったレスポンス例です。あなたの結果はおそらく異なります。

**OpenAI Response:**

月明かりの森で、銀色のユニコーンが眠れない子どもの枕元にそっと星くずを置くと、やさしい夢がふわりと広がりました。

**Gemini Response:**

星屑のたてがみを持つ優しいユニコーンが、あなたの枕元でそっと角を光らせて、今夜も温かく心地よい夢の国へと連れて行ってくれます。

**Claude Response:**

星空の下、小さなユニコーンは虹色のたてがみを揺らしながら、眠れない子どもたちの夢の中へそっと忍び込み、優しい子守唄を歌ってくれました。

おやすみなさい。✨🦄

---

## 🧠 アーキテクチャ上の基本要素：各APIの考え方

3つのAPIはすべて、シンプルな寝る前のお話を生成できます。ただし、内部で使うデータ構造はそれぞれ異なります。

各プラットフォームは、異なる中心概念をもとに設計されています。

| プロバイダー | 中核となるAPIシステム | 中核となる基本要素 | 考え方 |
|---|---|---|---|
| OpenAI | Responses API `/v1/responses` | **Item** | 状態を持たないイベントノードがチェーンのようにつながっていく、進化する台帳。 |
| Gemini | Models API `/v1/models` | **Content** | テキスト、画像、音声などのマルチモーダルな要素を、1つのレイアウトとして処理する状態を持たない配列。 |
| Claude | Messages API `/v1/messages` | **Message** | userとassistantの会話ターンが交互に並ぶ、厳密で状態を持つ対話。 |

---

## 1. OpenAI：Responses API

OpenAIは、以前から広く模倣されてきた **Chat Completions** API を使用していました。

新しい **Responses API** では、基本的なテキストチャットの配列から離れ、エージェント的なシステムのタイムラインに近いモデルへと移行しています。

### 仕組み

OpenAIは、やり取りを **Items** としてまとめます。

item は次のようなものを表すことができます。

- メッセージ
- 生のシステムコマンド
- 関数呼び出し
- 関数呼び出しの結果

### 重要な理由

この構造により、ツールの使用とサーバー側での状態管理が扱いやすくなります。

毎回、過去のメッセージ配列全体を送信する代わりに、`previous_response_id` ポインターを渡すことができます。OpenAIは、そのItemチェーンの履歴を再構築できます。

---

## 2. Gemini: Google Gen AI SDK

Geminiは、ネイティブなマルチモーダル入力を前提として設計されています。

つまり、テキスト、画像、音声、コード、動画形式のデータを、統一された構造を通して処理できます。

### 仕組み

主な入力パラメーターは `contents` です。

`contents` は **Part** オブジェクトの配列にマッピングされます。

1つのユーザープロンプトには、次のようなものを含めることができます。

- テキスト部分
- 画像バッファ部分
- 音声ファイル部分
- その他の対応しているマルチモーダル入力

### 重要な理由

これにより、Geminiはマルチモーダルプロンプトに対して、統一された処理モデルを持つことができます。

標準的なテキスト生成の場合、SDKは次のような便利なヘルパーを提供します。

```ts
response.text
```

これにより、深くネストされたレスポンスブロックを手動で解析しなくても、最終的な出力にアクセスできます。

---

## 3. Claude: Anthropic Messages API

Anthropic Claude は、Messages API を通じて、高度に構造化された会話重視のモデルを使用します。

### 仕組み

各リクエストは、`user` と `assistant` のメッセージブロックが交互に並ぶ配列を中心に作成されます。

Anthropicは、リクエストの境界も明確に定めています。

例：

`max_tokens` を明示的に設定する必要があります。
会話構造は意図的に厳密です。
レスポンスには、異なる種類のコンテンツブロックが含まれる場合があります。

### 重要な理由

Claudeの厳密な構造は、次のような専門的な機能を支えています。

**Extended Thinking**：モデルが最終回答の前に、別の `thinking` ブロックを返すことができます。
**Prompt Caching**：特定のコンテキストブロックをキャッシュして、遅延と実行コストを減らすことができます。

---

## 最後に

このワークスペースは、シンプルなAPI学習と実験を目的としています。

これらのスクリプトを実行する前に、次を確認してください。

- APIキーが設定されていることを確認する。
- ハード請求上限を設定する。
- テスト用ループを小さく保つ。
- シークレットをハードコードしない。
- サンドボックス作業では低コストのモデルを使用する。