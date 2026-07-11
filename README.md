# youtube-digest-agent

Claude Code 用のサブエージェント定義。YouTube のチャンネル URL または動画 URL を渡すと、自動字幕から内容を要約して Markdown に保存する。

## できること

- **チャンネル URL を渡す** → 取得条件（並び順・件数）を確認してから一括取得
  - 並び順: 最新順 / 人気順（再生数降順）/ 古い順 / 期間指定
  - 出力: `<起動ディレクトリ>/YouTube/<チャンネル名>/` に動画ごとの要約 MD ＋ 索引 `_index.md`
- **動画 URL を渡す** → 確認なしで即実行し、同じ規則で 1 ファイル保存
- 字幕は日本語優先、無ければ英語（要約は日本語で出力）

## 導入

1. このリポジトリの `youtube-digest.md` を `~/.claude/agents/` 配下（任意のサブディレクトリ可）に置く

   ```bash
   mkdir -p ~/.claude/agents
   curl -fsSL https://raw.githubusercontent.com/Ubik0914/youtube-digest-agent/main/youtube-digest.md \
     -o ~/.claude/agents/youtube-digest.md
   ```

2. yt-dlp を導入する

   ```bash
   brew install yt-dlp   # macOS
   ```

3. Claude Code のセッションを開き直すとエージェントが読み込まれる

## 使い方

Claude Code のプロンプトに URL を貼るだけ。

```
https://www.youtube.com/@Fireship/videos このチャンネルの動画をまとめて
```

```
https://youtu.be/XXXXXXXXXXX この動画を要約して
```

## 仕組み

- `yt-dlp --flat-playlist` で動画一覧を取得（`--extractor-args "youtube:lang=ja"` の有無で日本語タイトルと再生数を 2 回に分けて取得し id で結合）
- `yt-dlp --write-auto-subs --sub-format json3` で自動字幕を取得し、テキスト化して要約
- yt-dlp が使えない場合はページ埋め込みの `ytInitialData`（`lockupViewModel`）パースにフォールバック

## 注意

- 要約は自動生成字幕にもとづくため、固有名詞に誤変換が含まれる場合がある（明らかなものは補正し MD 内に注記される）
- 字幕が存在しない動画はメタ情報のみの MD になる
- 連続取得時に YouTube 側のレート制限（429）が出ることがある。その場合エージェントはスリープを挟んで再試行する
