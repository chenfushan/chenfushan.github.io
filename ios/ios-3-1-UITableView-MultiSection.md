# UITableView

## 多section table view实现

### 实现多section

```swift
class EditPasswordTableViewController: UITableViewController{
	
	let headSection : Int = 0;
    let detailSection : Int = 1;
    let extraSection : Int = 2;
    let addSection : Int = 3;

    /**
     * 有几个section
     */
    override func numberOfSections(in tableView: UITableView) -> Int {
        // #warning Incomplete implementation, return the number of sections
        return 4
    }
}

```

### 每个section多个row

```swift
/**
 * 每个section有多少row
 */
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    // #warning Incomplete implementation, return the number of rows
    switch section {
    case headSection:
        return 1;
    case detailSection:
        return 4;
    case extraSection:
        return self.extraCount;
    case addSection:
        return 1;
    default:
        return 0;
    }
}
```

### 定义不同section每个row的高度

```swift
/**
 * 每个sectione的row的高度
 */
override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
    let section = indexPath.section
    switch section {
    case headSection:
        return CGFloat(88)
    case detailSection:
        return CGFloat(70)
    case extraSection:
        return CGFloat(70)
    case addSection:
        return CGFloat(60);
    default:
        return CGFloat(60)
    }
}
```

### 定义不同section的标题
```swfit
override func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
    return "Section \(section)"
}
```

如果需要单个section里面的row高度不同的话, 通过`indexPath`来返回不同高度.