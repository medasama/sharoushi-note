# 社労士 進捗帳（sharoushi-note）セットアップ

置き場所：`久山事務所\sharoushi-note`（GitHub：medasama/sharoushi-note、Vercel連携済み）
構成：`index.html` 1ファイル（計画88Sectionを内蔵）。記録は合言葉で暗号化してSupabase（medasama-tools）に保存。

## 1. Supabase（medasama-tools）でテーブルを作る

SQL Editorで実行：

```sql
create table if not exists public.sharoushi_logs (
  id bigint generated always as identity primary key,
  created_at timestamptz not null default now(),
  payload text not null check (char_length(payload) < 100000)
);
alter table public.sharoushi_logs enable row level security;

-- 読む・追加するだけ許可（更新・削除のポリシーは作らない）
create policy "sharoushi_logs_select" on public.sharoushi_logs for select to anon using (true);
create policy "sharoushi_logs_insert" on public.sharoushi_logs for insert to anon with check (true);
revoke update, delete on public.sharoushi_logs from anon, authenticated;
```

- 中身は暗号化された文字列のみ（payload）。合言葉がないと読めない。
- 記録の訂正は「取消」の記録を追加する方式なので、更新・削除の権限は不要。

## 2. index.html に接続情報を入れる

`index.html` 内の次の2行に、medasama-tools の Project URL と anon key を入れる（他ツールと同じ値）。

```js
const SUPABASE_URL = "";
const SUPABASE_ANON_KEY = "";
```

空のままだと「お試しモード（この端末のみに保存）」で動く。

## 3. GitHub → Vercel

- リポジトリとVercelプロジェクトは作成済み（2026-10-15）。変更したらcommit・push
- push後は `vercel ls` でProductionがReadyか確認

## 4. 初回の使い方

1. 開くと合言葉の入力画面 → 6文字以上の合言葉を決めて入力（この端末に保存される）
2. **合言葉はパスワード管理アプリ等にも控える**（忘れると記録は読めない）
3. 別の端末で使うときも同じ合言葉を入力する

## 計画を変更するとき

計画（88Sectionの予定日・ページ・問題番号）は index.html の `const PLAN = [...]` に内蔵。
計画を見直したら、チャットのClaudeが data/セクション一覧.csv から作り直した index.html を渡すので、それをpushする。
記録（Supabase側）は計画の変更の影響を受けない（Section番号で紐づけ）。

## dataフォルダへの自動保存

初回だけ：設定（⚙）→「保存先フォルダを選ぶ」→ `久山事務所\社労士試験合格\data` を選んで「許可」。
以後は記録・取消・講義の視聴済みを保存するたびに、次の3ファイルが自動で上書きされる（PCのChrome／Edge、Vercel公開版で動作）。

- 演習記録.csv（問題・肢ごとの○△×）
- 学習時間.csv（日ごとの学習時間と内容）
- 講義視聴.csv（全93本の配信目安と視聴日）

ブラウザを開き直して許可が切れていると、画面上部に「保存を許可」ボタンが出る（1回押すだけ）。手動ダウンロードは設定画面から。

## お試しモードで入れた記録を本番に移す

- 本番版と同じページ（同じURL）でお試し記録を入れていた場合：本番版の設定（⚙）→「お試し記録を本番に移す」
- 別の画面（ファイルを直接開いた版など）で入れていた場合：その画面の設定→「お試し記録をファイルに保存」→ 本番版の設定→「ファイルから取り込む」
- 同じ記録は二重に取り込まれない（記録ごとのIDで判定）。お試し側の記録は消さずに残る。
