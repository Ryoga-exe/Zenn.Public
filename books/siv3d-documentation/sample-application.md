---
title: "サンプル集 | アプリ"
free: true
---

## スケッチ

```cpp
# include <Siv3D.hpp>

void Main()
{
	// キャンバスのサイズ
	constexpr Size canvasSize{ 600, 600 };

	// ペンの太さ
	constexpr int32 thickness = 8;

	// ペンの色
	constexpr Color penColor = Palette::Orange;

	// 書き込み用の画像データを用意
	Image image{ canvasSize, Palette::White };

	// 表示用のテクスチャ（内容を更新するので DynamicTexture）
	DynamicTexture texture{ image };

	while (System::Update())
	{
		if (MouseL.pressed())
		{
			// 書き込む線の始点は直前のフレームのマウスカーソル座標
			// （初回はタッチ操作時の座標のジャンプを防ぐため、現在のマウスカーソル座標にする）
			const Point from = (MouseL.down() ? Cursor::Pos() : Cursor::PreviousPos());

			// 書き込む線の終点は現在のマウスカーソル座標
			const Point to = Cursor::Pos();

			// image に線を書き込む
			Line{ from, to }.overwrite(image, thickness, penColor);

			// 書き込み終わった image でテクスチャを更新
			texture.fill(image);
		}

		// 描いたものを消去するボタンが押されたら
		if (SimpleGUI::Button(U"Clear", Vec2{ 640, 40 }, 120))
		{
			// 画像を白で塗りつぶす
			image.fill(Palette::White);

			// 塗りつぶし終わった image でテクスチャを更新
			texture.fill(image);
		}

		// テクスチャを表示
		texture.draw();
	}
}
```


## ピアノ

```cpp
# include <Siv3D.hpp>

void Main()
{
	// 白鍵の大きさ
	constexpr Size keySize{ 55, 400 };

	// 楽器の種類
	constexpr GMInstrument instrument = GMInstrument::Piano1;

	// ウインドウをリサイズ
	Window::Resize((12 * keySize.x), keySize.y);

	// 鍵盤の数
	constexpr int32 NumKeys = 20;

	// 音を作成
	std::array<Audio, NumKeys> sounds;
	for (auto i : step(NumKeys))
	{
		sounds[i] = Audio{ instrument, static_cast<uint8>(PianoKey::A3 + i), 0.5s };
	}

	// 対応するキー
	constexpr std::array<Input, NumKeys> keys =
	{
		KeyTab, Key1, KeyQ,
		KeyW, Key3, KeyE, Key4, KeyR, KeyT, Key6, KeyY, Key7, KeyU, Key8, KeyI,
		KeyO, Key0, KeyP, KeyMinus, KeyEnter,
	};

	// 描画位置計算用のオフセット値
	constexpr std::array<int32, NumKeys> keyPositions =
	{
		0, 1, 2, 4, 5, 6, 7, 8, 10, 11, 12, 13, 14, 15, 16, 18, 19, 20, 21, 22
	};

	while (System::Update())
	{
		// キーが押されたら対応する音を再生
		for (auto i : step(NumKeys))
		{
			if (keys[i].down())
			{
				sounds[i].playOneShot(0, 0.5);
			}
		}

		// 白鍵を描画
		for (auto i : step(NumKeys))
		{
			// オフセット値が偶数
			if (IsEven(keyPositions[i]))
			{
				RectF{ (keyPositions[i] / 2 * keySize.x), 0, keySize.x, keySize.y }
					.stretched(-1).draw(keys[i].pressed() ? Palette::Pink : Palette::White);
			}
		}

		// 黒鍵を描画
		for (auto i : step(NumKeys))
		{
			// オフセット値が奇数
			if (IsOdd(keyPositions[i]))
			{
				RectF{ (keySize.x * 0.68 + keyPositions[i] / 2 * keySize.x), 0, (keySize.x * 0.58), (keySize.y * 0.62) }
					.draw(keys[i].pressed() ? Palette::Pink : Color(24));
			}
		}
	}
}
```


## 万華鏡スケッチ
![](/images/doc_v6/quick-example/2.png)
```cpp
# include <Siv3D.hpp>

void Main()
{
	// キャンバスのサイズ
	constexpr Size canvasSize{ 600, 600 };
	
	// ウィンドウをキャンバスのサイズに
	Window::Resize(canvasSize);
	
	// 分割数
	constexpr int32 N = 12;

	// 背景色
	constexpr Color backgroundColor{ 20, 40, 60 };

	// 書き込み用の画像
	Image image{ canvasSize, backgroundColor };

	// 画像を表示するための動的テクスチャ
	DynamicTexture texture{ image };

	while (System::Update())
	{
		if (MouseL.pressed())
		{
			// 画面の中心が (0, 0) になるようにマウスカーソルの座標を移動
			const Vec2 begin = (MouseL.down() ? Cursor::PosF() : Cursor::PreviousPosF()) - Scene::Center();
			const Vec2 end = (Cursor::PosF() - Scene::Center());

			for (auto i : step(N))
			{
				// 円座標に変換
				std::array<Circular, 2> cs = { begin, end };

				for (auto& c : cs)
				{
					// 角度をずらす
					c.theta = IsEven(i) ? (-c.theta - 2_pi / N * (i - 1)) : (c.theta + 2_pi / N * i);
				}

				// ずらした位置をもとに、画像に線を書き込む
				Line{ cs[0], cs[1] }.moveBy(Scene::Center())
					.overwrite(image, 2, HSV{ Scene::Time() * 60.0, 0.5, 1.0 });
			}

			// 書き込んだ画像でテクスチャを更新
			texture.fillIfNotBusy(image);
		}
		else if (MouseR.down()) // 右クリックされたら
		{
			// 画像を塗りつぶす
			image.fill(backgroundColor);

			// 塗りつぶした画像でテクスチャを更新
			texture.fill(image);
		}

		// テクスチャを描く
		texture.draw();
	}
}
```


