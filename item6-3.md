
## 6.3 詳細ページの拡張

### 6.3.1 詳細ページへドリルダウン

以下のようなジャンプができるようにしたい。

- 書籍のリスト→書籍の詳細ページ→著者ページ、出版社ページ
- 著者ページ（執筆した書籍リストを持つ）→各書籍のリスト
- 出版社ページ（出版した書籍リストを持つ）→各書籍のリスト

### 6.3.2 一覧ページにリンクを追加

一覧ページの書名をクリックすると詳細ページにジャンプするようにする。
リスト6-13の(1)のようにViews/Books/Index.cshtmlを編集する
（元々は`@Html.DisplayFor(modelItem => item.Title)`だけだったところを次のように編集する）。

```cshtml
<a asp-action="Details" asp-route-id="@item.Id"> // (1)
    @Html.DisplayFor(modelItem => item.Title)
</a>
```

図6-11のように詳細ページへのリンクが追加される。

### 6.3.3 詳細ページのレイアウト変更

詳細ページの著者名、出版社名についてもクリックすると著者の詳細ページ、出版社の詳細ページにジャンプするようにする。
リスト6-14の(1)、(2)のようにViews/Books/Details.cshtmlを編集する。

```cshtml
<a asp-controller="Authors" asp-action="Details" asp-route-id="@Model.AuthorId"> // (1)
    @Html.DisplayFor(model => model.Author.Name)
</a>
```

`asp-controller`属性を指定すると、他のコントローラー`Action`メソッドを呼び出すことができる。

注：一見、著者のIDは`Model.Author.Id`で取得できるので、`Book`クラスが`AuthorId`を持つのは冗長な気がしたのですが、
https://docs.microsoft.com/ja-jp/ef/core/modeling/relationships によると
「依存エンティティ クラスで定義されている外部キー プロパティを持つことをお勧めしますが、これは必要ありません。」
らしいです。
`AuthorId`は常にDBから取得するので（`Author`はincludeしないとDBから取得しないので）、安全な気もします。
また、DB上はBookテーブルにAuthorId列があるので、`Book`クラスにもあった方が分かりやすいのかもしれません。

### 6.3.4 書籍ページのレイアウト変更

著者の詳細ページに執筆した書籍リストを表示するようにする。
リスト6-15の(1)、(2)のようにViews/Authors/Details.cshtmlを編集する
（`@foreach`を使って箇条書きリストを作るだけ）。

```cshtml
<ul>
    @foreach (var book in Model.Book) // (1)
    {
        <li>
            <a asp-controller="Books" asp-action="Details" asp-route-id="book.Id">@book.Title</a> // (2)
        </li>
    }
</ul>
````

さらにリスト6-16の(3)のようにControllers/AuthorsController.csを編集する。
（`include`メソッドを使って、`Author`をDBから取得するときに`Book`プロパティも取得する）。

```csharp
var author = await _context.Author
    .include(a => a.Prefecture)
    .include(a => a.Book) // (3)
    .SingleOrDefaultAsync(m => m.Id == id);
```

注１：EF Core 1.0の場合、6.2.3で見たように`include`を付けないと`Book`プロパティの値がnullになってしまう（生成されるhtml上では空白で表示）。
`include`メソッドを使うと、`Author`取得時に`Book`プロパティ（ナビゲーションプロパティ）の内容もDBから取得するようなクエリ（SQL文）が発行される。

注２：EF Core 2.1では遅延読み込みが導入されているが、効率には注意が必要であろう。
この例の場合、`include`を付けることでループの外側で（コントローラー側で）`Book`の内容がDBから取得できているが、
遅延読み込みに頼るとループの内側でDBから取得することになる効率が悪い。
詳しくは
https://docs.microsoft.com/ja-jp/ef/core/querying/related-data
を参照。

注３：（Core以前の）Entity Frameworkについては昔から（4以上で）遅延読み込みが導入されていた。

### 6.3.5 出版社ページのレイアウトを変更

著者ページと同様に修正する（略）。

（`include`を使う代わりに）`Controller`クラスの`Detail`メソッドでLINQを使って作成することもできる。

```csharp
ViewData["Books"]
    = from b in _context.Book
        where b.AuthorId == id.Value
        select b;
```

注：
演者はクエリ構文よりメソッド構文の方が好みである（聞かれてない）。

```csharp
ViewData["Books"] = _context.Book.Where(b => b.AuthorId == id.Value)
```

Controllerに対応しないModelクラスのプロパティをViewで必要とするときは、`ViewData`や`ViewBag`を使ってViewに渡す。

注：とはいえ基本的にはModelクラス側にプロパティを持たせるべしなのだろう（`ViewData`や`ViewBag`はグローバル変数なので乱用禁物）。

## 6.4 一覧ページのページング機能

Webアプリケーションでは、一覧を表示するときに少し注意が必要である。
データ件数が数千件に及ぶときに、そのまま一覧を表示してしまうとサーバーからの応答が遅くなったり、
クライアントでデータを表示できない状態に陥ってしまう。
表示件数が多い場合、同時に問い合わせが集中するとサーバーが応答しなくなってしまう。
このような状況を防ぐために、Webアプリケーションでは、ある程度データ件数が多い場合はブラウザーで表示する件数を絞る。
これを「ページング機能」という。

### 6.4.1 ページング機能のない状態

図6-16 データ転送のプロセスの概要を参照。
単に表示するときに絞るのではなく、コントローラからDBへの問い合わせ（クエリ）を生成するときに条件を付ける。
そうすることによって、WebサーバーとDB間の負荷を減らすことができる。

### 6.4.2 Indexメソッドにページング機能を実装する

実装したものがリスト6-20である。

```csharp
public async Task<IActionResult> Index(int? page) // (1)
{
    if (page == null) // (2)
    {
        page = 0;
    }
    int max = 5; // (3)

    books = _context.Book // (4)
        .Skip(max * page.Value).Take(max)
        .Include(b => b.Author).Include(b => b.Publisher);

    if (page.Value > 0) // (5)
    {
        ViewData["prev"] = page.Value - 1;
    }
    if (books.Count() >= max) // (6)
    {
        ViewData["next"] = page.Value + 1;
        if (_context.Book.Skip(max * (page.Value + 1)).Take(max).Count() == 0) // (7)
        {
            ViewData["next"] = null;
        }
    }
    return View(await books.ToListAsync()); // (8)
}
```

- (1) `Index`メソッドに`int?`型の`page`引数を追加。
- (2) `page == null`のときは`page = 0`になるようにして最初のページが表示されるようにする。
- (3) 1ページに表示する最大要素数を設定。
- (4) `Skip`と`Take`を使って指定ページのデータのみをDBから取得。
- (5) 前のページがあるときはprevリンク用の変数`ViewData["prev"]`を設定。
- (6) (7) 次のページがあるときはnextリンク用の変数`ViewData["next"]`を設定。
- (8) 検索結果をIndexページ（View）に引き渡す

### 6.4.3 Indexページにページングのリンクを付ける

Books/Index.cshtmlも編集。

```cshtml

