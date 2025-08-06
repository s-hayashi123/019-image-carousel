# 【React & TypeScript】React HooksとCSSで作る！画像カルーセル開発チュートリアル (019)

## 🎯 学習ゴール
このチュートリアルでは、Webサイトで頻繁に利用される「画像カルーセル（スライダー）」コンポーネントをReactとTypeScriptでゼロから構築します。状態管理の基本である`useState`やライフサイクルを扱う`useEffect`、そしてCSSの`transform`プロパティを駆使したアニメーションまで、フロントエンド開発の必須スキルを実践的に学びます。

**完成形のイメージ:**
- 左右のボタンで画像を切り替えられる
- 下部のインジケーターで現在の位置がわかり、クリックで移動できる
- (発展) 自動で画像がスライドする

この課題を通して、「状態（State）に応じてUIがどう変化するか」というReactの基本的な考え方を体得しましょう。

---

## 📚 事前準備：環境構築

まずは、開発環境を整えましょう。Viteを使って、React + TypeScriptのプロジェクトを高速に立ち上げます。

### 1. Viteプロジェクトの作成
ターミナルを開き、以下のコマンドを実行してください。

```bash
# npm 6.x
npm create vite@latest my-carousel-app --template react-ts

# npm 7+, extra double-dash is needed:
npm create vite@latest my-carousel-app -- --template react-ts

# yarn
yarn create vite my-carousel-app --template react-ts

# pnpm
pnpm create vite my-carousel-app --template react-ts
```

プロジェクトが作成されたら、そのディレクトリに移動し、必要なパッケージをインストールします。

```bash
cd my-carousel-app
npm install
```

### 2. Tailwind CSSの導入（スタイリング用）
今回はUIのスタイリングを効率化するためにTailwind CSSを導入します。

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

次に、`tailwind.config.js`ファイルを開き、`content`部分を以下のように設定します。

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

最後に、`src/index.css`の中身を以下の3行に書き換えて、Tailwind CSSを読み込みます。

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

これで準備は完了です！ `npm run dev` を実行して、開発サーバーを起動しましょう。

---

## 🚀 開発ステップ

### Step 1: コンポーネントの骨格と画像データの準備

まず、カルーセルで表示するための画像を用意しましょう。好きな画像を3〜5枚ほど用意し、`public/images`フォルダ（なければ作成）に保存してください。

次に、`src/components`フォルダを作成し、その中に`Carousel.tsx`というファイルを作成します。

```tsx
// src/components/Carousel.tsx

import React from 'react';

// TODO: カルーセルで表示する画像のURLを配列として定義しましょう
const images = [
  '/images/image1.jpg', // publicフォルダからのパス
  '/images/image2.jpg',
  '/images/image3.jpg',
];

const Carousel = () => {
  // ここにロジックを実装していきます
  return (
    <div className="relative w-full max-w-2xl mx-auto">
      <p>カルーセルコンポーネント</p>
    </div>
  );
};

export default Carousel;
```

`App.tsx`でこのコンポーネントを呼び出して、画面に「カルーセルコンポーネント」と表示されることを確認してください。

### Step 2: 現在の画像を表示する（useStateの導入）

カルーセルは「今、何番目の画像を表示しているか」という**状態**を持つ必要があります。この状態管理のために`useState`フックを使いましょう。

```tsx
// src/components/Carousel.tsx
import React, { useState } from 'react'; // useStateをインポート

// ... (images配列)

const Carousel = () => {
  // TODO: 現在表示している画像のインデックスを管理するstateを定義しましょう
  // 初期値は0（最初の画像）とします
  const [currentIndex, setCurrentIndex] = useState(0);

  return (
    <div className="relative w-full max-w-2xl mx-auto">
      {/* 画像を表示するコンテナ */}
      <div className="overflow-hidden relative h-96">
        {/* TODO: images配列をmapで展開し、画像を表示してみましょう */}
        {/* ただし、このままでは全画像が縦に並んでしまいます。次のステップでスライド式にします。 */}
        <img src={images[currentIndex]} alt={`Slide ${currentIndex}`} className="w-full h-full object-cover" />
      </div>
    </div>
  );
};

export default Carousel;
```

**🤔 思考のヒント:**
`useState(0)`とすることで、`currentIndex`という名前の変数に`0`が格納され、それを更新するための`setCurrentIndex`という関数が手に入ります。まずは現在のインデックスの画像が1枚表示されることを確認しましょう。

### Step 3: CSS Transformでスライドを実現する

全画像を横一列に並べ、コンテナごと動かすことでスライドを実現します。これはCSSの`transform: translateX()`を使うと効率的です。

```tsx
// src/components/Carousel.tsx
// ...

const Carousel = () => {
  const [currentIndex, setCurrentIndex] = useState(0);

  return (
    <div className="relative w-full max-w-2xl mx-auto">
      <div className="overflow-hidden relative h-96 rounded-lg">
        {/* 全画像を横並びにするためのインナーコンテナ */}
        <div
          className="flex transition-transform duration-500 ease-in-out"
          // TODO: currentIndexに応じてコンテナ全体を左に動かすスタイルを適用しましょう
          // ヒント: translateX(-${currentIndex * 100}%)
          style={{ transform: `translateX(-${currentIndex * 100}%)` }}
        >
          {/* images配列をmapで展開して、全ての画像を描画 */}
          {images.map((src, index) => (
            <img
              key={index}
              src={src}
              alt={`Slide ${index}`}
              className="w-full h-full object-cover flex-shrink-0"
            />
          ))}
        </div>
      </div>
    </div>
  );
};

export default Carousel;
```