## Image to Polygon

```cpp
# include <Siv3D.hpp>

void Main()
{
	// 使用する画像
	const Image image{ U"example/siv3d-kun.png" };

	// テクスチャ
	const Texture texture{ image };

	// アルファ値 1 以上の領域を Polygon 化
	const Polygon polygon = image.alphaToPolygon(1, AllowHoles::No);

	// 移動のオフセット
	constexpr Vec2 pos{ 40, 40 };

	// Polygon 単純化時の基準距離（ピクセル）
	double maxDistance = 4.0;

	// 単純化した Polygon
	Polygon simplifiedPolygon = polygon.simplified(maxDistance);

	while (System::Update())
	{
		// 単純化した Polygon の三角形数を表示
		ClearPrint();
		Print << simplifiedPolygon.num_triangles() << U" triangles";

		texture.draw(pos);

		// 単純化した Polygon を表示
		simplifiedPolygon.movedBy(pos)
			.draw(ColorF{ 1.0, 1.0, 0.0, 0.2 })
			.drawWireframe(2, Palette::Yellow);

		// 単純化した Polygon をマウスカーソルに追従させて表示
		simplifiedPolygon.movedBy(Cursor::Pos() - image.size() / 2)
			.draw(ColorF{ 0.5 });

		// Polygon 単純化時の基準距離を制御
		if (SimpleGUI::Slider(U"{:.1f}"_fmt(maxDistance), maxDistance, 0, 30, Vec2{ 400, 40 }, 60, 200))
		{
			// スライダーに変更があれば、単純化した Polygon を新しい基準値で再作成
			simplifiedPolygon = polygon.simplified(maxDistance);
		}
	}
}
```


## Sketch to Polygon

```cpp
# include <Siv3D.hpp>

void Main()
{
	// ウィンドウを 1280x720 にリサイズ
	Window::Resize(1280, 720);

	// 作成した Polygon の配列
	Array<Polygon> polygons;

	// 書き途中の LineString
	LineString points;

	while (System::Update())
	{
		// 左クリックもしくはクリックしたままの移動が発生したら
		if (MouseL.down() ||
			(MouseL.pressed() && (not Cursor::DeltaF().isZero())))
		{
			points << Cursor::PosF();
		}
		else if (MouseL.up())
		{
			points = points.simplified(2.0);

			if (const Polygon polygon = Polygon::CorrectOne(points))
			{
				polygons << polygon;
			}

			points.clear();
		}

		// それぞれの Polygon を描画
		for (auto [i, polygon] : Indexed(polygons))
		{
			polygon.draw(HSV{ (i * 20), 0.4, 1.0 })
				.drawWireframe(1, Palette::Gray)
				.drawFrame(4, HSV{ i * 20 });
		}

		points.draw(4);
	}
}
```


## 音楽プレイヤー
パソコンに保存されている音楽ファイルを再生します。
![](/images/doc_v6/quick-example/13.png)
```cpp
# include <Siv3D.hpp>

void Main()
{
	// 音楽
	Audio audio;

	// FFT の結果
	FFTResult fft;

	// 再生位置の変更の有無
	bool seeking = false;

	while (System::Update())
	{
		ClearPrint();

		// 再生・演奏時間
		const String time = FormatTime(SecondsF{ audio.posSec() }, U"M:ss")
			+ U'/' + FormatTime(SecondsF{ audio.lengthSec() }, U"M:ss");

		// プログレスバーの進み具合
		double progress = static_cast<double>(audio.posSample()) / audio.samples();

		if (audio.isPlaying())
		{
			// FFT 解析
			FFT::Analyze(fft, audio);

			// 結果を可視化
			for (auto i : step(Min(Scene::Width(), static_cast<int32>(fft.buffer.size()))))
			{
				const double size = Pow(fft.buffer[i], 0.6f) * 1000;
				RectF{ Arg::bottomLeft(i, 480), 1, size }.draw(HSV{ 240.0 - i });
			}

			// 周波数表示
			Rect{ Cursor::Pos().x, 0, 1, Scene::Height() }.draw();
			Print << U"{:.2f} Hz"_fmt(Cursor::Pos().x * fft.resolution);
		}

		// 再生
		if (SimpleGUI::Button(U"Play", Vec2{ 40, 500 }, 120, audio && !audio.isPlaying()))
		{
			audio.play(0.2s);
		}

		// 一時停止
		if (SimpleGUI::Button(U"Pause", Vec2{ 170, 500 }, 120, audio.isPlaying()))
		{
			audio.pause(0.2s);
		}

		// フォルダから音楽ファイルを開く
		if (SimpleGUI::Button(U"Open", Vec2{ 300, 500 }, 120))
		{
			audio.stop(0.5s);
			audio = Dialog::OpenAudio();
			audio.play();
		}

		// スライダー
		if (SimpleGUI::Slider(time, progress, Vec2{ 40, 540 }, 130, 590, !audio.isEmpty()))
		{
			audio.pause(0.05s);

			while (audio.isPlaying()) // 再生が止まるまで待機
			{
				System::Sleep(0.01s);
			}

			// 再生位置を変更
			audio.seekSamples(static_cast<size_t>(audio.samples() * progress));

			// ノイズを避けるため、スライダーから手を離すまで再生は再開しない
			seeking = true;
		}
		else if (seeking && MouseL.up())
		{
			// 再生を再開
			audio.play(0.05s);
			seeking = false;
		}
	}

	// 終了時に再生中の場合、音量をフェードアウト
	if (audio.isPlaying())
	{
		audio.fadeVolume(0.0, 0.3s);
		System::Sleep(0.3s);
	}
}
```


