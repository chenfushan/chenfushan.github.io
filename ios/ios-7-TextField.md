# TextField

## 获取输入的文字

1. 关联当前`textField`的outlet
```swift
@IBOutlet weak var PassText: UITextField!
```

2. 然后直接获取当前text属性

```swift
let text : String = PassText.text ?? ""
```

## 隐藏输入内容

``` swift
PassText.isSecureTextEntry = true;
```


## 监听键盘的确认按键

1. 先把View的类继承`UITextFieldDelegate`

```swift
class LoginViewController: UIViewController, UITextFieldDelegate {
	override func viewDidLoad() {
        super.viewDidLoad()
        
 		//把当前的textField的代理指定到当前
        self.PassText.delegate = self;
    }
}
```

2. 新增方法实现
```swift
func textFieldShouldReturn(_ textField: UITextField) -> Bool {
	//收起键盘 不需要不加
    self.PassText.resignFirstResponder()
    //some other action
    return true;
}
```