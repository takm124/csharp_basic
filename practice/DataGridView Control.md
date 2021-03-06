# DataGridView Control

```markdown
개요

[폴더 - 파일] 형식으로 DataGridView 두 개를 컨트롤 하는 방법
```

## 기본 세팅

![](https://user-images.githubusercontent.com/72305146/144361679-20b31b1c-21ae-474b-bbff-5b1aace34992.png)

- 부모 테이블과 자식 테이블을 나누어 폴더-파일 형식의 데이터 컨트롤까지 해볼 예정

- 열은 미리 DataGridView에서 추가해 놓았음

- Parent
  
  - Add : 폴더 추가, Add시 자동으로 자식 건 임의로 하나 추가
  
  - Remove : 폴더 삭제, 자동으로 폴더에 속한 자식 데이터들도 함께 삭제
  
  - Merge : 병합, 두 개 이상의 폴더 선택후 병합 시 하나의 폴더로 합쳐짐

- Child
  
  - Add : 파일 추가
  
  - Remove : 파일 삭제
  
  - Split : 체크박스로 선택된 항목을 새로운 폴더를 만들어 삽입 시켜줌

## ParentGroup, Parent, Child Class 생성

- Parent DataGridView와 Child DataGridView에 삽입 할 클래스 생성

- ParentGroup은 Parent 객체들을 담기위한 그릇

### ParentGroup 클래스

```csharp
namespace DgvControl
{
    class ParentGroup
    {
        List<Parent> parent = new List<Parent>();
    }
}
```

### parent 클래스

```csharp
namespace DgvControl
{
    public class Parent
    {
        public bool isChecked { get; set; }
        public string content { get; set; }
        public List<Child> childList = new List<Child>();
    }
}
```

### child 클래스

```csharp
namespace DgvControl
{
    public class Child
    {
        public bool isChecked { get; set; }
        public string parentName { get; set; }
        public string content { get; set; }
    }
}
```

## Parent - Add 기능

- Add 클릭 시 프로세스 절차
  
  - Parent 객체를 보관할 ParentGroup 전역변수 생성
  
  - Parent 객체 생성 및 정보 주입
  
  - Child 객체 생성, 정보 주입 및 Parent 클래스의 childList에 추가하기
  
  - dgvParent, dgvChild 화면에 데이터 출력하기

### Add 기능을 위한 전역변수 추가

```csharp
// ParentGroup
ParentGroup _pg = new ParentGroup();

// 현재 선택된 Parent의 Index
int _prIdx;
```

### btnParentAdd_Click : Parent, Add 버튼 클릭 이벤트

```csharp
private void btnParentAdd_Click(object sender, EventArgs e)
{
    // Parent 객체 생성
    Parent pr = new Parent();
    pr.content = DateTime.Now.ToString("yyyyMMddHHmmss") + "에 생성된 Parent";

    // Child 객체 생성
    Child ch = new Child();
    ch.content = DateTime.Now.ToString("yyyyMMddHHmmss") + "에 생성된 Child";

    // Parent에 Child 추가
    pr.childList.Add(ch);

    // ParentGroup에 Parent 추가
    _pg.parent.Add(pr);

    // parent 객체 dgvParent에 출력
    fDisplayParent();

    // child 객체 dgcChild에 출력
    fDisplyChild();
}
```

### fDisplayParent : Parent 데이터 출력 함수

```csharp
/// <summary>
/// dgvParent에 Parent 객체 출력
/// </summary>
private void fDisplayParent()
{
    // No
    int count = 1;

    // dgvParent 초기화
    dgvParent.Rows.Clear();

    // Row 하나씩 추가
    foreach (Parent item in _pg.parent)
    {
        // Row 생성
        dgvParent.Rows.Add();

        // 값 지정
        dgvParent.Rows[dgvParent.RowCount - 1].Tag = item as Parent;
        dgvParent.Rows[dgvParent.RowCount - 1].Cells[0].Value = (count++).ToString();
        dgvParent.Rows[dgvParent.RowCount - 1].Cells[1].Value = false;
        dgvParent.Rows[dgvParent.RowCount - 1].Cells[2].Value = item.content;
    }
}
```

### fDisplayChild : Child 데이터 출력 함수

```csharp
/// <summary>
/// Child 객체 dgcChild에 출력
/// </summary>
private void fDisplyChild()
{
    // No
    int count = 1;

    //dgvChild 초기화
    dgvChild.Rows.Clear();

    // Row 추가
    foreach (Child item in _pg.parent[_prIdx].childList)
    {
        // Row 생성
        dgvChild.Rows.Add();

        // 값 지정
        dgvChild.Rows[dgvChild.RowCount - 1].Cells[0].Value = (count++).ToString();
        dgvChild.Rows[dgvChild.RowCount - 1].Cells[1].Value = false;
        dgvChild.Rows[dgvChild.RowCount - 1].Cells[2].Value = (_prIdx + 1).ToString();
        dgvChild.Rows[dgvChild.RowCount - 1].Cells[3].Value = item.content;
    }
}
```

### dgvParent_SelectionChanged : dgvParent Row 선택 변경 시 발생하는 이벤트

```csharp
/// <summary>
/// Parent 데이터 클릭 시 Child 데이터 출력, 현재 선택된 Parent 데이터의 인덱스 저장
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void dgvParent_SelectionChanged(object sender, EventArgs e)
{
    if (dgvParent.SelectedRows.Count > 0 && dgvParent.SelectedRows[0].Tag != null)
    {
        _prIdx = dgvParent.SelectedRows[0].Index;
        fDisplyChild();
    }
}
}
```

- datagridview의 cellclick 이나 기타 click 이벤트를 사용하지 않고 SelectionChange 이벤트를 선택한 이유
  
  - 추후에 checkbox 담당 이벤트를 cellclick으로 발생시키기 위해

### 결과 (Parent의 Add 버튼 클릭 시)

![parent-add](https://user-images.githubusercontent.com/72305146/144365785-9a438d17-80bc-40a8-acb6-b2f6f47a05f9.png)

## Child - Add 기능

- Child의 Add 버튼 클릭 시 프로세스
  
  - 새로운 Child 객체 생성
  
  - 현재 선택된 Parent 객체 호출
  
  - Child객체 Parent 객체에 Add
  
  - 화면 재출력

### btnChildAdd_Click

```csharp
private void btnChildAdd_Click(object sender, EventArgs e)
{
    if (dgvParent.RowCount < 1) return;

    // 새로운 Child 객체 생성
    Child ch = new Child();
    ch.content = DateTime.Now.ToString("yyyyMMddHHmmss") + "에 추가로 생성된 Child";

    // Parent 객체 호출
    Parent pr = _pg.parent[_prIdx];

    // Child Add 후 화면 재호출
    pr.childList.Add(ch);
    fDisplyChild();
}yChild();
}
```

### 결과

![child-add result](https://user-images.githubusercontent.com/72305146/144775003-7e02537b-0380-4b2e-92dd-dfc75249ccc1.png)

## Parent-Remove 기능

- Parent의 Remove 버튼 클릭시 프로세스
  
  - 현재 Parent에서 체크박스 체크된 Parent 객체 확인
  
  - ParentGroup에서 삭제

### DataGridView 체크박스 작업

- CellClick 이벤트 발생시 Cell 위치값으로 checkbox 설정

- 체크된 Parent의 개수를 count하기 위한 전역변수 _prChecked 추가_



```csharp
/// <summary>
/// Parent 체크박스 관련 이벤트
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void dgvParent_CellClick(object sender, DataGridViewCellEventArgs e)
{
    if (e.ColumnIndex == 1 && e.RowIndex != -1)
    {
        if (dgvParent.Rows[e.RowIndex].Cells[e.ColumnIndex].Value.ToString().ToUpper() == "TRUE")
        {
            dgvParent.Rows[e.RowIndex].Cells[e.ColumnIndex].Value = false;
            Parent pr = dgvParent.Rows[e.RowIndex].Tag as Parent;
            pr.isChecked = false;
            _prChecked--;
        }
        else
        {
            dgvParent.Rows[e.RowIndex].Cells[e.ColumnIndex].Value = true;
            Parent pr = dgvParent.Rows[e.RowIndex].Tag as Parent;
            pr.isChecked = true;
            _prChecked++;
        }
    }
}
```



### btnParentRemove_Click

```csharp
/// <summary>
/// 부모 객체 삭제
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void btnParentRemove_Click(object sender, EventArgs e)
{
    // 체크되어 있는 Parent가 있는지 확인
    if (_prChecked < 1) return;

    try
    {
        for (int i = _pg.parent.Count - 1; i >= 0; i--)
        {
            if (_pg.parent[i].isChecked)
            {
                _pg.parent.RemoveAt(i);
            }
        }

        // 선택된 인덱스 초기화
        _prIdx = 0;

        // 화면 재출력
        fDisplayParent();
        fDisplyChild();
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.Message);
    }
}
```

 



## Child - Remove 기능

- 골자는 Parent의 Remove와 동일하다.



### 체크박스 작업

- Parent때와 마찬가지로 체크박스 개수를 count하기 위한 전역변수 _chChecked 생성_

```csharp
/// <summary>
/// Child 체크박스 관련 이벤트
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void dgvChild_CellClick(object sender, DataGridViewCellEventArgs e)
{
    if (e.ColumnIndex == 1 && e.RowIndex != -1)
    {
        if (dgvChild.Rows[e.RowIndex].Cells[e.ColumnIndex].Value.ToString().ToUpper() == "TRUE")
        {
            dgvChild.Rows[e.RowIndex].Cells[e.ColumnIndex].Value = false;
            Child ch = dgvChild.Rows[e.RowIndex].Tag as Child;
            ch.isChecked = false;
            _chChecked--;
        }
        else
        {
            dgvChild.Rows[e.RowIndex].Cells[e.ColumnIndex].Value = true;
            Child ch = dgvChild.Rows[e.RowIndex].Tag as Child;
            ch.isChecked = true;
            _chChecked++;
        }
    }
}
```



### btnChildRemove_Click

```csharp
/// <summary>
/// 자식 객체 삭제
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void btnChildRemove_Click(object sender, EventArgs e)
{
    // 체크되어 있는 Child가 있는지 확인
    if (_chChecked < 1) return;

    try
    {
        for (int i = _pg.parent[_prIdx].childList.Count - 1; i >= 0; i--)
        {
            if (_pg.parent[_prIdx].childList[i].isChecked)
            {
                _pg.parent[_prIdx].childList.RemoveAt(i);
            }
        }

        // 화면 재출력
        fDisplayParent();
        fDisplyChild();
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.Message);
    }
}
```
