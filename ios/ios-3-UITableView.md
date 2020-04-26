#TableView


if you want to add a table view to UIViewController.

you need to `implement UITableViewDelegate and UITableViewDataSource`

you can see in define for UITableViewController.

``` swift
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
	
	@IBOutlet weak var tableView: UITableView!

	override func viewDidLoad() {
        super.viewDidLoad()
        
        self.tableView.dataSource = self
        // Do any additional setup after loading the view.
    }

    ...

    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 0 // your number of cells here
    }

    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
//        let cell = tableView.dequeueReusableCell(withIdentifier: "reuseIdentifier", for: indexPath)
        
            return UITableViewCell();
    }

    func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
        // cell selected code here
    }
}
```



## 直接建立tableview

```swift
class EditPasswordTableViewController: UITableViewController{
	override func viewDidLoad() {
        super.viewDidLoad()
    }
}
```

## Reload Data

重新加载数据:

```
self.tableView.reloadData()
```

### 在页面返回的时候, 加载数据

```
override func viewWillAppear(_ animated: Bool) {
    print("password detail view will appear")
    loadData()
    self.tableView.reloadData()
}
```