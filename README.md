# Strategic Study Tracker (クラウド同期対応・スマート学習プラットフォーム)

`Strategic Study Tracker` は、学習効率を極限まで高めるために設計されたクラウド同期対応のスマート学習管理プラットフォームです。

ユーザーは学びたい「分野（Fields）」を設定し、その中に「教科書（PDF）」「動画（Movie）」「参考サイト（Web）」およびそれらをカリキュラムとしてまとめた「講座（Course）」を登録して進捗を記録できます。
さらに、iPad等のデバイスを用いた滑らかな手書きアノテーション機能、Google Gemini 3.1 Pro Preview APIを活用したAIアシスタント機能（高速翻訳・Q&A）、そしてAIの内部思考プロセスを視覚的に再現する「Reasoning Visualization（思考プロセス視覚化）」など、次世代の学習体験をサポートする数々の機能を搭載しています。

---

## 🚀 主な機能 (Key Features)

### 1. 多次元な学習リソースの管理 (Hierarchical Study Management)
*   **分野（Fields）分類**: 数学、情報科学、デザインなど、学びたいカテゴリごとに教材をスマートに隔離・整理。
*   **4つの教材タイプ**:
    *   📖 **PDF（TEXTBOOK）**: デジタル教科書や論文を読み込み、閲覧中の現在ページとスクロール比率（%）を完全にクラウド同期。
    *   🎥 **動画（MOVIE）**: 講義動画やYouTube、またはアップロードされた動画を管理。秒単位での再生・学習進捗の追跡に対応。
    *   🌐 **Web（WEBSITE）**: 役立つ技術ブログ、公式ドキュメント、参考サイトのブックマーク管理。
    *   🗂️ **講座（COURSE）**: 複数のPDF、動画、Web教材を内部に包含する「シラバス/カリキュラム」コンテナ。教材をコースへ自由に追加・離脱（Detach）させることが可能。
*   **Bin-pack レイアウトアルゴリズム**: カテゴリごとの教材登録状況に応じて、カード型グリッド（最大4列）をデッドスペースなく美しく最適配置。

### 2. 高機能PDFビューア & 手書きアノテーション (Advanced PDF Canvas)
*   **滑らかな描画（Canvas API）**: 独自の2層描画キャッシュ構造と `quadraticCurveTo`（2次ベジェ曲線）処理を用いた、超高反応なペンおよび消しゴム機能。
*   **マルチデバイス・レスポンシブ**: iPad + Apple Pencil や Android タブレット、PCマウスによる入力を完全にサポート。
*   **自動座標正規化**: 画面の解像度やズーム倍率、アスペクト比に依存しないよう、描画位置を `0~1` の相対座標に正規化してクラウドに保存。異なるデバイス間で閲覧しても正確な位置に同期・再描画。

### 3. Gemini 3.1 Pro 搭載 AI学習アシスタント (AI Copilot)
*   **ユーザー専用の API キー**: ユーザー自身が Google AI Studio から取得した Gemini API キーを設定画面から暗号化して登録。APIは直接クライアント/サーバーアクションから呼び出され、プラットフォーム側にキーを保存しない安全設計。
*   **テキスト自動翻訳 & カスタム質問 (Translate & Ask)**: 教科書内の任意のテキストを選択、または手書きアノテーションで囲った領域に対して「瞬時に翻訳」や「AIへの専門的な質問」を呼び出し可能。
*   **AI 会話履歴の自動保存**: 教材ごとに過去のQ&Aスレッドが紐づいてクラウドに保存され、後からいつでも復習可能。

### 4. 隙間時間を活用する理論フラッシュカード (Reasoning Visualization)
*   **Reasoning Visualization**: AIの応答待ち時間（ローディング時間）などの僅かな隙間時間を無駄にせず勉強できるよう、線形代数（Kernel, Rank-Nullity 定理等）やディープラーニング、確率統計といった高度な学術理論とAIでの応用シーンを、美しいサイバーパンク調のアニメーションと共に1問1答形式で表示する学習サポートユーティリティ。

### 5. 高度なセキュリティ & クラウド同期 (Supabase Cloud Architecture)
*   **Supabase Auth & Row Level Security (RLS)**: すべてのデータ（分野、教材、手書きログ、設定情報）はPostgreSQLのRLSポリシーにより厳格に防護され、他のユーザーから干渉を受けることは一切ありません。
*   **セキュアな Storage 隔離**: PDFやカバー画像を保存する `materials` バケットには、`auth.uid()`（ユーザーUUID）をベースにしたフォルダ分離セキュリティポリシーを適用。他人のオブジェクトへの不正なアクセスや読み書きをサーバーレベルで阻止。