## Text to Polygon

```cpp
# include <Siv3D.hpp>

void Main()
{
	Window::Resize(1280, 720);

	Scene::SetBackground(ColorF{ 0.94, 0.91, 0.86 });

	const Font font{ 100, Typeface::Bold };

	// 2D 物理演算のシミュレーションステップ（秒）
	constexpr double stepSec = (1.0 / 200.0);

	// 2D 物理演算のシミュレーション蓄積時間（秒）
	double accumulatorSec = 0.0;

	// 物理演算のワールド
	P2World world;

	// 床
	const P2Body line = world.createLine(P2Static, Vec2{ 0, 0 }, Line{ -1600, 0, 1600, 0 }, OneSided::Yes, P2Material{ 1.0, 0.1, 1.0 });

	// 文字のパーツ
	Array<P2Body> bodies;

	String text;
	int32 generation = 0;
	HashTable<P2BodyID, int32> table;

	// 2D カメラ
	Camera2D camera{ Vec2{ 0, -500 }, 0.38, Camera2DParameters::MouseOnly() };

	constexpr Vec2 textPos{ -400, -500 };

	while (System::Update())
	{
		for (accumulatorSec += Scene::DeltaTime(); stepSec <= accumulatorSec; accumulatorSec -= stepSec)
		{
			// 2D 物理演算のワールドを更新
			world.update(stepSec);
		}

		// 2D カメラの更新
		camera.update();

		// テキストの入力
		TextInput::UpdateText(text);

		{
			// 2D カメラを適用する Transformer2D の作成
			const auto t = camera.createTransformer();

			// 世代に応じた色で Body を描画
			for (const auto& body : bodies)
			{
				body.draw(HSV{ (table[body.id()] * 45 + 30), 0.8, 0.8 });
			}

			// 床を描画
			line.draw(Palette::Green);

			const String currentText = (text + TextInput::GetEditingText());

			// 入力文字を描画
			{
				const Transformer2D scaling(Mat3x2::Scale(2.5));

				font(currentText).draw(textPos, ColorF{ 0.5 });
			}

			// 改行文字が入力されたらテキストを P2Body にする
			if (currentText.includes(U'\n'))
			{
				// 入力文字を PolygonGlyph 化
				const Array<PolygonGlyph> glyphs = font.renderPolygons(currentText.removed(U'\n'));

				// P2Body にする Polygon を得る
				Array<Polygon> polygons;
				{
					Vec2 penPos{ textPos };

					for (const auto& glyph : glyphs)
					{
						for (const auto& polygon : glyph.polygons)
						{
							polygons << polygon
								.movedBy(penPos + glyph.getOffset())
								.scaled(2.5)
								.simplified(2.0);
						}

						penPos.x += glyph.xAdvance;
					}
				}

				for (const auto& polygon : polygons)
				{
					bodies << world.createPolygon(P2Dynamic, Vec2{ 0, 0 }, polygon, P2Material{ 1, 0.0, 0.4 });

					// 現在の世代を保存
					table[bodies.back().id()] = generation;
				}

				text.clear();

				// 世代を進める
				++generation;
			}

			// 2D カメラ、右クリック時の UI を表示
			camera.draw(Palette::Orange);
		}

		// 消去ボタン
		if (SimpleGUI::Button(U"Clear", Vec2{ 1100, 40 }))
		{
			bodies.clear();
		}
	}
}
```


## マンデルブロ集合

```cpp
# include <Siv3D.hpp>

int32 Mandelbrot(double x, double y)
{
	double a = 0.0, b = 0.0;

	for (int32 n = 0; n < 360; ++n)
	{
		const double t = (a * a - b * b + x);
		const double u = (2.0 * a * b + y);

		if (4.0 < (t * t + u * u))
		{
			return n;
		}

		a = t;
		b = u;
	}

	return 0;
}

void Main()
{
	constexpr Size resolutuion{ 640, 480 };
	Window::Resize(resolutuion);

	Vec2 center(0, 0);
	double scale = -4.0;

	// 結果を保存する画像
	Image image{ resolutuion, Palette::Black };

	// 描画用の動的テクスチャ
	DynamicTexture texture(image);

	while (System::Update())
	{
		const double wheel = Mouse::Wheel();
		const bool clicked = MouseL.down();

		// 最初のフレームか、操作されたときだけ更新
		if (wheel || clicked || (Scene::FrameCount() == 1))
		{
			scale -= wheel;

			const double s = Pow(1.25, scale);
			const double d = (1.0 / s) / resolutuion.x;

			if (clicked)
			{
				center += (Cursor::PosF() - resolutuion / 2) * d;
			}

			const double xb = center.x - d * (resolutuion.x * 0.5);
			const double yb = center.y - d * (resolutuion.y * 0.5);

			for (auto y : step(resolutuion.y))
			{
				const double yPos = yb + (d * y);

				for (auto x : step(resolutuion.x))
				{
					const double xPos = xb + (d * x);

					if (const int32 m = Mandelbrot(xPos, yPos))
					{
						image[y][x] = HSV{ (240 - m), 0.8, 1.0 };
					}
					else
					{
						image[y][x] = Palette::Black;
					}
				}
			}

			// 動的テクスチャの中身を image で更新
			texture.fill(image);
		}

		// テクスチャを描画
		texture.draw();
	}
}
```


