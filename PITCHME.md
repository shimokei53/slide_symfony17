# Symfony Meetup #17

「いいね」機能の実装で見る
Inheritance MappingとPolymorphicと
抽象クラスとインターフェイスと

---

- 最近プロダクトで実装した機能が知見に溢れていた
- 「良い設計」は手を動かさないと、良さを実感しにくい？

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

### クラス図

+++?image=https://i.gyazo.com/8c7d7af47e470bd6fd3db39fb9482fb0.png&size=auto 90%

+++

### テーブル構成

+++?image=https://i.gyazo.com/c30e7452d2e55a962ff1a016ea99e177.png&size=auto 90%

+++?image=https://i.gyazo.com/d12e14e272604ace9e8261d1b2302d01.png&size=auto 90%

+++

# Table Inheritance Mapping

+++?image=https://i.gyazo.com/953ef58be366e9adef01d8bd52049c29.png&size=auto 90%

---

AbstractLike

```
/**
 * @ORM\DiscriminatorColumn(name="item_type", type="string")
 * @ORM\DiscriminatorMap({
 *   "Abstract": "Like",
 *   "Blog":   "LikeBlog",
 *   "News":   "LikeNews",
 * })
 * @ORM\InheritanceType(value="SINGLE_TABLE")
 */
abstract class Like
{
}
```

+++

LikeBlog

```
class LikeBlog extends Like
{
    /**
     * @ORM\ManyToOne(targetEntity="Blog", inversedBy="likes")
     * @ORM\JoinColumn(name="item_id", referencedColumnName="id", nullable=false)
     */
    private $item;

    /**
     * @return Blog
     */
    public function getItem()
    {
        return $this->item;
    }
}
```

+++

LikeNews

```
class LikeNews extends Like
{
    /**
     * @ORM\ManyToOne(targetEntity="News", inversedBy="likes")
     * @ORM\JoinColumn(name="item_id", referencedColumnName="id", nullable=false)
     */
    private $item;

    /**
     * @return News
     */
    public function getItem()
    {
        return $this->item;
    }
}
```
@[4](ここだけNewsに変更)

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
    // else if...がたくさん増えそう
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
- →「処理される対象」によって、異なる処理が必要なとき |
- →「処理される対象」そのものにロジックを移してしまう |

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
- `createLike`メソッドを作ってやれば良い |
- そのことを今後も強制したい

+++ 
# Interface

+++
```
interface LikeableInterface
{
    public function createLike(User $user);
}
```

```
class Blog implements LikeableInterface
{
  public function createLike(User $user){
    return new LikeBlog();
  }
}
```

```
class Hoge implements LikeableInterface
{
  public function createLike(User $user){
    return new LikeHoge();
  }
}
```
