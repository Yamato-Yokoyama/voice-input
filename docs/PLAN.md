# PLAN — 8週の計画と完了条件

期間: 2026-10-12(月)〜 2026-12-06(日)。1週1本。応募で問われる C++ と Spring Boot を最初の3週で形にする。

各週は「完了条件」をすべて満たしたら完了。満たせなかった項目は次週に持ち越し、`results/week-NN.md` に理由を書く。

## 第1週(10/12〜10/18) リポジトリとベースライン

- [x] リポジトリを GitHub に公開(README、CLAUDE.md、docs/)
- [x] M4 Mac で whisper.cpp をビルドし、Metal で動かす
- [ ] 普段の話し方で50発話を録音し、正解を付けて `data/testset/v1/` に凍結(半分以上に技術用語・固有名詞を含める)
- [ ] 辞書語リスト `data/testset/v1/bias_terms.txt` を作る(B-WER 用)
- [ ] `eval/` に最小の Python スクリプトを置き、iPhone 音声入力と whisper.cpp の MER を測る

**完了条件**: `results/week-01.md` に2エンジンの MER・CER・B-WER の表がある。成功の定義の目標値(MER をいくつまで下げるか)が決まっている

## 第2週(10/19〜10/25) C++ エンジン

- [ ] `cpp/` に whisper.cpp を包むライブラリ: モデルの読み込み・解放を RAII のクラスに閉じ込める(生ポインタは境界だけ)
- [ ] `DecodeStrategy` インターフェースと実装3つ(言語固定 ja、自動判定、辞書ヒント付き)= Strategy パターン
- [ ] CMake でビルド、GoogleTest で単体テスト
- [ ] AddressSanitizer でテストを回す。clang-tidy をかける。指摘と直し方を README に残す
- [ ] whisper.cpp の server サンプルを読み、自作ライブラリを HTTP サーバー(`POST /transcribe`)として公開

**完了条件**: `cmake --build` と `ctest` が通る。ASan でエラー0。HTTP で wav を送ると文字と処理時間が返る。3つの Strategy の MER を表にした `results/week-02.md`

## 第3週(10/26〜11/1) Spring Boot の司令塔

- [ ] `java/` に Spring Boot プロジェクト(Gradle)
- [ ] `Recognizer` インターフェースと `WhisperCppRecognizer`(C++ の HTTP を呼ぶ)
- [ ] 設定(`application.yml`)と依存性注入で Recognizer を切り替える
- [ ] `POST /api/transcribe`(音声を受け取り、文字・使ったエンジン・遅延を返す)
- [ ] JUnit 5 のテスト(Recognizer はモックで)

**完了条件**: `./gradlew test` が通る。curl で音声を送ると Spring Boot 経由で C++ エンジンの結果が返る。`results/week-03.md` に Spring Boot 経由で増えた遅延を記録

## 第4週(11/2〜11/8) Python 評価ハーネスと CI

- [ ] `eval/` のハーネス: REST API 経由でテストセットを流し、MER・CER・B-WER・U-WER・遅延 p50/p95・RTF を CSV と Markdown 表に出す
- [ ] `docker-compose.yml`: C++ エンジンと Spring Boot を1コマンドで起動
- [ ] GitHub Actions: PR ごとに C++(ビルド・テスト)、Java(テスト)、公開データの固定セット(10発話程度)での評価
- [ ] 公開データの利用条件を確認し、README に出典を書く

**完了条件**: `docker compose up` と `python -m eval.run` の2コマンドで、全指標の表が出る。PR に CI の緑のチェックが付く

## 第5週(11/9〜11/15) 辞書と LLM で直す、エンジン比較

- [ ] 辞書ヒントを C++ エンジンに渡し、B-WER と U-WER を比べる(寄せすぎの副作用を見る)
- [ ] Spring Boot から Gemini に認識結果を渡して文脈で直す `GeminiCorrector`。MER の改善と増えた遅延を並べる
- [ ] `VoskRecognizer`(Java API)と `GeminiRecognizer` を追加
- [ ] 全エンジン × 補正あり/なしを1つの表にする

**完了条件**: `results/week-05.md` に、エンジン4種 × 補正の有無の表(MER、B-WER、U-WER、p95)

## 第6週(11/16〜11/22) MCP でエージェントにつなぐ

- [ ] `mcp/` に TypeScript の MCP サーバー: `transcribe`(音声ファイル → 文字)、`evaluate`(エンジン名 → 指標の表)
- [ ] Claude Code に登録し、「辞書に1語足して評価を回し、前回と比べる」を Claude Code から実行する
- [ ] その手順と結果を README に残す

**完了条件**: Claude Code から MCP 経由で評価が回り、前回との差分が表で返る

## 第7週(11/23〜11/29) 速くして毎日使う

- [ ] C++ 側で話し終わりの検出(VAD)とストリーミング。処理ごとの時間を計測
- [ ] 遅延 p95 を1秒未満に近づける。どこを削ったかを記録
- [ ] Mac のホットキーで話すと、どのアプリにも文字が入る
- [ ] Spring Boot を常時起動し、Actuator で状態と遅延の指標を見る。毎日の使用ログ(遅延、手で直したか)を記録開始

**完了条件**: 毎日の運用記録が始まっている。`results/week-07.md` に遅延の内訳(録音、認識、補正、貼り付け)

## 第8週(11/30〜12/6) 回帰チェックとまとめ

- [ ] 公開データの普通の発話で、改善による悪化がないか確認
- [ ] 8週分の結果を README の1ページにまとめる(問題、構成、結果の表、学んだこと)
- [ ] 運用記録の集計(何日使ったか、誤り率と遅延の推移)

**完了条件**: README だけで、何を解いて、どう測って、どれだけ良くなったかが分かる

## フェーズ2(12月以降・候補)

求人の要件で残る穴を埋める順に並べる。始める前に `docs/APPLY.md` を見直して順番を決める。

| 候補 | 埋まる要件 | 中身 |
|---|---|---|
| Whisper を自分の声で LoRA 微調整 | PyTorch、fine-tuning | Colab の GPU で小さいモデルを自分の録音で微調整し、同じテストセットで比べる |
| Spring Boot をクラウドに置く | クラウド、GCP | コンテナを Cloud Run に置き、iPhone から使う |
| 運用ダッシュボード | React / TypeScript | 毎日の誤り率と遅延の推移を見る画面 |
| 音声エージェント化 | 音声対話、エージェント | 認識 → LLM → 音声合成、ドイツ語の会話練習とパソコン操作 |
