---
title: [Rust] 関数における所有権と借用の使い分け [Rust初心者がGPTと学ぶ Day 3]
tags: Rust 初心者 学習記録
author: sak9ra
slide: false
---
## ChatGPTとRust No.3

### 本シリーズについて
ChatGPTにRustの課題を出してもらい、初学者の筆者が回答 → 解説 → 反省という流れで学んでいくシリーズです。  
 詳細は本記事末尾の[補足・詳細](#補足詳細)をご覧ください。

:::note warn
ChatGPT+プログラミング初学者による記事ですので誤った内容を含む可能性があります。
:::

**本シリーズ記事の目次**
- [課題](#課題by-chatgpt)
- [回答・解説](#回答解説)
- [筆者の回答・採点結果](#筆者の回答採点結果)
  
## 課題（by ChatGPT）
難易度：基礎
想定時間：15〜30分

**問題**
以下のコードは、関数 calculate_length に String を引数として渡しています。このコードをベースに、関数呼び出し後も元の文字列を使用可能とするように修正してください。

```rust: problem.rs
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(s1);
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: String) -> usize {
    s.len()
}
```

**条件**
- s1 の所有権を calculate_length に渡さずに処理できるようにしてください
- 変更後のコードでは s1 を println! で使用できること
- 所有権と借用に関する適切な使い分けを意識してください


**課題内容**

このコードがコンパイルエラーになる理由を説明してください。
上記条件を満たすように、コードを修正してください（s1 を変更せず calculate_length に渡す方法）。
借用（不変参照）を使うことの利点について、メモリ管理と安全性の観点から200字以内で説明してください。

---
## 解答・解説

#### 課題1. コンパイルエラーの原因

```
error[E0382]: use of moved value: `s1`
  --> src/main.rs:3:27
   |
2  |     let len = calculate_length(s1);
   |                              -- value moved here
3  |     println!("The length of '{}' is {}.", s1, len);
   |                               ^^ value used here after move
```

calculate_length(s1) の呼び出しで、引数の型が String であるため、s1 の所有権が関数内にムーブ（移動）されます。

所有権がムーブされた後の s1 を println! で再利用しようとするため、コンパイラが「使用済みの値を再度使ってはいけない」とエラーを出します



### 課題2. 修正後コード

**方法① スライス（&str）を使う**

```rust: fixed1.rs
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);               // ← &s1 で不変借用
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &str) -> usize {            // ← &str を受け取る
    s.len()
}
```

**方法② &String を受け取る（内部で &str へデリファレンス）**

```rust: fixed2.rs
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {         // ← &String を受け取る
    s.len()
}
```

どちらの場合も s1 の所有権はムーブせず、関数内では参照によってデータを読むだけなので、呼び出し後も s1 を安全に再利用できます

:::note info
String型とスライス型について
`String`はUTF-8の配列で表されるバイトのベクタ（`Vec<u8>`）、`&str`はUTF-8の配列のスライス（`&[u8]`）でいつでも`String`に変換できます。
### Stringと&strの相互変換について
Derefトレイトによる自動デリファレンス
https://doc.rust-lang.org/std/string/struct.String.html#deref
> String implements Deref<Target = str>, and so inherits all of str’s methods. In addition, this means that you can pass a String to a function which takes a &str by using an ampersand (&):

つまり、`Defef<Target = str>`によって、`String`は`str`のメソッドを継承しているということです。
`s.len() s:&String`のように文字列スライス型のメソッドを`&String`型の変数を通じて呼び出すと、内部では`&String -> &str`となるように自動デリファレンスが行われています。
> 正確な内容は後述する公式ドキュメントを参照ください。
:::
:::note warn
&String -> &strのデリファレンスは、
`&( *( *&String -> String) -> str) -> &str `
だと思っていますがあっているかはわかりません...
:::


### 課題3. 借用（不変参照）を使うことの利点

**所有権の非移動**
引数に参照を渡すことで、元の変数の所有権は保持される。関数呼び出し後も再利用可能。

**コスト削減**
ヒープ領域の再確保やデータの深いコピー（clone）が不要になり、高速かつメモリ効率的。

**安全性の担保**
Rust の借用チェッカーがライフタイムを検査し、ダングリング参照やデータ競合を防止する。

---
参考リンク

https://doc.rust-jp.rs/book-ja/ch04-02-references-and-borrowing.html

https://doc.rust-jp.rs/book-ja/ch04-03-slices.html

https://doc.rust-jp.rs/rust-by-example-ja/std/str.html

https://doc.rust-lang.org/std/string/struct.String.html#deref

---

## 筆者の回答・採点結果

### コンパイルエラーになる理由
`calculate_length()`の引数が`String`型なので、実引数として渡した変数の所有権はムーブされてしまう。しかし、元のプログラムでは`calculate_length(s1)`呼出し後に`println!`マクロ内で`s1`を使用しているのでエラーとなっている。

### コードの修正

```diff_rust: fixed.rs
fn main() {
    let s1 = String::from("hello");
-   let len = calculate_length(s1);
+   let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}

- fn calculate_length(s: String) -> usize { 
+ fn calculate_length(s: &str) -> usize {
    s.len()
}
```

### 借用を使うことの利点
Rustにおける借用は、その参照が安全かつ有効であることを保証するという点で安全である。不変参照が参照スコープ内に存在する場合には可変参照の使用ができず、ライフタイムの管理によって借用の有効スコープも保証される。
また、借用によって所有権を管理することなく変数にアクセスできる。

:::note warn
プログラマ目線のことしか答えられていなかった。
参照によって有限のヒープ領域を有効活用できることにも言及できればよかった。
:::


## 補足・詳細

### 環境

- ChatGPT-4o-mini-high
タスク機能に以下のように指示をしています

``` :タスク指示

毎朝5時に、Rustの幅広い要素を系統的に習得するため、まずは基礎トピックをすべてマスターできるように
設計された、15〜30分で取り組めるクイズ、調査テーマ、またはコーディング問題を出してください。  
基礎が理解できた後に応用内容へステップアップするよう成長度合いに合わせて難易度を調整し、各課題には
採点と詳細な解説を含めてください。

```

- Rust 
version : 常にstableを使用（投稿時点で1.87.0）


## 参考


https://doc.rust-jp.rs/book-ja/ch04-02-references-and-borrowing.html

https://doc.rust-jp.rs/book-ja/ch04-03-slices.html

https://doc.rust-jp.rs/rust-by-example-ja/std/str.html

https://doc.rust-lang.org/std/string/struct.String.html#deref
