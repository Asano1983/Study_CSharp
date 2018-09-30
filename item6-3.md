
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
EF Core 2.1�ł͒x���ǂݍ��݂���������Ă��邪�iEntity Framework 6�ł��g���邪�j�A�����ɂ͒��ӂ��K�v�ł��낤�B

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


