# Symfony Meetup #17

「いいね」機能の実装で見る
Inheritance MappingとPolymorphicと

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

+++

# テーブル構成

+++

# Table Inheritance Mapping

+++

()
（カルテットコミュニケーションズさんのブログ）

---

# 実コード

AbstractClip

+++

ClippedBlog
ClippedNews

+++

これでポリモーフィックはできたぞ！
- 「いろんなもの」とリレーションを持つ場合はポリモーフィック

---

# 「いいね」を「する」コードをどう書くか？

+++

```
class User
{
  public function doClip($item){
    if ($item instanceof Blog) {
      $clip = new ClippedBlog();
    } else if ($item instanceof News) {
      $clip = new ClippedNews();
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
  public function doClip($item){
    $clip = $item->generateClip($this);
  }
}
```

```
class Blog
{
  public function createClip(User $user){
    return new ClippedBlog();
  }
}
```

```
class News
{
  public function createClip(User $user){
    return new ClippedNews();
  }
}
```

