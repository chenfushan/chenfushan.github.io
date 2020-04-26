# Swift 语言

## switch

在swift中, switch 的case是自动break的.

```swift
let a : Int = 1
switch a {
	case 1:
		//xxxx
	case 2:
		//xxxx
	default:
		break;
}
```

如果想要继续往下执行需要增加`fallthrough`关键字.

```swift
let a : Int = 1
switch a {
	case 1:
		//xxxx
		fallthrough
	case 2:
		//xxxx
		fallthrough
	default:
		break;
}
```