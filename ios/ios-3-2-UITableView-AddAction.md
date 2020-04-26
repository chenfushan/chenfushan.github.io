# 对UITableView增加滑动操作

## IOS 13之前

```swift
override func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath) -> [UITableViewRowAction]? {

//        // action one
//        let editAction = UITableViewRowAction(style: .default, title: "Edit", handler: { (action, indexPath) in
//            print("Edit tapped")
//        })
//        editAction.backgroundColor = UIColor.blue

	//对不同的section增加
    let section = indexPath.section
    print("add action for section:\(section)")
    if section != self.extraSection {
        return []
    }
    
    // action two
    let deleteAction = UITableViewRowAction(style: .destructive, title: "Delete", handler: { (action, indexPath) in
        print("Delete tapped")
    })
    deleteAction.backgroundColor = UIColor.red

    return [deleteAction]
}
```

## IOS 13之后

在IOS13之后,上面的方法已经不再被推荐使用了. 主要是推荐使用下面的方法.

```swift
override func tableView(_ tableView: UITableView, trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
        let deleteAction = UIContextualAction(style: .destructive, title: "删除", handler: {(contextualAction, view, boolValue) in
            self.passwordList.remove(at: indexPath.row)
            
            let info = self.appData.initData!
            if let tempGroupIndex = info.firstIndex(where: {$0.groupId == self.groupId!}){
                self.appData.initData![tempGroupIndex].passwordList = self.passwordList
            }
            self.tableView.beginUpdates()
            self.tableView.deleteRows(at: [indexPath], with: .automatic)
            self.tableView.endUpdates()
        })
        let swipeAction = UISwipeActionsConfiguration(actions: [deleteAction])
        return swipeAction
    }
```

上面是从右向左滑动,增加尾部的动作.

----

```swift
override func tableView(_ tableView: UITableView, leadingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
        //code
    }
```
上面是从左向右滑动,增加头部的动作.