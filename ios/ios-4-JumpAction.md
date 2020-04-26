# Jump Action

## 跳转的种类

- show  
在NavigationController存在的情况下，segue连接的Controller会被压入导航栈。新压入的视图控制器有返回按钮，单击可以返回.

- show Detail  
这种类型是不压栈的，不管有没有NavigationController，它只是replace取代了当前的视图，不提供返回按钮。在detail area中展现内容。例如：即使app同时显示master和detail视图，那么内容将被压入detail区域如果app当前仅显示Master或者detail视图，那么内容将替换当前视图控制器堆栈中的顶层视图。

- Present Modally  
这种类型是不压栈的，以模态的方式显示，类似于弹出的警告窗口、登陆框一类的视图；用户无法与上一个视图交互，除非关闭当前视图。

如果想要以`form sheet` 方式展示页面, 第一: 设置segue方式为 `Present Modally`. 第二: 设置view的presentation方式为`form sheet`.

- Present As Popover  
这种类型不压栈，类似于下拉菜单；在iPad中，目标视图以浮动窗样式呈现，点击目标视图以外区域，目标视图消失；在iPhone中，默认目标视图以模态覆盖整个屏幕。

## Jump action by interface

1. connect the jump action by interface
2. write relation segue code.

```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        // Get the new view controller using segue.destination.
        // Pass the selected object to the new view controller.
        
        print(segue.identifier ?? "nil")
    }

<!-- 这里如果返回false 就不会进行跳转 -->
override func shouldPerformSegue(withIdentifier identifier: String, sender: Any?) -> Bool {
	// if you want to prevent the jump action : return false.
        return self.loginResult
    }
```

## Jump action by code in viewController

```swift

@IBAction func ClickLogin(_ sender: UIButton) {
        jumpToHome()
    }
    
    func jumpToHome(){
    	<!-- 如果这里是多个story board -->
        let storyBoard = UIStoryboard(name: "Main", bundle: nil)
        <!-- 也可以直接 self.storyboard -->
        let homeView = storyBoard.instantiateViewController(withIdentifier: "HomeNavigator") as! HomeTableViewController
        self.showDetailViewController(homeView, sender: self)
    }
```

## Jump action by code in navigationController

```siwft
//Main is the storyboard id
let storyBoard = UIStoryboard(name: "Main", bundle: nil)
let editView = storyBoard.instantiateViewController(withIdentifier: "EditPasswordTableViewController") as! EditPasswordTableViewController
//set attribute
editView.groupId = self.groupId
editView.editType = EditType.ADD
//push view
self.navigationController?.pushViewController(editView, animated: true)
```