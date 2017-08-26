# Symfony Meetup #17

「いいね」機能の実装で見る
Inheritance MappingとPolymorphicと
抽象クラスとインターフェイスと

---

- 最近プロダクトで実装した機能が知見に溢れてました
- 「設計の良さ」をどう実感してもらうか？

---

# 自己紹介
shimokei53

---?image=https://i.gyazo.com/d2ea4fdf57ce313f186e787e29b78574.png&size=auto 90%

+++?image=https://i.gyazo.com/75b333a3fe47afec5dea297a8776cf93.png&size=auto 90%

+++?image=https://i.gyazo.com/e71b51b5e94b3e7faa17841fd7d61490.png&size=auto 90%

---
## 「いいね」機能

+++
## 「いいね」する対象が...

- ブログ |
- ニュース | 
- フォト |
- これら全てに「いいね」できるようにしたい |
- 「いいね」した一覧を表示したい |
- （しかも今後も対象が増えるかも） | 

+++

?????

+++

(いらすとや"困った人のイラスト")

---

# クラス図

- id
- user_id
- blog_id
- news_id
- photo_id
- etc...

+++

# テーブル構成

+++

# Table Inheritance Mapping

+++

()

---

# 実コード

AbstractLike

+++

LikeBlog
LikeNews

+++

これでポリモーフィックはできたぞ！
- 「いろんなもの」とリレーションを持つ場合はポリモーフィック

---

# 「いいね」を「する」コードをどう書くか？

+++

```
class User
{
  public function doLike($item){
    if ($item instanceof Blog) {
      $like = new LikeBlog();
    } else if ($item instanceof News) {
      $like = new LikeNews();
    }
  }
}
```

+++

外から「いいねされるもの」を判定しなくてはならない

これは正しいPolymorphicではない

+++

絶対に `if ($item instanceof Hoge)` を書かない！

+++

```
class User
{
  public function doLike($item){
    $like = $item->createLike($this);
  }
}
```

```
class Blog
{
  public function createLike(User $user){
    return new LikeBlog();
  }
}
```

```
class News
{
  public function createLike(User $user){
    return new LikeNews();
  }
}
```

+++

if文を書きたくなったら…
→処理される対象によって、異なる処理が必要なとき
→「処理される対象」そのものに処理のロジックを移してしまう

---

# 「いいね」される側をどう書くか?

+++


```
class Blog
{
  public function createLike(User $user){
    return new LikeBlog();
  }
}
```

```
class News
{
  public function createLike(User $user){
    return new LikeNews();
  }
}
```

+++

# 「いいね」されるためには
- `createLike` メソッドを作ってやれば良い|
- そのことを今後も強制したい

+++ 
# Interface