**📚 深掘りコラム: なぜ`transform`なのか？**
`left`や`margin-left`プロパティをアニメーションさせるよりも、`transform`をアニメーションさせる方がブラウザの描画パフォーマンスが良いとされています。`transform`はGPUアクセラレーションが効きやすく、より滑らかなアニメーションが実現できます。

### Step 4: 操作ボタンを実装する

次に、画像を切り替えるための「次へ」「前へ」ボタンを追加します。

```tsx
// src/components/Carousel.tsx
// ...

const Carousel = () => {
  const [currentIndex, setCurrentIndex] = useState(0);

  // TODO: 次の画像へ進むための関数を定義しましょう
  const goToNext = () => {
    // ヒント: (currentIndex + 1) % images.length を使うとループできます
    const newIndex = (currentIndex + 1) % images.length;
    setCurrentIndex(newIndex);
  };

  // TODO: 前の画像へ戻るための関数を定義しましょう
  const goToPrevious = () => {
    // ヒント: (currentIndex - 1 + images.length) % images.length でマイナスになるのを防げます
    const newIndex = (currentIndex - 1 + images.length) % images.length;
    setCurrentIndex(newIndex);
  };

  return (
    <div className="relative w-full max-w-2xl mx-auto">
      {/* ... (画像表示部分) ... */}

      {/* 操作ボタン */}
      <button
        onClick={goToPrevious}
        className="absolute top-1/2 left-2 transform -translate-y-1/2 bg-black bg-opacity-50 text-white p-2 rounded-full"
      >
        &#10094;
      </button>
      <button
        onClick={goToNext}
        className="absolute top-1/2 right-2 transform -translate-y-1/2 bg-black bg-opacity-50 text-white p-2 rounded-full"
      >
        &#10095;
      </button>
    </div>
  );
};

export default Carousel;
```

### Step 5: インジケーターを実装する

今何枚目の画像を見ているかを示す「ドット」型のインジケーターを追加しましょう。

```tsx
// src/components/Carousel.tsx
// ...

const Carousel = () => {
  // ... (state, goToNext, goToPrevious)

  // TODO: インジケーターのドットをクリックしたときに、その画像にジャンプする関数を定義しましょう
  const goToSlide = (index: number) => {
    setCurrentIndex(index);
  };

  return (
    <div className="relative w-full max-w-2xl mx-auto">
      {/* ... (画像表示部分とボタン) ... */}

      {/* インジケーター */}
      <div className="absolute bottom-4 left-1/2 -translate-x-1/2 flex space-x-2">
        {images.map((_, index) => (
          <button
            key={index}
            onClick={() => goToSlide(index)}
            className={`w-3 h-3 rounded-full ${
              currentIndex === index ? 'bg-white' : 'bg-gray-400'
            }`}
          ></button>
        ))}
      </div>
    </div>
  );
};

export default Carousel;
```

---

## 🔥 挑戦課題

### 1. 自動再生機能の実装 (Easy)

`useEffect`と`setInterval`を使って、5秒ごとに自動で次のスライドに切り替わる機能を実装してみましょう。

**🤔 思考のヒント:**
- `useEffect`内で`setInterval`をセットします。
- `setInterval`のコールバック関数内で`goToNext()`を呼び出します。
- **重要:** コンポーネントがアンマウントされる際に`clearInterval`でタイマーをクリーンアップするのを忘れないように！さもないとメモリリークの原因になります。

```tsx
// useEffectのヒント
useEffect(() => {
  // TODO: 5秒ごとにgoToNextを実行するタイマーをセットする
  const timer = setInterval(goToNext, 5000);

  // TODO: クリーンアップ関数でタイマーをクリアする
  return () => {
    clearInterval(timer);
  };
}, [currentIndex]); // currentIndexが変わるたびにタイマーをリセットするために依存配列に設定
```

### 2. マウスホバーで自動再生を停止 (Medium)

カルーセルにマウスカーソルが乗っている間は、自動再生を一時停止する機能を実装してみましょう。
`onMouseEnter`と`onMouseLeave`イベントを使います。

### 3. Framer Motionでリッチなアニメーション (Hard)

CSS Transitionの代わりに、アニメーションライブラリ`Framer Motion`を使って、より表現力豊かなアニメーションを実装してみましょう。

```bash
npm install framer-motion
```

`AnimatePresence`と`motion`コンポーネントを使うと、画像の出入り（enter/exit）を宣言的に定義できます。

```tsx
// Framer Motion実装のヒント
import { motion, AnimatePresence } from 'framer-motion';

// ...
<div className="overflow-hidden relative h-96">
  <AnimatePresence initial={false}>
    <motion.img
      key={currentIndex}
      src={images[currentIndex]}
      initial={{ opacity: 0, x: 100 }} // 右から入ってくる
      animate={{ opacity: 1, x: 0 }}
      exit={{ opacity: 0, x: -100 }} // 左へ消えていく
      transition={{ duration: 0.5 }}
      className="absolute w-full h-full object-cover"
    />
  </AnimatePresence>
</div>
```

---

## 振り返り (memo)

お疲れ様でした！このチュートリアルでは、
- **状態管理:** `useState`でUIの状態を管理する方法
- **イベント処理:** ボタンクリックで状態を更新する方法
- **CSSアニメーション:** `transform`を使った滑らかなアニメーション
- **ライフサイクル:** `useEffect`を使った副作用（タイマー処理）の管理とクリーンアップ

といった、React開発のコアとなる概念を学びました。
特に「UIは状態の写し鏡である」という考え方は非常に重要です。`currentIndex`というたった一つの状態を変更するだけで、表示される画像、インジケーターのハイライトが連動して変化するのを体感できたはずです。

この基本を応用して、ぜひ挑戦課題にも取り組んでみてください！
