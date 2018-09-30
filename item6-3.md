
## 6.3 �ڍ׃y�[�W�̊g��

### 6.3.1 �ڍ׃y�[�W�փh�����_�E��

�ȉ��̂悤�ȃW�����v���ł���悤�ɂ������B

- ���Ђ̃��X�g�����Ђ̏ڍ׃y�[�W�����҃y�[�W�A�o�ŎЃy�[�W
- ���҃y�[�W�i���M�������Ѓ��X�g�����j���e���Ђ̃��X�g
- �o�ŎЃy�[�W�i�o�ł������Ѓ��X�g�����j���e���Ђ̃��X�g

### 6.3.2 �ꗗ�y�[�W�Ƀ����N��ǉ�

�ꗗ�y�[�W�̏������N���b�N����Əڍ׃y�[�W�ɃW�����v����悤�ɂ���B
���X�g6-13��(1)�̂悤��Views/Books/Index.cshtml��ҏW����
�i���X��`@Html.DisplayFor(modelItem => item.Title)`�����������Ƃ�������̂悤�ɕҏW����j�B

```cshtml
<a asp-action="Details" asp-route-id="@item.Id"> // (1)
    @Html.DisplayFor(modelItem => item.Title)
</a>
```

�}6-11�̂悤�ɏڍ׃y�[�W�ւ̃����N���ǉ������B

### 6.3.3 �ڍ׃y�[�W�̃��C�A�E�g�ύX

�ڍ׃y�[�W�̒��Җ��A�o�ŎЖ��ɂ��Ă��N���b�N����ƒ��҂̏ڍ׃y�[�W�A�o�ŎЂ̏ڍׂ؁[���ɃW�����v����悤�ɂ���B
���X�g6-14��(1)�A(2)�̂悤��Views/Books/Details.cshtml��ҏW����B

```cshtml
<a asp-controller="Authors" asp-action="Details" asp-route-id="@Model.AuthorId"> // (1)
    @Html.DisplayFor(model => model.Author.Name)
</a>
```

`asp-controller`�������w�肷��ƁA���̃R���g���[���[Action���\�b�h���Ăяo�����Ƃ��ł���B

���F�ꌩ�A���҂�ID��Model.Author.Id�Ŏ擾�ł���̂ŁABook�N���X��AuthorId�����̂͏璷�ȋC�������̂ł����A
https://docs.microsoft.com/ja-jp/ef/core/modeling/relationships �ɂ���
�u�ˑ��G���e�B�e�B �N���X�Œ�`����Ă���O���L�[ �v���p�e�B�������Ƃ������߂��܂����A����͕K�v����܂���B�v
�炵���ł��B
Author��include���Ȃ��Ǝ擾���Ȃ��̂ŁiAuthorId�͏�Ɏ擾����̂Łj�A���S�ȋC�����܂��B
�܂��ADB���AuthorId�񂪂���̂ŁA����������������₷���̂����B

### 6.3.4 ���Ѓy�[�W�̃��C�A�E�g�ύX

���҂̏ڍ׃y�[�W�Ɏ��M�������Ѓ��X�g��\������悤�ɂ���B
���X�g6-15��(1)�A(2)�̂悤��Views/Authors/Details.cshtml��ҏW����
�i`@foreach`���g���ĉӏ��������X�g����邾���j�B

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

����Ƀ��X�g6-16��(3)�̂悤��Controllers/AuthorsController.cs��ҏW����B
�iInclude���\�b�h���g���āAAuthor��DB����擾����Ƃ���Book���擾����j�B

```csharp
var author = await _context.Author
    .include(a => a.Prefecture)
    .include(a => a.Book) // (3)
    .SingleOrDefaultAsync(m => m.Id == id);
```

���Finclude���\�b�h���g���ƁAAuthor�擾����Book�i�i�r�Q�[�V�����v���p�e�B�j�̓��e��DB����擾����悤�ȃN�G���iSQL���j�����s�����B
���̗�̏ꍇ�A���[�v�̊O���Łi�R���g���[���[���ŁjBook�̓��e���擾�ł��Ă��邱�Ƃɒ��Ӂi���[�v�̓����Ŏ擾����ƌ����������j�B
�ڂ�����
https://docs.microsoft.com/ja-jp/ef/core/querying/related-data
���Q�ƁB
EF Core 2.1�ł͒x���ǂݍ��݂���������Ă��邪�iEntity Framework�Ȃ�4�ł��g���邪�j�A�����ɂ͒��ӂ��K�v�ł��낤�B

### 6.3.5 �o�ŎЃy�[�W�̃��C�A�E�g��ύX

���҃y�[�W�Ɠ��l�ɏC������i���j�B

�iinclude���g������ɁjController�N���X��Detail���\�b�h��LINQ���g���č쐬���邱�Ƃ��ł���B

```csharp
ViewData["Books"]
    = from b in _context.Book
        where b.AuthorId == id.Value
        select b;
```

Controller�ɑΉ����Ȃ�Model�N���X�̃v���p�e�B��View�ŕK�v�Ƃ���Ƃ��́AViewData��ViewBag���g����View�ɓn���B

���F�Ƃ͂�����{�I�ɂ�Model�N���X���Ƀv���p�e�B����������ׂ��Ȃ̂��낤�iViewData��ViewBag�̓O���[�o���ϐ��Ȃ̂ŗ��p�֕��j�B

## 6.4 �ꗗ�y�[�W�̃y�[�W���O�@�\

Web�A�v���P�[�V�����ł́A�ꗗ��\������Ƃ��ɏ������ӂ��K�v�ł���B
�f�[�^���������猏�ɋy�ԂƂ��ɁA���̂܂܈ꗗ��\�����Ă��܂��ƃT�[�o�[����̉������x���Ȃ�����A
�N���C�A���g�Ńf�[�^��\���ł��Ȃ���ԂɊׂ��Ă��܂��B
���̂悤�ȏ󋵂�h�����߂ɁAWeb�A�v���P�[�V�����ł́A������x�f�[�^�����������ꍇ�̓u���E�U�[�ŕ\�����錏�����i��B
������u�y�[�W���O�@�\�v�Ƃ����B

### 6.4.1 �y�[�W���O�@�\�̂Ȃ����

�}6-16 �f�[�^�]���̃v���Z�X�̊T�v���Q�ƁB
�ǂ��Ō������i��̂����l���悤�B

### 6.4.2 Index���\�b�h�Ƀy�[�W���O�@�\����������

�����������̂����X�g6-20�ł���B

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
    ViewData["search"] = search;
    return View(await books.ToListAsync()); // (8)
}
```

- (1) Index���\�b�h��int?�^��page������ǉ��B
- (2) `page == null`�̂Ƃ���`page = 0`�ɂȂ�悤�ɂ��čŏ��̃y�[�W���\�������悤�ɂ���B
- (3) 1�y�[�W�ɕ\������ő�v�f����ݒ�B
- (4) `Skip`��`Take`���g���Ďw��y�[�W�̃f�[�^�݂̂�DB����擾�B
- (5) �O�̃y�[�W������Ƃ���prev�����N�p�̕ϐ�`ViewData["prev"]`��ݒ�B
- (6) (7) ���̃y�[�W������Ƃ���next�����N�p�̕ϐ�`ViewData["next"]`��ݒ�B
- (8) �������ʂ�Index�y�[�W�iView�j�Ɉ����n��

### 6.4.3 Index�y�[�W�Ƀy�[�W���O�̃����N��t����

Books/Index.cshtml���ҏW�B

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

- (1) BooksController����`ViewData["prev"]`���ݒ肳��Ă���΁A�O�y�[�W�̃����N��t����B
- (2) asp-action, asp-route-page�������g���ă����N��t����Bhref������ http://localhost/Books/Index?page=���S�y�[�W�ԍ��� �ɂȂ�B
- (3) `ViewData["prev"]`���ݒ肳��ĂȂ���΁A�����N�ł͂Ȃ�������Ƃ��Ắuprev�v������\���B
- (4) ���y�[�W�����N�����l�B

### 6.4.4 ������m�F����B

�}6-17���Q�ƁB

## 6.5 �ꗗ�y�[�W�������ōi�荞��

Index�y�[�W�ɍ��ڂ̍i�荞�݂̋@�\��t����B
�u���E�U�[�ŏ��Ђ̃^�C�g���iBook.Title�j�̈ꕔ����͂��āA���̌��ʂ��\�������悤�ɂ���B
�������ʂ�Index�y�[�W�ŗ��p���Ă���table�^�O�����̂܂ܗ��p���邪�A
���i�̎ʐ^�Ȃǂ���ׂČ������ʂ̃��C�A�E�g�y�[�W�iSearch.cshtml�Ȃǁj����邱�Ƃ�����B

### 6.5.1 Index�y�[�W�Ɍ����p�̃e�L�X�g�{�b�N�X��t����

�u���E�U�[�ōi�荞�݌������󂯕t���邽�߂ɁA�e�L�X�g�{�b�N�X��ǉ�����i���X�g6-22�j�B
�e�L�X�g�{�b�N�X�ɏ����Ɋ܂܂�镶�������͂��āA[Filter]�{�^�����N���b�N���či�荞�݂����s����B

```cshtml
<form asp-action="Index" method="get">
    Title: <input type="text" name="search" value="@ViewBag.Search" />
    <input type="submit" value="Filter" />
</form>
```

- (1) �e�L�X�g�{�b�N�X���g���̂�form�^�O���g���B
- (2) �������镶�����input�^�O�Ŏw�肷��Bname������"search"�Ǝw�肷�邱�ƂŁABookController�N���X��Index���\�b�h��search�����Ŏ󂯎���悤�ɂȂ�B

�y�[�W���O�̃����N���ȉ��̂悤�ɏC������i���X�g6-23�j�B

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

��������ƁA�y�[�W���O�@�\�ƍi�荞�݋@�\�������Ɏg����悤�ɂȂ�B

### 6.5.2 Index���\�b�h�Ɍ����@�\��ǉ�����B

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

- (1) Index���\�b�h��search������ǉ�����B
- (2) LINQ���g���ăN�G�����\�z����i���ۂ�DB�ɖ₢���킹��̂�(7)�j�B
- (3) search��null���łȂ����ǂ����ŕ���B
- (4) search��null���łȂ���΁A�i�荞�ݏ������N�G���ɒǉ��B
- (5) �]���̃y�[�W���O�����̃N�G���B
- (6) �����������ViewData�R���N�V�����ɐݒ�B����ɂ���čĂѓ���������Ō��������s�ł���B
- (7) �N�G���ɏ]���āADB�̌��������s�����B

### 6.5.3 ������m�F����B

�}6-18�A�}6-19���Q�ƁB

