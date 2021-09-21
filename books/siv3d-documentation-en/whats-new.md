---
title: "リリースノート"
free: true
---

# v0.6.1

## 新機能
- SDF / MSDF テクスチャ描画時のパラメータを簡単に指定できる `Graphics2D::SetSDFParameters(const TextStyle&)`, `Graphics2D::SetMSDFParameters(const TextStyle&)` を追加しました ([#647](https://github.com/Siv3D/OpenSiv3D/issues/647))

## ユーザビリティ向上
- Windows 版のプロジェクトで発生していたビルド時の IntelliSense の警告を 2 件抑制しました ([#643](https://github.com/Siv3D/OpenSiv3D/issues/643))
- ドキュメンテーションを追加しました

## 仕様変更
- `Monitor` は `MonitorInfo` に名前が変更されました ([#649](https://github.com/Siv3D/OpenSiv3D/issues/649))
- `UUID` は `UUIDValue` に名前が変更されました

## 不具合・バグ修正
- v0.6.0 において、Windows 版でタッチ操作をしたときに左クリックと判定されにくくなっていた不具合を修正しました ([#645](https://github.com/Siv3D/OpenSiv3D/issues/645))
- v0.6.0 において、`<Siv3D/Windows/Windows.hpp>` をインクルードするとコンパイルエラーになっていた問題を修正しました ([#644](https://github.com/Siv3D/OpenSiv3D/issues/644))
- v0.6.0 において、`Platform::Windows::GetHWND()` が実装されていなかった問題を修正しました ([#646](https://github.com/Siv3D/OpenSiv3D/issues/646))
- `MathParser` に空の文字列を渡したときに例外を投げないようにしました ([#641](https://github.com/Siv3D/OpenSiv3D/issues/641))


# v0.6.0

[サンプルコード、画像・動画での紹介](https://zenn.dev/reputeless/scraps/f2e28f40ba44f174d87a)

:::message
非常に多くの更新があるため、現在も執筆中です。
:::

## 新機能

### Featured

- 基本的な 3D 描画に対応しました
- C++20 に対応し、Concepts や Designated initialization, コンストラクタへの `[[nodiscard]]`, 宇宙船演算子、より多くの `constexpr`, 新しい標準機能ライブラリ機能などが活用されています
- 試験的な Web 版の実装を追加しました (詳しくは [OpenSiv3D for Web](https://siv3d.kamenokosoft.com/ja/index))
- Windows で OpenGL バックエンドが利用可能になりました (詳しくは チュートリアル 35)
- ファイルの非同期ダウンロードなどを行う HTTP クライアント機能を追加しました
- デフォルトで HighDPI に対応しました
- SVG ファイルの読み込みに対応しました
- MIDI ファイルの読み込みに対応しました
- 動画をテクスチャとして扱える `VideoTexture` クラスを追加しました
- Windows でペンタブレットの入力（筆圧・傾き）を取得する機能を追加しました
- 2D 描画でカスタム頂点シェーダを利用できるようになりました。3D でも頂点シェーダ、ピクセルシェーダをカスタマイズできます
- すべてのプラットフォームでオーディオのフェードイン・フェードアウト（再生、停止、音量、パン、スピード）をサポートしました
- HPF, LPF, ピッチシフトなどのリアルタイム音声フィルタ機能を追加しました
- 文字の輪郭や Polygon を正確に取得できるようになりました
- `Font` のレンダリング形式に SDF / MSDF を指定できるようになりました
- `Font` の拡大縮小描画、輪郭、シャドウに対応しました
- オーディオファイルのストリーミング再生に対応しました
-


### その他

- `String` 型に対応した、正規表現を扱う機能を追加しました
- 実行ファイルに埋める文字列の難読化をする機能を追加しました
- デマングルを行う関数を追加しました
- Kahan の加算アルゴリズムを行うクラスを追加しました
- 128-bit 整数型を追加しました
- `Stopwatch` や `Timer` がユーザ定義の基準時刻インタフェース `ISteadyClock` を利用することで、複数の `Stopwatch` や `Timer` を一括して一時停止させたり、遅く/早く進行させることが容易になりました
- `TimeProfiler` がより詳細な統計情報を提供するようになりました
- 地理空間データの交換形式である GeoJSON を読み込む機能を追加しました
- 多くの数学定数を追加しました
- `JSONReader`, `JSONWriter` を統合した `JSON` クラスを追加しました
- 簡易的なキーフレームによるアニメーションを行う `SimpleAnimation` クラスを追加しました
- 統計処理を行う関数を追加しました
- 数値に応じたカラーマップを行う機能を追加しました
- ベクトルクラスに多数の便利なメンバ関数を追加しました
- 図形クラスに多数の便利なメンバ関数を追加しました
- `Shape2D` にハート形、両方向矢印、Squircle 形を追加しました 
- `Polygon` に柔軟にテクスチャをマッピングする機能を追加しました
- 長方形詰込みに 90° 回転を許可するオプションを追加しました
- ホモグラフィ変換を行うための機能を追加しました
- 各種乱数関数が乱数エンジンを引数に取れるようになりました
- UUID 生成機能を追加しました
- 環境変数の取得機能を追加しました
- コマンドライン引数の取得機能を追加しました
- モニターの物理サイズなど、詳細な情報を取得できるようになりました
- シリアル通信の詳細なオプションを追加しました
- Klat 方式による音声読み上げ機能を追加しました
- 画像形式 WebP, TIFF の読み込みに対応しました
- 音声形式 Opus, AIFF, FLAC, MIDI, WMA の読み込みに対応しました
- 画像の一部分に画像処理を適用できるようになりました
- GrabCut 機能を追加しました
- QR コード生成機能を改善しました
- `VideoWriter` を改善しました
- サウンドフォントファイルを利用できるようになりました
- マウスカーソルのスタイルを追加しました
- 全てのキー入力を取得する関数を追加しました
- アセット管理における非同期ロードがより便利になりました
- example ファイルを多数追加しました
- ナビメッシュがより簡単に使えるようになりました
- `Spline2D` クラスを追加しました
- 図形の輪郭の一部の取得に対応しました
- 図形の Lerp に対応しました
- GPU だけでの三角形描画に対応しました
- カスタムマウスカーソルに対応しました
- オーディオをグループ化し、グループごとに音量を調整できる機能を追加しました
- Ogg Vorbis のループタグ取得に対応しました
- レーベンシュタイン距離機能を追加しました
- 凹包 (Concave hull) 機能を追加しました
- 柔軟な画像デコーダ、エンコーダクラスを追加されました
- 閉 / 開区間を指定した乱数生成を追加しました
- SIV3D_SET() によるビルド時のエンジン設定機能を追加しました
- Effect の再帰が可能になりました
- CJK フォントを追加し、`Print` が中国語、韓国語の表示に対応しました
- 動画ファイルを読み込む `VideoReader` クラスを追加しました
- 2D 物理演算の機能を追加しました
- Siv3D くんドット絵素材が追加されました
- Siv3D くん .obj 3D モデルファイルが追加されました
- `Image::stamp()` が追加されました
- `Line::drawDoubleHeadedArrow()` で両方向矢印を描けるようになりました
- スクリーンショット保存のショートカットキーをカスタマイズできるようになりました
- スクリプト機能を大幅に改善しました
- 2D 図形の交差判定をより多くの組み合わせに対応しました
- 多くの 3D 形状のクラスを追加しました
- Linux 版の IMEの挙動を改善しました 
- (執筆中）
-
-
-
-
-
- 
- ユーザアドオンの追加機能を追加しました
- その他多数の新機能が追加されています

## パフォーマンス向上
- Windows 版のアプリの起動時間が数百ミリ秒前後短縮しました
- `Heterogeneous lookup` により、文字列リテラルや `StringView` で `HashTable` や `HashSet` のルックアップをする際の実行時性能が向上しました
- ファイルの読み書き、画像ファイルや音声ファイルのロードが高速になりました
- `Parse` / `ParseOpt` / `ParseOr` の速度を改善しました
- (執筆中）
- 
- 
- 
- 

## ユーザビリティ向上
- インライン関数が .hpp ファイルから .ipp ファイルに移され、ヘッダファイルが読みやすくなりました
- Windows 版のプロジェクトがデフォルトでプリコンパイル済みファイルを使用するようになり、ビルドが高速化しました
- (執筆中）
- 
- 


## 仕様変更
- bool 型の関数パラメータの多くが、名前の付いた YesNo 型に置き換えられ、コードの可読性が向上しました
- `Optional` 型が C++ 標準の `std::optional` に近い動作をするよう改善されました
- `HashTable` 型が C++ 標準の `std::unordered_map` に近い動作をするよう改善されました
- `KDTree` がより短い記述で利用可能になりました
- `Concurrenttask` は `AsyncTask` に名前が変更されました
- 子プロセス作成関数は `ChildProcess` クラスに統合されました
- `Unicode::FromWString()` は `Unicode::FromWstring()` に名前が変更されました
- 浮動小数点数型の `U"{}"_fmt(x)` は、有効桁数すべてを表示するようになりました
- `Time::Get～` はシステム起動時間ではなく、アプリケーション起動からの時間を返すようになりました
- `CustomStopwatch` は `VariableSpeedStopwatch` に名前が変更されました
- `FileSystem::SpecialFolderPath()` は `FileSystem::GetFolderPath()` に名前が変更されました
- `FileSystem::UniqueFilePath()` は UUID 機能を使って名前を作るようになりました
- `ByteArray` は `Blob` および `MemoryReader` に置き換えられました
- `CSVData` は `CSV` に名前が変更されました
- `INIData` は `INI` に名前が変更されました
- `JSONReader`, `JSONWriter` は `JSON` に統合されました
- `EasingController` は `EasingAB` に名前が変更されました
- `Sprite` は `Buffer2D` に置き換えられました
- インデックス配列は `TriangleIndex` を使うようになりました
- `MessageBox` の仕様が変わりました
- トースト通知の仕様が変わりました
- 物体検出機能は `CascadeClassifier` に置き換えられました
- `Key` は `Input` になりました
- 絵文字とアイコンデータセットを更新し使える絵文字やアイコンの種類が大幅に増えました
- `Image` の最大サイズを 1 辺 8192px → 16384px に拡張しました
- ConstantBuffer サイズ 16 × N バイト制約が撤廃されました
- 並列実行に関する機能は `SIV3D_CONCURRENT` マクロを定義しなくても使えるようになりました
- (執筆中）
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

## 不具合・バグ修正
- `Array` の並列関数の不具合を修正しました
- `AsyncTask` のプラットフォーム間による差異を解消しました
- Windows の `MakeDragDrop()` の不具合を修正しました
- PPM 画像読み込みの不具合を修正しました
- プラットフォームごとの乱数の再現性の改善しました
- (執筆中）
- 
- 
- 
- 
- 
- 


## 注意点
- `Math::SmoothDmap()` の引数順が変更されています。コンパイルエラーで発見できません
- フォントの縦書き機能は一時的に非搭載になりました
- 自然言語処理機能は一時的に非搭載になりました
- `SimpleGUIManager` 機能はキャンセルされました
- `NoiseGenerator ` クラスは一時的に非搭載になりました
- Shift_JIS 形式のテキストファイルはサポートしなくなりました
- シーンのリサイズについて、仕組みが変更されました (チュートリアル 15 参照)
- 絵文字のデザインが変更されました
- 乱数の再現性が v0.4.3 と互換がありません
- 2D 物理演算は cm をデフォルトの単位に変更しました
- `Glyph` 単位での描画の方法が変更されました
- Windows 版は `<Siv3D.hpp>` のプリコンパイル済みを全てのソースファイルで自動でインクルードするようになりました。Main.cpp にある `# include <Siv3D.hpp>` は実質的には無意味です。`# define NO_S3D_USING` が必要な場合はプリコンパイル済みヘッダ作成用ヘッダ `stdafx.h` で行ってください
- `Audio` は `Wave` と互換の形式でデータを保持しなくなりました。`.getWave()` は `.getSamples()` に置き換わりました。`GlobalAudio::BusGetSamples()` も利用できます
- 
- 
- 

## コントリビューション
### コミッタ―
（敬称略）
- [nokotan](https://github.com/nokotan)
  - **Web 版開発を全面的に担当**
- [Ebishu-0309](https://github.com/Ebishu-0309)
  - **`Geometry2D::` に多数の関数を実装**
  - **`Shape2D::Squircle()` の実装**
  - `Bezier2`, `Bezier3` の `.boundingRect()` を実装
  - コードの改善
- [taotao54321](https://github.com/taotao54321)
  - `Grid` の修正
  - コードの改善
- [sthairno](https://github.com/sthairno)
  - **Linux 版の IME 処理改善**
- [itakawa](https://github.com/itakawa) 
  - **Siv3D くん .obj ファイル提供**
- [take-cheeze](https://github.com/take-cheeze)
  - **GitHub Actions を使った CI の整備**
- [Luke256](https://github.com/Luke256)
  - コードの改善
- [YASAI03](https://github.com/YASAI03)
  - **HTTP クライアント機能 `SimpleHTTP` の提案・実装**
- [falrnd](https://github.com/falrnd)
  - `Geometry2D` の交差判定の改善
- [yurkth](https://github.com/yurkth)
  - **`GeoJSON` 関連の機能を提案・実装**
- [ianCK](https://github.com/ianCK)
  - コードの改善
- [lriki](https://github.com/lriki)
  - **Siv3D くんドット絵素材の提供**
- [Ryoga-exe](https://github.com/Ryoga-exe)
  - `Color::gamma()` のバグ修正
- [sivboard](https://github.com/sivboard)
  - スクリプト機能の実装追加とバグ修正
- [agehama](https://github.com/agehama)
  - PPM 画像読み込みのバグ修正
- [kurokoji](https://github.com/kurokoji)
  - **Linux 版 MessageBox の追加**
- [ichi-raven](https://github.com/ichi-raven)
  - コードの改善
- [azaika](https://github.com/azaika)
  - **`JSON` クラスの設計・実装**

### OpenSiv3D Challenge 参加者
- #01 統計関数
  - 白坂, マキア, fal_rnd
- #03 Shape2D::Heart
  - 野菜ジュース, てゃいの
- #04 2D 図形の交差判定
  - Ebishu, fal_rnd, きつねび
- [#05 Squircle](https://zenn.dev/link/comments/ed180236e76181)
  - Ebishu, Ryoga.exe
- [#07 国と都市](https://zenn.dev/link/comments/76224190023391)
  - torin (yurkth)
- [#10 `OutlineGlyph` to `Array<Polygon>`](https://zenn.dev/link/comments/e75e5e80eee914)
  - Ebishu, fal_rnd