## ライフゲーム エディタ
![](/images/doc_v6/quick-example/3.png)
```cpp
# include <Siv3D.hpp>

// 1 セルが 1 バイトになるよう、ビットフィールドを使用
struct Cell
{
	bool previous : 1 = 0;
	bool current : 1 = 0;
};

// フィールドをランダムなセル値で埋める関数
void RandomFill(Grid<Cell>& grid)
{
	grid.fill(Cell{});

	// 境界のセルを除いて更新
	for (auto y : Range(1, grid.height() - 2))
	{
		for (auto x : Range(1, grid.width() - 2))
		{
			grid[y][x] = Cell{ 0, RandomBool(0.5) };
		}
	}
}

// フィールドの状態を更新する関数
void Update(Grid<Cell>& grid)
{
	for (auto& cell : grid)
	{
		cell.previous = cell.current;
	}

	// 境界のセルを除いて更新
	for (auto y : Range(1, grid.height() - 2))
	{
		for (auto x : Range(1, grid.width() - 2))
		{
			const int32 c = grid[y][x].previous;

			int32 n = 0;
			n += grid[y - 1][x - 1].previous;
			n += grid[y - 1][x].previous;
			n += grid[y - 1][x + 1].previous;
			n += grid[y][x - 1].previous;
			n += grid[y][x + 1].previous;
			n += grid[y + 1][x - 1].previous;
			n += grid[y + 1][x].previous;
			n += grid[y + 1][x + 1].previous;

			// セルの状態の更新
			grid[y][x].current = (c == 0 && n == 3) || (c == 1 && (n == 2 || n == 3));
		}
	}
}

// フィールドの状態を画像化する関数
void CopyToImage(const Grid<Cell>& grid, Image& image)
{
	for (auto y : step(image.height()))
	{
		for (auto x : step(image.width()))
		{
			image[y][x] = grid[y + 1][x + 1].current
				? Color{ 0, 255, 0 } : Palette::Black;
		}
	}
}

void Main()
{
	// フィールドのセルの数（横）
	constexpr int32 width = 60;

	// フィールドのセルの数（縦）
	constexpr int32 height = 60;

	// 計算をしない境界部分も含めたサイズで二次元配列を確保
	Grid<Cell> grid(width + 2, height + 2, Cell{ 0,0 });

	// フィールドの状態を可視化するための画像
	Image image{ width, height, Palette::Black };

	// 動的テクスチャ
	DynamicTexture texture{ image };

	Stopwatch stopwatch{ StartImmediately::Yes };

	// 自動再生
	bool autoStep = false;

	// 更新頻度
	double speed = 0.5;

	// グリッドの表示
	bool showGrid = true;

	// 画像の更新の必要があるか
	bool updated = false;

	while (System::Update())
	{
		// フィールドをランダムな値で埋めるボタン
		if (SimpleGUI::ButtonAt(U"Random", Vec2{ 700, 40 }, 170))
		{
			RandomFill(grid);
			updated = true;
		}

		// フィールドのセルをすべてゼロにするボタン
		if (SimpleGUI::ButtonAt(U"Clear", Vec2{ 700, 80 }, 170))
		{
			grid.fill({ 0, 0 });
			updated = true;
		}

		// 一時停止 / 再生ボタン
		if (SimpleGUI::ButtonAt(autoStep ? U"Pause" : U"Run ▶", Vec2{ 700, 160 }, 170))
		{
			autoStep = !autoStep;
		}

		// 更新頻度変更スライダー
		SimpleGUI::SliderAt(U"Speed", speed, 1.0, 0.1, Vec2{ 700, 200 }, 70, 100);

		// 1 ステップ進めるボタン、または更新タイミングの確認
		if (SimpleGUI::ButtonAt(U"Step", Vec2{ 700, 240 }, 170)
			|| (autoStep && stopwatch.sF() >= (speed * speed)))
		{
			Update(grid);
			updated = true;
			stopwatch.restart();
		}

		// グリッド表示の有無を指定するチェックボックス
		SimpleGUI::CheckBoxAt(showGrid, U"Grid", Vec2{ 700, 320 }, 170);

		// フィールド上でのセルの編集
		if (Rect(0, 0, 599).mouseOver())
		{
			const Point target = (Cursor::Pos() / 10 + Point{ 1, 1 });

			if (MouseL.pressed())
			{
				grid[target].current = true;
				updated = true;
			}
			else if (MouseR.pressed())
			{
				grid[target].current = false;
				updated = true;
			}
		}

		// 画像の更新
		if (updated)
		{
			CopyToImage(grid, image);
			texture.fill(image);
			updated = false;
		}

		// 画像をフィルタなしで拡大して表示
		{
			ScopedRenderStates2D sampler{ SamplerState::ClampNearest };
			texture.scaled(10).draw();
		}

		// グリッドの表示
		if (showGrid)
		{
			for (auto i : step(61))
			{
				Rect{ 0, i * 10, 600, 1 }.draw(ColorF{ 0.4 });
				Rect{ i * 10, 0, 1, 600 }.draw(ColorF{ 0.4 });
			}
		}

		if (Rect(0, 0, 599).mouseOver())
		{
			Cursor::RequestStyle(CursorStyle::Hidden);
			Rect{ Cursor::Pos() / 10 * 10, 10 }.draw(Palette::Orange);
		}
	}
}
```


