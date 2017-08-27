# Symfony Meetup #17

「いいね」機能の実装で見る
Inheritance MappingとPolymorphicと
抽象クラスとインターフェイスと

---

- 最近プロダクトで実装した機能に設計のキモがたくさん
- 「良い設計」は手を動かさないと、良さを実感しにくい？

---

# 自己紹介
shimokei53
![](https://i.gyazo.com/dbadc18da47c016d1450eb1282be2fa7.png&size=80% auto)

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

---

### これらの機能を

- 「いいね」そのもの
  - Userと何らかのObjectの多対多
- 「いいね」を「する」コード
  - User側の処理
- 「いいね」される側のコード
  - Blog,News,Photo,Hoge,...

---

### 「いいね」をどう定義するか？
- いろいろ何らかのObjectとの多対多？？

+++

### クラス図

+++?image=https://i.gyazo.com/8c7d7af47e470bd6fd3db39fb9482fb0.png&size=auto 90%

+++

### テーブル構成

+++?image=https://i.gyazo.com/c30e7452d2e55a962ff1a016ea99e177.png&size=auto 90%

+++?image=https://i.gyazo.com/d12e14e272604ace9e8261d1b2302d01.png&size=auto 80%

+++

# Table Inheritance Mapping

+++?image=https://i.gyazo.com/953ef58be366e9adef01d8bd52049c29.png&size=auto 90%

---

実際のコード

+++

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

+++?image=https://i.gyazo.com/72f2d844ad6d2f26e6c7462a14b474a4.png&size=auto 90%

+++

これでポリモーフィックはできたぞ！
- 「いろんなもの」とリレーションを持つ場合はポリモーフィック
- Single Table inheritanceを使えば上手く実装できそう

---

### 「いいね」を「する」コードをどう書くか？
- User側の処理

+++

```
class User
{
  public function doLike($item){
    // $itemがBlogならLikeBlog
    // $itemがNewsならLikeNews
    // $itemがHogeならLikeHoge
    // ....
    return $like;
  }
}
```

+++

```
class User
{
  public function doLike($item){
    if ($item instanceof Blog) {
      $like = new LikeBlog();
    } else if ($item instanceof News) {
      $like = new LikeNews();
    } // else ifが増えそう…
  }
}
```
@[4-5]
@[6-8]

+++

外から「いいね対象」を判定しなくてはならない

これは正しいPolymorphicではない

+++

絶対に `if ($item instanceof Hoge)` を書かない！

+++

というわけで
```
class User
{
  public function doLike($item){
    $like = $item->createLike($this);
  }
}
```
@[4](対象に任せる)

+++

```
class Blog
{
  public function createLike(User $user){
    $like = new LikeBlog();
    $like->setUser($user);
    $like->setItem($this);
    return $like;
  }
}
```

```
class News
{
  public function createLike(User $user){
    $like = new LikeNews();
    $like->setUser($user);
    $like->setItem($this);
    return $like;
  }
}
```

+++
めんどいのでコンストラクタでやる
```
class Blog
{
  public function createLike(User $user){
    return new LikeBlog($user, $this);
  }
}
```

```
class News
{
  public function createLike(User $user){
    return new LikeNews($user, $this);
  }
}
```
※不要なsetterはなるべく作らないでおくと変更が防げる

+++

if文を書きたくなったら…
- →「処理される対象」によって、異なる処理が必要なとき |
- →「処理される対象」そのものにロジックを移してしまう |

---

### 「いいね」される側をどう定義する?
- Blog
- News
- その他もろもろ側の処理

+++

```
class Blog
{
  public function createLike(User $user){
    return new LikeBlog($user, $this);
  }
}
```

```
class News
{
  public function createLike(User $user){
    return new LikeNews($user, $this);
  }
}
```

+++

### 「いいね」されるためには
- createLikeメソッドを作ってやれば良い |
- そのことを今後も強制したい |

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
    return new LikeBlog($user, $this);
  }
}
```

```
/**
 * 新しく「いいね」対象となったHogeクラス
 */
class Hoge implements LikeableInterface
{
  public function createLike(User $user){
    return new LikeHoge($user, $this);
  }
}
```

+++
「いいね」される側に共通で欲しいメソッドはinterfaceに書いておく

```
interface LikeableInterface
{
    public function createLike(User $user);
    
    /**
     * 例： $userにいいねされているか
     * @return boolean
     */
    public function isLikedBy(User $user);
}
```

---

# まとめ

+++

### なんでも「いいね」できる機能
- 「いいね」そのもの
  - ポリモーフィック + 多対多をSTIで
- 「いいね」する処理
  - if文書かずに処理を移動する
- 「いいね」される側の定義
  - インターフェイスで「いいね」されるためのルールを作る

+++

- 良い設計を心がけると、長い目で見て嬉しいことがたくさんある
- 「動く」コードから「より良い設計」を追求しよう
