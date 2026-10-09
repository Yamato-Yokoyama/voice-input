# SETUP — 開発環境の作り方(MacBook Air M4)

2026-10-08 に構築して動作を確認した手順。

## ツール

```sh
brew install cmake ninja llvm openjdk@21 gradle uv ffmpeg
```

| 用途 | ツールと確認したバージョン | 備考 |
|---|---|---|
| C++ | Apple clang 21、CMake 4.4、Ninja 1.13 | Xcode Command Line Tools の clang を使う |
| C++ 静的解析 | clang-tidy(Homebrew LLVM 23) | keg-only。`/opt/homebrew/opt/llvm/bin/clang-tidy` で呼ぶ |
| Java | OpenJDK 21.0.12、Gradle 9.8 | keg-only。既定の `java` は 17 のまま。下の「Java 21」を参照 |
| Python | uv、Python 3.12.15 | `eval/` で `.python-version` により 3.12 に固定 |
| TypeScript | Node 26.9 | CLAUDE.md の想定は Node 22。第6週の前に揃えるか決める |
| 音声変換 | ffmpeg 9.0 | whisper.cpp 用に 16kHz モノラル wav へ変換 |

### Java 21

Gradle の toolchain が Homebrew の JDK 21 を見つけられるように、一度だけ次を実行する(sudo が必要):

```sh
sudo ln -sfn /opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-21.jdk
```

## whisper.cpp

`cpp/third_party/whisper.cpp` に submodule として v1.9.5 を固定している。

```sh
git submodule update --init
cmake -S cpp/third_party/whisper.cpp -B cpp/build/whisper -G Ninja \
  -DCMAKE_BUILD_TYPE=Release -DGGML_METAL=ON -DWHISPER_BUILD_TESTS=OFF
cmake --build cpp/build/whisper -j
```

モデルは `models/` に置く(git に入れない)。

```sh
mkdir -p models
for m in small medium; do
  curl -L -o models/ggml-$m.bin https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-$m.bin
done
```

動作確認(Metal で動くことと、日本語の書き起こし):

```sh
say -v Kyoko -o /tmp/t.aiff "シープラスプラスのビルドが通らないので、クロードに聞いてみます"
ffmpeg -y -i /tmp/t.aiff -ar 16000 -ac 1 -c:a pcm_s16le /tmp/t.wav
cpp/build/whisper/bin/whisper-cli -m models/ggml-small.bin -l ja -f /tmp/t.wav
```

ログに `GPU name: MTL0 (Apple M4)` が出れば Metal が使われている。合成音声1本での確認なので、精度と速さの数字としては使わない(数字は `data/testset/v1/` で測る)。

## 評価環境(eval/)

```sh
cd eval
uv sync
uv run python -c "import jiwer, pandas, matplotlib"
```