<div>
    @if (ViewBag.Prev != null) // (1)
    {
        <a asp-action="Index" asp-route-page="@ViewBag.Prev">prev</a> // (2)
    }
    else
    {
        <span>prev</span> // (3)
    }
    /
    @if (ViewBag.Next != null) // (4)
    {
        <a asp-action="Index" asp-route-page="@ViewBag.Next">next</a>
    }
    else
    {
        <span>next</span>
    }
</div>
```

- (1) `BooksController`側で`ViewData["prev"]`が設定されていれば、前ページのリンクを付ける。
- (2) `asp-action`, `asp-route-page`属性を使ってリンクを付ける。href属性が http://localhost/Books/Index?page=＜全ページ番号＞ になる。
- (3) `ViewData["prev"]`が設定されてなければ、リンクではない文字列としての「prev」だけを表示。
- (4) 次ページリンクも同様。

ここではprevとnextの2つのデータしかないが、付加的なデータが増えてきた場合は適宜クラスにまとめると管理がしやすくなる。

### 6.4.4 動作を確認する。

図6-17を参照。

## 6.5 一覧ページを検索で絞り込み

Indexページに項目の絞り込みの機能を付ける。
ブラウザーで書籍のタイトル（`Book.Title`）の一部を入力して、その結果が表示されるようにする。
検索結果はIndexページで利用しているtableタグをそのまま利用するが、
商品の写真などを並べて検索結果のレイアウトページ（Search.cshtmlなど）を作ることもある。

### 6.5.1 Indexページに検索用のテキストボックスを付ける

ブラウザーで絞り込み検索を受け付けるために、テキストボックスを追加する（リスト6-22）。
テキストボックスに署名に含まれる文字列を入力して、[Filter]ボタンをクリックして絞り込みを実行する。

```cshtml
<form asp-action="Index" method="get"> // (1)
    Title: <input type="text" name="search" value="@ViewBag.Search" /> // (2)
    <input type="submit" value="Filter" />
</form>
```

- (1) テキストボックスを使うのでformタグを使う。
- (2) 検索する文字列はinputタグで指定する。name属性に"search"と指定することで、`BookController`クラスの`Index`メソッドは`search`引数で受け取れるようになる。

ページングのリンクも以下のように修正する（リスト6-23）。

```cshtml

<div>
    @if (ViewBag.Prev != null )
    {
        <a asp-action="Index" asp-route-page="@ViewBag.Prev" asp-route-search="@ViewBag.Search" >prev</a>
    }
    else
    {
        <span>prev</span>
    }
    /
    @if (ViewBag.Next != null)
    {
        <a asp-action="Index" asp-route-page="@ViewBag.Next" asp-route-search="@ViewBag.Search">next</a>
    }
    else
    {
        <span>next</span>
    }
</div>
```

こうすると、ページング機能と絞り込み機能が同時に使えるようになる。

### 6.5.2 Indexメソッドに検索機能を追加する。

```csharp
public async Task<IActionResult> Index(int? page, string search) // (1)
{
    if (page == null)
    {
        page = 0;
    }
    int max = 5;

    var books = from m in _context.Book select m; // (2)
    if (!string.IsNullOrEmpty(search)) // (3
    {
        books = books.Where(b => b.Title.Contains(search)); //(4)
    }
    books = books // (5)
        .Skip(max * page.Value).Take(max)
        .Include(b => b.Author).Include(b => b.Publisher);

    if (page.Value > 0)
    {
        ViewData["prev"] = page.Value - 1;
    }
    if (books.Count() >= max)
    {
        ViewData["next"] = page.Value + 1;
        if (_context.Book.Skip(max * (page.Value + 1)).Take(max).Count() == 0)
        {
            ViewData["next"] = null;
        }
    }
    ViewData["search"] = search; // (6)
    return View(await books.ToListAsync()); // (7)
}
```

- (1) `Index`メソッドに`search`引数を追加する。
- (2) LINQを使ってクエリを構築する（実際にDBに問い合わせるのは(7)）。
- (3) `search`がnullや空でないかどうかで分岐。
- (4) `search`がnullや空でなければ、絞り込み条件をクエリに追加。
- (5) 従来のページング処理のクエリ。
- (6) 検索文字列を`ViewData`コレクションに設定。これによって再び同じ文字列で検索を実行できる。
- (7) クエリに従って、DBの検索が実行される。

### 6.5.3 動作を確認する。

図6-18、図6-19を参照。

