# 手势识别

## 点击事件

有些Button按钮是自带点击事件的, 可以直接关联Action. 但是有些组件,例如图片组件, 文字组件是没有点击事件的, 就需要手动添加. 同时需要在点击的时候传递需要的一些数据.

```swift
<!-- 定义自定义的点击事件类 放上自己想要的数据 -->
import UIKit

class avatarTap: UITapGestureRecognizer {
    var avatarName : String?
}
```

```swift
<!-- 有一张图片 -->
let image = UIImage()
<!-- 允许交互 -->
image.isUserInteractionEnabled = true;
<!-- 添加手势识别 -->
let tap = avatarTap(target: self, action: #selector(tapAvatar(sender:)))
tap.avatarName = name;
image.addGestureRecognizer(tap)

<!-- 接收点击事件 -->
@objc func tapAvatar(sender: avatarTap){
    print(sender.avatarName ?? "empty")
}
```