## QR コード作成
キーボードでテキストを入力します。  
スマートフォンの QR コードリーダーで読み取ることができます。
![](/images/doc_v6/quick-example/4.png)
```cpp
# include <Siv3D.hpp>

void Main()
{
	Window::Resize(1280, 720);

	// 変換するテキスト
	TextEditState textEdit{ U"Abc" };
	String previous;

	// QR コードを表示するための動的テクスチャ
	DynamicTexture texture;

	while (System::Update())
	{
		// テキスト入力
		SimpleGUI::TextBox(textEdit, Vec2{ 20,20 }, 1240);

		// テキストの更新があれば QR コードを再作成
		if (const String current = textEdit.text;
			current != previous)
		{
			// 入力したテキストを QR コードに変換
			if (const auto qr = QR::EncodeText(current))
			{
				// 枠を付けて拡大した画像で動的テクスチャを更新
				texture.fill(QR::MakeImage(qr).scaled(500, 500, InterpolationAlgorithm::Nearest));
			}

			previous = current;
		}

		texture.drawAt(640, 400);
	}
}
```


## ドットお絵かき

```cpp
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(Palette::White);

	constexpr int32 cellSize = 40;

	// シーンのサイズとセルの大きさから縦横のセルの個数を計算
	Grid<int32> cells(Scene::Size() / cellSize);

	while (System::Update())
	{
		// カーソルを手の形に
		Cursor::RequestStyle(CursorStyle::Hand);

		for (auto p : step(cells.size()))
		{
			const Rect rect{ (p * cellSize), cellSize };

			if (rect.leftClicked())
			{
				// 0 → 1 → 2 →　3 → 0 → 1 → ... と遷移させる
				++cells[p] %= 4;
			}

			rect.stretched(-1).draw(ColorF{ 0.95 - cells[p] * 0.3 });
		}
	}
}
```


## 時計

```cpp
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF{ 0.5, 0.7, 0.6 });

	const Font font{ 50, Typeface::Bold };
	const Vec2 center = Scene::Center();

	while (System::Update())
	{
		Circle{ center, 230 }.draw().drawFrame(20, ColorF{ 0.8 });

		// 数字
		for (auto i : Range(1, 12))
		{
			const Vec2 pos = OffsetCircular{ center, 170, (i * 30_deg ) };
			font(i).drawAt(pos, ColorF{ 0.25 });
		}

		for (auto i : Range(0, 59))
		{
			const Vec2 pos = OffsetCircular{ center, 210, i * 6_deg };
			Circle{ pos, (i % 5 ? 3 : 6) }.draw(ColorF{ 0.25 });
		}

		// 現在時刻を取得
		const DateTime time = DateTime::Now();

		// 時針
		const double hour = ((time.hour + time.minute / 60.0) * 30_deg);
		Line{ center, Arg::direction = Circular(110, hour) }
			.draw(LineStyle::RoundCap, 18, ColorF{ 0.1 });

		// 分針
			const double minute = ((time.minute + time.second / 60.0) * 6_deg);
		Line{ center, Arg::direction = Circular(190, minute) }
			.draw(LineStyle::RoundCap, 8, ColorF{ 0.1 });

		// 秒針
		const double second = (time.second * 6_deg);
		Line{ center, Arg::direction = Circular(190, second) }
			.stretched(40, 0)
			.draw(3, ColorF{ 0.1 });
	}
}
```


## JPEG Glitch

```cpp
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF{ 0.8, 0.9, 1.0 });

	// 画像
	const Image image{ U"example/windmill.png" };

	// 表示用の動的テクスチャ
	DynamicTexture texture{ image };

	// JPEG のバイナリデータ
	const Blob originalBlob = image.encodeJPEG();

	// 改変するデータの個数
	const size_t noiseCount = (image.num_pixels() / 4000);

	while (System::Update())
	{
		if (SimpleGUI::Button(U"Glitch", Vec2{ 20, 20 }))
		{
			// Array を作成
			Blob modifiedBlob = originalBlob;

			for (size_t i = 0; i < noiseCount; ++i)
			{
				// ランダムな位置の 1 バイトについて、ランダムな値に書き換え
				// ヘッダ部分（先頭）は改変しない
				const size_t pos = Random<size_t>(630, modifiedBlob.size() - 1);
				modifiedBlob[pos] = Byte{ RandomUint8() };
			}

			// JPEG データとして読み込んで画像を作成、動的テクスチャに転送
			texture.fill(Image{ MemoryReader{ modifiedBlob }, ImageFormat::JPEG });
		}

		texture.drawAt(Scene::Center());
	}
}
```


## マイクで入力した音の周波数解析

```cpp
# include <Siv3D.hpp>

void Main()
{
	// マイクをセットアップ（ただちに録音をスタート）
	Microphone mic{ StartImmediately::Yes };

	if (not mic)
	{
		// マイクを利用できない場合、終了
		throw Error{ U"Microphone not available" };
	}

	FFTResult fft;

	while (System::Update())
	{
		// FFT の結果を取得
		mic.fft(fft);

		// 結果を可視化
		for (auto i : step(800))
		{
			const double size = Pow(fft.buffer[i], 0.6f) * 1200;
			RectF{ Arg::bottomLeft(i, 600), 1, size }.draw(HSV{ 240 - i });
		}

		// 周波数表示
		Rect{ Cursor::Pos().x, 0, 1, Scene::Height() }.draw();

		ClearPrint();
		Print << U"{} Hz"_fmt(Cursor::Pos().x * fft.resolution);
	}
}
```


