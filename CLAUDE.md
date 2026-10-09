# CLAUDE.md — voice-input

Claude Code はこのファイルを最初に読み、ここに書かれたルールに従って作業する。

## このプロジェクトは何か

日本語の中に英語の技術用語が混ざる話し方(例: 「C++のビルドが通らない」「Claudeに聞く」)を、iPhone の音声入力より正確に、話し終わりから1秒未満で文字にする道具を作る。自分で毎日使い、誤り率と遅延を記録し続ける。

同時に、応募先の求人が求める言語(C++、Java + Spring Boot、TypeScript、MCP、CI/CD、Docker)の実績を、動くコードと測った数字で作る。趣味の位置づけだが本気でやる。

- 設計メモ(全体像): https://claude.ai/code/artifact/fccf5022-bfb4-456c-8122-43476cc0b84c
- 週ごとの計画と完了条件: `docs/PLAN.md`
- 求人と証拠の対応、応募の準備状況: `docs/APPLY.md`(ローカルのみ。.gitignore 済みで公開しない)
- 評価の決まり: `docs/EVAL.md`
- 開発環境の作り方: `docs/SETUP.md`

## 構成と言語の役割(変えない)

| ディレクトリ | 言語 | 役割 |
|---|---|---|
| `cpp/` | C++17 | 認識エンジン。whisper.cpp を包む自作ライブラリ。RAII でモデルのメモリを管理、デコード設定は Strategy パターン、ストリーミング用リングバッファ。HTTP サーバーとして公開 |
| `java/` | Java 21 + Spring Boot 3 | 司令塔。REST API、`Recognizer` インターフェースを依存性注入で差し替え(whisper.cpp / Vosk / Gemini)、辞書補正、Gemini での文脈補正、ログ、Actuator |
| `eval/` | Python 3.12 | 評価ハーネス。REST API 経由でテストセットを流し、jiwer で MER・CER・B-WER・U-WER、遅延 p50/p95、RTF を出す |
| `mcp/` | TypeScript | MCP サーバー。`transcribe` と `evaluate` をツールとして公開し、Claude Code から評価を回せるようにする |
| `docker-compose.yml` | YAML | C++ エンジンと Spring Boot を1コマンドで起動 |
| `.github/workflows/` | YAML | PR ごとに C++ と Java のテスト、公開データの小さな固定セットで評価 |
| `data/` | — | 自分の録音と正解。**git に入れない**(.gitignore 済み) |
| `results/` | CSV / MD | 週ごとの評価結果。git に入れる |

依存の向き: `eval/` と `mcp/` は Spring Boot の REST API だけを呼ぶ。Spring Boot は C++ エンジンの HTTP を呼ぶ。逆向きの依存を作らない。

## 開発環境

- MacBook Air M4、メモリ 16GB。whisper.cpp は Metal で動かす。モデルは small か medium から始め、メモリを見て決める
- C++: CMake、GoogleTest、AddressSanitizer、clang-tidy
- Java: Gradle(Kotlin DSL)、JUnit 5
- Python: uv で仮想環境、jiwer、pandas、matplotlib
- TypeScript: Node 22、公式の MCP TypeScript SDK
- LLM: Gemini API(無料枠には個人的な音声・内容を送らない)

## 作業のルール

1. **測ってから変える**。改善の前に必ず同じテストセットでベースラインを測り、変更後の数字と並べる。数字のない「良くなった」は書かない
2. **評価指標を自作しない**。`docs/EVAL.md` の標準指標と正規化だけを使う。正規化ルールを途中で変えない
3. **テストセットは凍結する**。`data/testset/` を作ったら中身を変えない。変えるときは版を上げ、旧版の結果と混ぜない
4. **録音を外に出さない**。`data/` を git・CI・Gemini 無料枠に出さない。CI は公開データの小さな固定セットで回す
5. **週1本で区切る**。各週の終わりに `results/week-NN.md` に「仮説 → 実装 → 結果の表 → 3行の考察」を書く
6. **事実だけ書く**。README、results、応募用の文に、やっていないこと・測っていない数字を書かない。未確認のことは「未確認」と書く
7. ブランチは `feat/weekNN-<内容>`、PR を作ってからマージする。コミットメッセージは英語

