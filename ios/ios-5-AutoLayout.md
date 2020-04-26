# Border radius

## Normal border radius

```swift
 @IBOutlet weak var ConfirmButon: UIButton!

 ConfirmButon.layer.cornerRadius = 10; //unit : pt
 ConfirmButon.clipsToBounds = true; //make radius work
```

## Circle border radius

1. make the component width equals the height.

> Add New Constraints -> Aspect ration ( 1:1 )

2. cornerRadius = half width

```swift
 @IBOutlet weak var ConfirmButon: UIButton!

 ConfirmButon.layer.cornerRadius = ConfirmButon.frame.size.width/2;
 ConfirmButon.clipsToBounds = true; //make radius work
```

### problems

> note : https://stackoverflow.com/questions/32362934/how-to-keep-a-round-imageview-round-using-auto-layout/43998774

if you are using auto layout. like : width equals 1/3 safe area's width.
then you can not make a circle radius.

```swift
// you need to write the radius code in the viewWillLayoutSubviews

override func viewWillLayoutSubviews() {
  super.viewWillLayoutSubviews()
  profileImageView.layer.cornerRadius = profileImageView.frame.height / 2.0
}
```

or 

```swift

class Foo: UIImageView {
    override func layoutSubviews() {
        super.layoutSubviews()

        let radius: CGFloat = self.bounds.size.width / 2.0

        self.layer.cornerRadius = radius
    }
}
```