---

## 🛠️ 技術スタック (Tech Stack)

| カテゴリ | 採用技術 |
| :--- | :--- |
| **Frontend** | React 18, Next.js 14 (App Router), TypeScript, Tailwind CSS |
| **Icons & UI Components** | Lucide React, Radix UI |
| **PDF Processing** | `react-pdf`, `pdfjs-dist` |
| **Database / Auth / Storage** | Supabase, PostgreSQL |
| **AI Integration** | Google Generative AI API (`gemini-3.1-pro-preview` model) |

---

## 📊 データベース設計 (Schema Definition)

本システムは PostgreSQL (Supabase) 上に構築されています。主なテーブル構成は以下の通りです。

```sql
-- 1. Fields (分野テーブル)
create table public.fields (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references auth.users(id) on delete cascade not null,
  name text not null,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null
);

-- 2. Materials (教材テーブル)
create table public.materials (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references auth.users(id) on delete cascade not null,
  field_id uuid references public.fields(id) on delete cascade not null,
  title text not null,
  type text not null check (type in ('TEXTBOOK', 'MOVIE', 'WEBSITE', 'COURSE')),
  parent_id uuid references public.materials(id) on delete set null, -- コース包含用
  cover_url text,      -- カバー画像のStorageリンク
  pdf_path text,       -- PDFのStorageリンク
  video_path text,     -- 動画URL / 外部リンク
  total_pages integer default 0,
  current_page integer default 1,
  scroll_ratio double precision default 0,
  progress double precision default 0,
  created_at timestamp with time zone default timezone('utc'::text, now()) not null
);

-- 3. Annotations (手書き/注釈データテーブル)
create table public.annotations (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references auth.users(id) on delete cascade not null,
  material_id uuid references public.materials(id) on delete cascade not null,
  page_number integer not null,
  type text not null check (type in ('highlight', 'note', 'stroke')),
  data jsonb not null, -- ストロークの座標(0-1正規化)配列を格納
  created_at timestamp with time zone default timezone('utc'::text, now()) not null,
  updated_at timestamp with time zone default timezone('utc'::text, now()) not null
);

-- 4. User Settings (個人設定・APIキー)
create table public.user_settings (
  user_id uuid references auth.users(id) on delete cascade primary key,
  gemini_api_key text,
  updated_at timestamp with time zone default timezone('utc'::text, now()) not null
);
```

---

## ⚙️ セットアップ手順 (Setup & Installation)

### 1. リポジトリのクローンと依存関係のインストール
```bash
git clone <repository-url>
cd study-materials
npm install
```

### 2. 環境変数の設定
プロジェクトのルートディレクトリに `.env.local` を作成し、お手持ちの Supabase の接続情報を設定します。

```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

### 3. Supabase データベースと RLS の構築
Supabase Dashboard の **SQL Editor** を開き、`/supabase/schema.sql` に定義されている SQL スクリプトをコピー＆ペーストして実行し、テーブル、RLSポリシー、トリガー、ヘルパー関数をセットアップします。

### 4. Storage (ストレージ) の設定
1. Supabase Dashboard の **Storage** セクションに移動し、`materials` という名前のバケットを新規作成します（Private バケットで問題ありません）。
2. SQL Editor を開き、`/supabase/schema.sql` の「Supabase Storage Buckets Setup」に記載されているセキュリティポリシーを実行し、ユーザーごとのフォルダ隔離セキュリティを有効化します。

### 5. ローカル開発サーバーの起動
```bash
npm run dev
```
ブラウザで [http://localhost:3000](http://localhost:3000) を開き、サインアップ（新規登録）を行うことで、クラウド同期された高度な学習追跡環境をすぐに体験できます。

---

## 💡 Gemini APIキーの登録方法
1. [Google AI Studio](https://aistudio.google.com/app/apikey) にログインし、APIキー（`AIza...` で始まる文字列）を無料で作成・取得します。
2. Strategic Study Tracker にログイン後、画面右上ナビバーの **「⚙️ 設定」** ボタンをクリックします。
3. 取得したAPIキーを入力し「保存」をクリックします。
4. これにより、PDFビューア画面での選択箇所の翻訳機能、およびAIアシスタント質問機能（Gemini 3.1 Pro モデル）が完全に稼働します。

---

## 📄 ライセンス (License)

本プロジェクトは **MIT ライセンス** の下で提供されています。詳細についてはリポジトリ内のライセンス規定をご参照ください。
