
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

詳細ページの著者名、出版社名についてもクリックすると著者の詳細ページ、出版社の詳細ぺーじにジャンプするようにする。
リスト6-14の(1)、(2)のようにViews/Books/Details.cshtmlを編集する。

```cshtml
<a asp-controller="Authors" asp-action="Details" asp-route-id="@Model.AuthorId"> // (1)
    @Html.DisplayFor(model => model.Author.Name)
</a>
```

`asp-controller`属性を指定すると、他のコントローラーActionメソッドを呼び出すことができる。

注：一見、著者のIDはModel.Author.Idで取得できるので、BookクラスがAuthorIdを持つのは冗長な気がしたのですが、
https://docs.microsoft.com/ja-jp/ef/core/modeling/relationships によると
「依存エンティティ クラスで定義されている外部キー プロパティを持つことをお勧めしますが、これは必要ありません。」
らしいです。
Authorはincludeしないと取得しないので（AuthorIdは常に取得するので）、安全な気もします。
また、DB上はAuthorId列があるので、あった方が分かりやすいのかも。

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
（Includeメソッドを使って、AuthorをDBから取得するときにBookも取得する）。

```csharp
var author = await _context.Author
    .include(a => a.Prefecture)
    .include(a => a.Book) // (3)
    .SingleOrDefaultAsync(m => m.Id == id);
```

注：includeメソッドを使うと、Author取得時にBook（ナビゲーションプロパティ）の内容もDBから取得するようなクエリ（SQL文）が発行される。
この例の場合、ループの外側で（コントローラー側で）Bookの内容が取得できていることに注意（ループの内側で取得すると効率が悪い）。
詳しくは
https://docs.microsoft.com/ja-jp/ef/core/querying/related-data
を参照。
EF Core 2.1では遅延読み込みが導入されているが（Entity Framework 6でも使えるが）、効率には注意が必要であろう。

### 6.3.5 出版社ページのレイアウトを変更

著者ページと同様に修正する（略）。

（includeを使う代わりに）ControllerクラスのDetailメソッドでLINQを使って作成することもできる。

```csharp
ViewData["Books"]
    = from b in _context.Book
        where b.AuthorId == id.Value
        select b;
```

Controllerに対応しないModelクラスのプロパティをViewで必要とするときは、ViewDataやViewBagを使ってViewに渡す。

注：とはいえ基本的にはModelクラス側にプロパティを持たせるべしなのだろう（ViewDataやViewBagはグローバル変数なので乱用禁物）。

## 6.4 一覧ページのページング機能