## Sketch to P2Body
四角や丸を描くと物体が生成されて物理演算をします。  
マウスホイールや右クリックで視点を移動できます。
![](/images/doc_v6/quick-example/5.png)
```cpp
# include <Siv3D.hpp>

void Main()
{
	// ウィンドウを 1280x720 にリサイズ
	Window::Resize(1280, 720);

	// 2D 物理演算のシミュレーションステップ（秒）
	constexpr double stepSec = (1.0 / 200.0);

	// 2D 物理演算のシミュレーション蓄積時間（秒）
	double accumulatorSec = 0.0;

	// 2D 物理演算のワールド
	P2World world;

	// [_] 地面
	const P2Body ground = world.createLine(P2Static, Vec2{ 0, 0 }, Line{ -600, 0, 600, 0 });

	// 物体
	Array<P2Body> bodies;

	// 2D カメラ
	Camera2D camera{ Vec2{ 0, -300 } };

	LineString points;

	while (System::Update())
	{
		for (accumulatorSec += Scene::DeltaTime(); stepSec <= accumulatorSec; accumulatorSec -= stepSec)
		{
			// 2D 物理演算のワールドを更新
			world.update(stepSec);
		}

		// 地面より下に落ちた物体は削除
		bodies.remove_if([](const P2Body& b) { return (200 < b.getPos().y); });

		// 2D カメラの更新
		camera.update();
		{
			// 2D カメラから Transformer2D を作成
			const auto t = camera.createTransformer();

			// 左クリックもしくはクリックしたままの移動が発生したら
			if (MouseL.down() ||
				(MouseL.pressed() && (not Cursor::DeltaF().isZero())))
			{
				points << Cursor::PosF();
			}
			else if (MouseL.up())
			{
				points = points.simplified(2.0);

				if (const Polygon polygon = Polygon::CorrectOne(points))
				{
					const Vec2 pos = polygon.centroid();

					bodies << world.createPolygon(P2Dynamic, pos, polygon.movedBy(-pos));
				}

				points.clear();
			}

			// すべてのボディを描画
			for (const auto& body : bodies)
			{
				body.draw(HSV{ body.id() * 10.0 });
			}

			// 地面を描画
			ground.draw(Palette::Skyblue);

			points.draw(3);
		}

		// 2D カメラの操作を描画
		camera.draw(Palette::Orange);
	}
}
```


## 画像ビューア

```cpp
# include <Siv3D.hpp>

void Main()
{
	Texture texture;

	while (System::Update())
	{
		// ファイルがドロップされた
		if (DragDrop::HasNewFilePaths())
		{
			// ファイルを画像として読み込めた
			if (const Image image{ DragDrop::GetDroppedFilePaths().front().path })
			{
				// 画面のサイズに合うように画像を拡大縮小
				texture = Texture{ image.fitted(Scene::Size()) };
			}
		}

		if (texture)
		{
			texture.drawAt(Scene::Center());
		}
	}
}
```


## 世界地図

```cpp
# include <Siv3D.hpp>

void Main()
{
	Window::Resize(1280, 720);

	const Array<MultiPolygon> countries = GeoJSONFeatureCollection{ JSON::Load(U"example/geojson/countries.geojson") }.getFeatures()
		.map([](const GeoJSONFeature& f) { return f.getGeometry().getPolygons(); });

	Camera2D camera{ Vec2{ 0, 0 }, 2.0, Camera2DParameters{.maxScale = 4096.0 } };
	Optional<size_t> selected;

	while (System::Update())
	{
		ClearPrint();

		camera.update();
		{
			const auto transformer = camera.createTransformer();
			const double lineThickness = (1.0 / Graphics2D::GetMaxScaling());
			const RectF viewRect = camera.getRegion();

			Print << Cursor::PosF();
			Print << camera.getScale() << U"x";

			Rect{ Arg::center(0, 0), 360, 180 }.draw(ColorF{ 0.2, 0.6, 0.9 }); // 海
			{
				for (auto [i, country] : Indexed(countries))
				{
					// 画面外にある場合は描画をスキップ
					if (!country.computeBoundingRect().intersects(viewRect))
					{
						continue;
					}

					if (country.leftClicked())
					{
						selected = i;
					}

					country.draw((selected == i) ? ColorF{ 0.9, 0.8, 0.7 } : ColorF{ 0.93, 0.99, 0.96 });
					country.drawFrame(lineThickness, ColorF{ 0.25 });
				}
			}
		}
		camera.draw(Palette::Orange);
	}
}
```


