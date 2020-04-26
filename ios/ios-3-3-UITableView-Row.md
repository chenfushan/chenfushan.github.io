# UITableView

## 插入一行

```swift
//cell数组
let newPasswordExtra = PasswordExtra()
//新增一个元素 这个新增的元素用于新增cell的数据使用
self.passwordExtra.append(newPasswordExtra)
//extraCount这个是行数,现在要新增一行
self.extraCount = self.passwordExtra.count

//开始更新
self.tableView.beginUpdates()
//这里要新建一个IndexPath 这个indexPath指向要添加的位置前面的一个位置
//self.extraCount-1因为已经加了一个,所以要-1 来表示未添加前最后一个元素位置
//self.extraSection表示如果是多个section的话,要再哪个section添加
self.tableView.insertRows(at: [IndexPath(row: self.extraCount-1, section: self.extraSection)], with: .automatic)
self.tableView.endUpdates()
```

## 删除一行

```swift
//要删除的元素, at: 数组的下标位置
self.passwordExtra.remove(at: indexPath.row)
//行数更新
self.extraCount = self.passwordExtra.count
//开始更新
self.tableView.beginUpdates()
//删除一行, indexPath指向要删除的哪一行的位置
self.tableView.deleteRows(at: [indexPath], with: .automatic)
self.tableView.endUpdates()
```

## 选择一行
```swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    print("select table view cell")
    print(indexPath)
}
```

## 取消选择一行
```swift
override func tableView(_ tableView: UITableView, didDeselectRowAt indexPath: IndexPath) {
    print("deselect table view cell")
    print(indexPath)
}
```