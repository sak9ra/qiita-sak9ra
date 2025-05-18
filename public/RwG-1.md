---
title: [Rust] 所有権の移動とコピー型の挙動 [Rust初心者がGPTと学ぶ Day 1] 
tags:
  - 'Rust'
  - '100DaysOfCode'
  - '初心者'
private: true
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## ChatGPTとRust No.1

### 本シリーズについて

プログラミング初心者の筆者がChatGPTと二人三脚で定期的にRustを勉強します。詳細は本記事末尾の[補足・詳細](#補足詳細)をご覧ください。
ChatGPTにRustに関する問題を出題してもらい、それに回答する形式で学習します。

:::note warn
ChatGPT+プログラミング初学者による記事ですので誤った内容を含む可能性があります。
:::

**筆者のRust歴**
[TRPL (the book)](https://doc.rust-jp.rs/book-ja/title-page.html)を１周だけしました。

https://doc.rust-jp.rs/book-ja

**本シリーズ記事の目次**
- [課題](#課題by-chatgpt)（主にコーディング問題）
- [回答・解説](#回答解説)
- [筆者の回答・採点結果](#筆者の回答採点結果)
  
## 課題（by ChatGPT）
難易度：基礎
想定時間：15〜30分

問題
以下のコードの挙動について答え、必要に応じて修正してください。

```rust: problem.rs
fn main() {
    let x = 5;
    let y = x;

    println!("x = {}, y = {}", x, y);

    let s1 = String::from("hello");
    let s2 = s1;

    println!("s1 = {}, s2 = {}", s1, s2);
}
```
### 課題内容
1. このコードをコンパイルするとどのようなエラーが出るか説明してください
2. なぜ x → y の代入は許され、s1 → s2 の代入は許されないのかを、所有権や Copy トレイトの観点から説明してください。 
3. `s2 = s1.clone()` に変更したときの動作とその意義について説明してください
4. このプログラムをコンパイルエラーなしに動作させる正しい修正版を示してください



---
## 解答・解説

#### 課題1. このコードをコンパイルするとどのようなエラーが出るか説明してください

Rust のエラー `error[E0382]: use of moved value: ‘s1’` は、次のコードが原因です：

```rust:
let s1 = String::from("hello");
let s2 = s1;         // ← String は Copy でないためムーブ
println!("s1 = {}, s2 = {}", s1, s2); 
//         ^ value used here after move
```
String はヒープメモリを管理する型で、Copy トレイトを実装していません。そのため s1 のムーブが行われ、以降 s1 を使うとコンパイラが所有権が消えた変数を参照しようとするエラーを出します。

### 課題2. なぜ x → y の代入は許され、s1 → s2 の代入は許されないのかを、所有権や Copy トレイトの観点から説明してください。

- **i32**
プリミティブな整数型で、スタック上に固定長データしか持たないため Copy トレイトを実装しています。`let y = x;` とした場合、ビット単位で値が複製され、x も y も同一の数値を保持します。

- **String**  
ヒープ領域を指すポインタや長さなど複数のフィールドを持ち、かつ Drop（解放処理）を実装しているため Copy を実装できません。

:::note info
StringにCopyを実装できない理由
Stringはヒープ上のデータへのポインタを持っています。仮にStringをコピーすると、このポインタがコピーされてしまい、複数の変数が同じヒープ領域を管理することになるので、Drop（ヒープの解放）時に二重で解放する危険があります。
:::

`let s2 = s1;` は s1 の所有権を移動（ムーブ）し、元の s1 は無効になります。
<br>
### **課題3. `s2 = s1.clone()` に変更したときの動作とその意義について説明してください**

Clone トレイトを通じて「深いコピー（deep copy）」を行います。
ヒープ上に新たなバッファを確保し、元の文字列データをコピーして s2 に格納。
s1 は引き続き元のバッファを管理する所有者として有効なので、s1／s2 両方を安全に使えます。

:::note warn
ヒープ確保やデータコピーのコストが発生するため、本番コードでは必要性に応じて使い分けましょう
:::


### **課題4. このプログラムをコンパイルエラーなしに動作させる正しい修正版を示してください**

```rust:fixed.rs
fn main() {
    let x = 5;
    let y = x;                   // i32 は Copy → ムーブではなくコピー

    println!("x = {}, y = {}", x, y);

    let s1 = String::from("hello");
    let s2 = s1.clone();         // clone で deep-copy

    println!("s1 = {}, s2 = {}", s1, s2);  // ムーブせず両方が有効
}
```

s1.clone() によって String の中身を複製し、s1 の所有権を保持したまま s2 に独立したデータを持たせます。

これにより「所有権移動後の無効参照」というコンパイルエラーを解消できます。
<br>

:::note info
参考リンク

[the book -所有権-](https://doc.rust-jp.rs/book-ja/ch04-01-what-is-ownership.html)
[Docs.rs -Copy-](https://doc.rust-lang.org/std/marker/trait.Copy.html?utm_source=chatgpt.com)

:::

## 筆者の回答・採点結果