## ノートのルール(絶対条件)

置き場所を迷わないため、次の分け方を必ず守る。

| 何を | どこに | 誰が書くか |
|---|---|---|
| 設計、README、コード、評価結果(`results/`)、計画(`docs/`) | このリポジトリ(VS Code) | Claude Code |
| 開発ログ(その日やったこと、詰まったこと、決めたこと) | Obsidian `Project/VoiceInput/devlog/YYYY-MM-DD.md` | Claude Code が作業の終わりに書く(下の「終わり」) |
| 学んだこと、理解のノート、相談や振り返りのメモ、全体スケジュール | Obsidian `Project/VoiceInput/` 配下 | Claude Code が書く。本人が追記する |
| 応募の準備状況の更新 | `docs/APPLY.md` と Obsidian のハブの「応募」節の両方 | Claude Code |

- Obsidian のハブ: `Project/VoiceInput/Hub — 日英混在の音声入力.md`。迷ったらここに戻る
- 開発ログは Obsidian のテンプレート `Templates/開発ログテンプレート.md` の形(Done / Stuck / Decision / Insight of the day / Next)で書く
- Obsidian の Vault の場所: `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault/`。Obsidian MCP が無いときは、このフォルダのファイルを直接読み書きする
- 直接も MCP も使えないときは、`notes-pending/YYYY-MM-DD.md` に同じ内容を書いて本人に伝える(このフォルダは git に入れない)

## セッションの始め方と終わり方

**始め**(「前回の続き」「確認リストの続き」などと言われたら、作業の前に):

1. Obsidian の `Project/VoiceInput/devlog/` で一番新しい日付のファイルを読み、最後のエントリの Next を確認する
2. ハブの「今どこ？」と、`Project/VoiceInput/` にある確認リスト・ハンズオンのノートで、本人が今どこまで進んだか(チェックの付き具合)を見る
3. `gh issue list --milestone <今の週>` で残りの Issue を見る
4. 上の3つから「今日やること」を1〜3個に絞って提案する

**終わり**(本人が「終わり」「今日はここまで」と言ったら、全部まとめてやる。本人は Issue を自分で閉じなくてよい):

1. この回で終わった Issue を閉じる。コメントに `devlog YYYY-MM-DD` と何をしたかを1行書く。終わったか迷う Issue は閉じずに本人に聞く
2. 開発ログを書く(同じ日のファイルがあれば、エントリを足す)
3. ハブの「今どこ？」を更新する
4. 証拠が増えたら `docs/APPLY.md` とハブの「応募」節を更新する
5. `hub sync` を実行し、project-hub の集計を Obsidian の `Project/Hub/` に写す
6. やったこと(閉じた Issue、書いたノート)を本人に短く報告する

- 進捗の数え方: project-hub が数えるのは GitHub の Milestone(W1〜W8)に紐づいた Issue だけ。Obsidian のチェックは数えない。新しい週に入るときは、`docs/PLAN.md` のその週の項目を Issue にして Milestone に付ける

## 応募について(最重要)

- **応募はこのプロジェクトの完成を待たない**。求人が見つかったらすぐ出す。進行中のプロジェクトとして GitHub のリンクを付ける
- 週が終わるたびに、`docs/APPLY.md` の「今言えること」を更新する。そこに書けるのは、完了して測った事実だけ
- 求人の一覧は claude.ai のプロジェクト「DE Jobhunt」の `job-tracker.md` にある。新しい求人で足りない言語・技術が見つかったら、`docs/APPLY.md` の表に足し、計画のどこで埋めるかを決める