## オーディオ処理
音楽にリアルタイムでエフェクトを適用できます。
![](/images/doc_v6/quick-example/14.png)
```cpp
# include <Siv3D.hpp> // OpenSiv3D v0.6

void Main()
{
	Window::Resize(1280, 720);

	Audio audio;
	double posSec = 0.0;
	double volume = 1.0;
	double pan = 0.0;
	double speed = 1.0;
	bool loop = false;

	Array<float> busSamples;
	Array<float> globalSamples;
	FFTResult busFFT;
	FFTResult globalFFT;
	LineString lines(256, Vec2{ 0, 0 });

	bool pitch = false;
	double pitchShift = 0.0;

	bool lpf = false;
	double lpfCutoffFrequency = 800.0;
	double lpfResonance = 0.5;
	double lpfWet = 1.0;

	bool hpf = false;
	double hpfCutoffFrequency = 800.0;
	double hpfResonance = 0.5;
	double hpfWet = 1.0;

	bool echo = false;
	double delay = 0.1;
	double decay = 0.5;
	double echoWet = 0.5;

	bool reverb = false;
	bool freeze = false;
	double roomSize = 0.5;
	double damp = 0.0;
	double width = 0.5;
	double reverbWet = 0.5;

	while (System::Update())
	{
		ClearPrint();
		Print << U"GlobalAudio::GetActiveVoiceCount(): " << GlobalAudio::GetActiveVoiceCount();
		Print << U"isEmpty : " << audio.isEmpty();
		Print << U"isStreaming : " << audio.isStreaming();
		Print << U"sampleRate : " << audio.sampleRate();
		Print << U"samples : " << audio.samples();
		Print << U"lengthSec : " << audio.lengthSec();
		Print << U"posSample : " << audio.posSample();
		Print << U"posSec : " << (posSec = audio.posSec());
		Print << U"isActive : " << audio.isActive();
		Print << U"isPlaying : " << audio.isPlaying();
		Print << U"isPaused : " << audio.isPaused();
		Print << U"samplesPlayed : " << audio.samplesPlayed();
		Print << U"isLoop : " << (loop = audio.isLoop());
		Print << U"getLoopTimingtLoop : " << audio.getLoopTiming().beginPos << U", " << audio.getLoopTiming().endPos;
		Print << U"loopCount : " << audio.loopCount();
		Print << U"getVolume : " << (volume = audio.getVolume());
		Print << U"getPan : " << (pan = audio.getPan());
		Print << U"getSpeed : " << (speed = audio.getSpeed());

		if (SimpleGUI::Button(U"Open audio file", Vec2{ 60, 560 }))
		{
			audio.stop(0.5s);
			audio = Dialog::OpenAudio(Audio::Stream);
		}

		{
			GlobalAudio::BusGetSamples(0, busSamples);
			GlobalAudio::BusGetFFT(0, busFFT);

			for (auto [i, s] : Indexed(busSamples))
			{
				lines[i].set((300.0 + i), (200.0 - s * 100.0));
			}

			if (busSamples)
			{
				lines.draw(2, Palette::Orange);
			}

			for (auto [i, s] : Indexed(busFFT.buffer))
			{
				RectF{ Arg::bottomLeft(300 + i, 300), 1, (s * 4) }.draw();
			}
		}

		{
			GlobalAudio::GetSamples(globalSamples);
			GlobalAudio::GetFFT(globalFFT);

			for (auto [i, s] : Indexed(globalSamples))
			{
				lines[i].set((300.0 + i), (550.0 - s * 100.0));
			}

			if (globalSamples)
			{
				lines.draw(2, Palette::Orange);
			}

			for (auto [i, s] : Indexed(globalFFT.buffer))
			{
				RectF{ Arg::bottomLeft(300 + i, 650), 1, (s * 4) }.draw();
			}
		}

		if (SimpleGUI::Button(U"Play", Vec2{ 600, 20 }, 80, !audio.isPlaying()))
		{
			audio.play();
		}

		if (SimpleGUI::Button(U"Pause", Vec2{ 690, 20 }, 80, (audio.isPlaying() && !audio.isPaused())))
		{
			audio.pause();
		}

		if (SimpleGUI::Button(U"Stop", Vec2{ 780, 20 }, 80, (audio.isPlaying() || audio.isPaused())))
		{
			audio.stop();
		}

		if (SimpleGUI::Button(U"Play in 2s", Vec2{ 870, 20 }, 120, !audio.isPlaying()))
		{
			audio.play(2s);
		}

		if (SimpleGUI::Button(U"Pause in 2s", Vec2{ 1000, 20 }, 120, (audio.isPlaying() && !audio.isPaused())))
		{
			audio.pause(2s);
		}

		if (SimpleGUI::Button(U"Stop in 2s", Vec2{ 1130, 20 }, 120, (audio.isPlaying() || audio.isPaused())))
		{
			audio.stop(2s);
		}

		if (SimpleGUI::Slider(U"{:.1f} / {:.1f}"_fmt(posSec, audio.lengthSec()), posSec, 0.0, audio.lengthSec(), Vec2{ 600, 60 }, 160, 360))
		{
			if (MouseL.down() || !Cursor::DeltaF().isZero()) // シークの連続（ノイズの原因）を防ぐ
			{
				audio.seekTime(posSec);
			}
		}

		if (SimpleGUI::CheckBox(loop, U"Loop", Vec2{ 1130, 60 }))
		{
			audio.setLoop(loop);
		}

		if (SimpleGUI::Slider(U"volume: {:.2f}"_fmt(volume), volume, Vec2{ 600, 110 }, 140, 130))
		{
			audio.setVolume(volume);
		}

		if (SimpleGUI::Button(U"0.0 in 2s", Vec2{ 880, 110 }, 110, audio.isActive()))
		{
			audio.fadeVolume(0.0, 2s);
		}

		if (SimpleGUI::Button(U"0.5 in 2s", Vec2{ 1000, 110 }, 110, audio.isActive()))
		{
			audio.fadeVolume(0.5, 2s);
		}

		if (SimpleGUI::Button(U"1.0 in 2s", Vec2{ 1120, 110 }, 110, audio.isActive()))
		{
			audio.fadeVolume(1.0, 2s);
		}

		if (SimpleGUI::Slider(U"pan: {:.2f}"_fmt(pan), pan, -1.0, 1.0, Vec2{ 600, 150 }, 140, 130))
		{
			audio.setPan(pan);
		}

		if (SimpleGUI::Button(U"-1.0 in 2s", Vec2{ 880, 150 }, 110, audio.isActive()))
		{
			audio.fadePan(-1.0, 2s);
		}

		if (SimpleGUI::Button(U"0.0 in 2s", Vec2{ 1000, 150 }, 110, audio.isActive()))
		{
			audio.fadePan(0.0, 2s);
		}

		if (SimpleGUI::Button(U"1.0 in 2s", Vec2{ 1120, 150 }, 110, audio.isActive()))
		{
			audio.fadePan(1.0, 2s);
		}

		if (SimpleGUI::Slider(U"speed: {:.3f}"_fmt(speed), speed, 0.0, 4.0, Vec2{ 600, 190 }, 140, 130))
		{
			audio.setSpeed(speed);
		}

		if (SimpleGUI::Button(U"0.8 in 2s", Vec2{ 880, 190 }, 110, audio.isActive()))
		{
			audio.fadeSpeed(0.8, 2s);
		}

		if (SimpleGUI::Button(U"1.0 in 2s", Vec2{ 1000, 190 }, 110, audio.isActive()))
		{
			audio.fadeSpeed(1.0, 2s);
		}

		if (SimpleGUI::Button(U"1.2 in 2s", Vec2{ 1120, 190 }, 110, audio.isActive()))
		{
			audio.fadeSpeed(1.2, 2s);
		}

		bool updatePitch = false;
		bool updateLPF = false;
		bool updateHPF = false;
		bool updateEcho = false;
		bool updateReverb = false;

		if (SimpleGUI::CheckBox(pitch, U"Pitch", Vec2{ 600, 240 }, 120, GlobalAudio::SupportsPitchShift()))
		{
			if (pitch)
			{
				updatePitch = true;
			}
			else
			{
				GlobalAudio::BusClearFilter(0, 0);
			}
		}
		updatePitch |= SimpleGUI::Slider(U"pitchShift: {:.2f}"_fmt(pitchShift), pitchShift, -12.0, 12.0, Vec2{ 720, 240 }, 160, 300);

		if (SimpleGUI::CheckBox(lpf, U"LPF", Vec2{ 600, 280 }, 120))
		{
			if (lpf)
			{
				updateLPF = true;
			}
			else
			{
				GlobalAudio::BusClearFilter(0, 1);
			}
		}
		updateLPF |= SimpleGUI::Slider(U"cutoffFrequency: {:.0f}"_fmt(lpfCutoffFrequency), lpfCutoffFrequency, 10, 4000, Vec2{ 720, 280 }, 220, 240);
		updateLPF |= SimpleGUI::Slider(U"resonance: {:.2f}"_fmt(lpfResonance), lpfResonance, 0.1, 8.0, Vec2{ 720, 310 }, 220, 240);
		updateLPF |= SimpleGUI::Slider(U"wet: {:.2f}"_fmt(lpfWet), lpfWet, Vec2{ 720, 340 }, 220, 240);

		if (SimpleGUI::CheckBox(hpf, U"HPF", Vec2{ 600, 380 }, 120))
		{
			if (hpf)
			{
				updateHPF = true;
			}
			else
			{
				GlobalAudio::BusClearFilter(0, 2);
			}
		}
		updateHPF |= SimpleGUI::Slider(U"cutoffFrequency: {:.0f}"_fmt(hpfCutoffFrequency), hpfCutoffFrequency, 10, 4000, Vec2{ 720, 380 }, 220, 240);
		updateHPF |= SimpleGUI::Slider(U"resonance: {:.2f}"_fmt(hpfResonance), hpfResonance, 0.1, 8.0, Vec2{ 720, 410 }, 220, 240);
		updateHPF |= SimpleGUI::Slider(U"wet: {:.2f}"_fmt(hpfWet), hpfWet, Vec2{ 720, 440 }, 220, 240);

		if (SimpleGUI::CheckBox(echo, U"Echo", Vec2{ 600, 480 }, 120))
		{
			if (echo)
			{
				updateEcho = true;
			}
			else
			{
				GlobalAudio::BusClearFilter(0, 3);
			}
		}
		updateEcho |= SimpleGUI::Slider(U"delay: {:.2f}"_fmt(delay), delay, Vec2{ 720, 480 }, 220, 240);
		updateEcho |= SimpleGUI::Slider(U"decay: {:.2f}"_fmt(decay), decay, Vec2{ 720, 510 }, 220, 240);
		updateEcho |= SimpleGUI::Slider(U"wet: {:.2f}"_fmt(echoWet), echoWet, Vec2{ 720, 540 }, 220, 240);

		if (SimpleGUI::CheckBox(reverb, U"Reverb", Vec2{ 600, 580 }, 120))
		{
			if (reverb)
			{
				updateReverb = true;
			}
			else
			{
				GlobalAudio::BusClearFilter(0, 4);
			}
		}
		updateReverb |= SimpleGUI::CheckBox(freeze, U"freeze", Vec2{ 720, 580 }, 110);
		updateReverb |= SimpleGUI::Slider(U"roomSize: {:.2f}"_fmt(roomSize), roomSize, 0.001, 1.0, { 830, 580 }, 150, 200);
		updateReverb |= SimpleGUI::Slider(U"damp: {:.2f}"_fmt(damp), damp, Vec2{ 720, 610 }, 220, 240);
		updateReverb |= SimpleGUI::Slider(U"width: {:.2f}"_fmt(width), width, Vec2{ 720, 640 }, 220, 240);
		updateReverb |= SimpleGUI::Slider(U"wet: {:.2f}"_fmt(reverbWet), reverbWet, Vec2{ 720, 670 }, 220, 240);

		if (pitch && updatePitch)
		{
			GlobalAudio::BusSetPitchShiftFilter(0, 0, pitchShift);
		}

		if (lpf && updateLPF)
		{
			GlobalAudio::BusSetLowPassFilter(0, 1, lpfCutoffFrequency, lpfResonance, lpfWet);
		}

		if (hpf && updateHPF)
		{
			GlobalAudio::BusSetHighPassFilter(0, 2, hpfCutoffFrequency, hpfResonance, hpfWet);
		}

		if (echo && updateEcho)
		{
			GlobalAudio::BusSetEchoFilter(0, 3, delay, decay, echoWet);
		}

		if (reverb && updateReverb)
		{
			GlobalAudio::BusSetReverbFilter(0, 4, freeze, roomSize, damp, width, reverbWet);
		}
	}

	if (GlobalAudio::GetActiveVoiceCount())
	{
		GlobalAudio::FadeVolume(0.0, 0.5s);
		System::Sleep(0.5s);
	}
}
```