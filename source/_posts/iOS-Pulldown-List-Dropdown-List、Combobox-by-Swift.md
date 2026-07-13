title: iOS Pulldown List(Dropdown List、Combobox) by Swift
date: 2015-07-17 19:30:46
categories:
- Tech
- iOS
- UI
tags:
- iOS
- Swift
- Objective-C
---

最近因為工作的關係開始接觸 iOS SDK，Apple 官方文件算是頗齊全的，但相較於 Android 的手冊，還是略遜一疇（Google 大神還是比較有善～）。而本次目標是在 iOS 裡面新增一個簡單下拉式選單（Dropdown Menu、Pulldown Menu），憑著之前新增元件的經驗，我開始在 Xcode Object Library 裡搜尋下拉選單，但不管我打 Dropdown、Pulldown，還是 Combobox 就是沒有！！！一向帶給人有善印象的 Apple 咬一口，居然沒有提供下拉選單這種元件，不會要自己刻吧，崩潰。。。

還好現在第三方社群很發達，大家都很 Nice！拜了幾輪 Google 大神後，大神給了指示，原來在 iOS 裡下拉選單需要用 [UITextField](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITextField_Class) 這個元件來改造，然後再加上一些其他元件完成選單的內容。選單的製作 Google 大神給了我兩個答案，一個是透過修改 UITextField InputView，另一個是使用 [ActionSheetPicker-3.0](https://github.com/skywinder/ActionSheetPicker-3.0)，以下將一一介紹：

### 方法一：修改 UITextField InputView（[Ref.](https://github.com/Darkseal/DownPicker))
1. 首先在目標 ViewController 裡面新增幾個成員變數。
```Swift
class ViewController: UIViewController {
        //UITextField 的 Reference Outlet 變數
        @IBOutlet weak var textField: UITextField!

        //選單元件
        private var pickerView: UIPickerView!;

        //下拉選單的內容
        private var dataArray: NSArray!;
        。。。。。
}
```
記得到 Storyboard 裡將Reference Outlet 變數與元件建立連結。

2. 在 ViewController 的 viewDidLoad() 裡改造 UITextField，現在先將它看起來像下拉選單（新增下拉符號）。
```Swift
override func viewDidLoad() {
        。。。。。
        //指定下拉符號圖片
        var img = UIImage(named: "dwn")
        self.textField.rightView = UIImageView(image: img)

        //設定圖片填滿右視圖
        self.textField.rightView!.contentMode = UIViewContentMode.ScaleAspectFit
        self.textField.rightView!.clipsToBounds = true

        //顯示下拉符號
        self.textField.rightViewMode = UITextFieldViewMode.Always

        //建立委托
        self.textField.delegate = self
        。。。。
}
```

3. 再來在 ViewController 裡新增兩個 function ，一個用來建立選單元件（使用 UIPickerView 及 Toolbar），一個用來告知 iOS 使用者已經完成選擇。
```Swift
//新增選單元件
private func showPicker() {
      //使用 UIPickerView 做為選擇器
      self.pickerView = UIPickerView()

      //顯示目前選擇指標
      self.pickerView.showSelectionIndicator = true

      //指定資料來源與委托
      self.pickerView.dataSource = self
      self.pickerView.delegate = self

      //使用 UIToolbar 建立選單元件與外觀
      var toolbar = UIToolbar()
      toolbar.barStyle = UIBarStyle.Default
      toolbar.sizeToFit()
      
      //新增完成選擇（Done）按鈕
      var doneButton = UIBarButtonItem(title: "Done", style: UIBarButtonItemStyle.Done, target: self, action: "doneClicked")

      //讓按鈕靠右對齊的元件
      var flexibleSpace = UIBarButtonItem(barButtonSystemItem: UIBarButtonSystemItem.FlexibleSpace, target: nil, action: nil)

      //將元件放到 toolbar 裡面
      toolbar.setItems([flexibleSpace, doneButton], animated: true)
}

//使用者點擊成選擇（Done）按鈕後的動作
func doneClicked() {
      //隱藏選單
      self.textField.resignFirstResponder()
}
```

4. 最後建立使用者與 UITextField 和 UIPickerView 的互動事件，以及設定 UIPickerView 的選項資料。
```Swift
class ViewController: UIViewController, UITextFieldDelegate, UIPickerViewDataSource, UIPickerViewDelegate {
        。。。。。
        //點擊 textField 的時候出現選單
        func textFieldShouldBeginEditing(textField: UITextField) -> Bool {
              //沒有資料的時候不顯示選單
              if self.dataArray != nil && self.dataArray.count > 0 {
                    self.showPicker(textField)
                    return true
              }
              return false
        }

        //不讓使用者透過建盤輸入文字
        func textField(textField: UITextField, shouldChangedCharactersInRange range: NSRange, replacementString string: String) {
              return false
        }

        //設定選單項目的列數，例如時間 (小時：分鐘) 的 pickerView 就是 return 2
        func numberOfComponentInPickerView(pickerView: UIPickerView) -> Int {
              return 1
        }

        //設定選單選項的數量
        func pickerView(pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
              return self.dataArray.count
        }

        //設定選單項目
        func pickerView(pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String! {
              return self.dataArray[row] as! String
        }

        //當選單被滑動（Sliding）的時候，改變 textField 的值
        func pickerView(pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
              self.textField.text = self.dataArray[row] as! String
        }
}
```

### 方法二：使用 ActionSheetPicker-3.0
[ActionSheetPicker-3.0](https://github.com/skywinder/ActionSheetPicker-3.0) 這個元件是 [Petr Korolev](https://github.com/skywinder) 這個好心人用 Objective-C 寫的，但我的專案目標語言是 Swift，所以要先設定套件關聯（Bridge），Petr Korolev 提供了三個方法安裝使用這個元件（CocoaPods、Carhage 和 Git Submodule），但可能我太好運了 CocoaPods 和 Carhage 的 Bridge 一直設定不起來，我只好用 Git Submodule，以下是我的作法：

1. 設定 Git Submodule
```Bash
# 打開 Terminal，然後切換目錄到目標專案的根目錄，最後打上這一行
$ git submodule add https://github.com/skywinder/ActionSheetPicker-3.0.git
```

2. 用 Finder 打開專案目錄底下的 ActionSheetPicker-3.0 資料夾，然後將 /CoreActionSheetPicker/CoreActionSheetPicker.xcodeproj 拖到 Xcode 的 Project Navigator 裡面。
<div style="width:25%;margin:0 auto;display:block;">

![ProjectSetting](images/Dropdwn.png)

</div>

3. 用滑數點擊 Xcode Project Navigator 裡最上面的專案圖案（藍色的 Icon），右邊跳出專案設定，如下圖：
<div style="width:70%;margin:0 auto;display:block;">

![ProjectSetting](images/Dropdwn2.png)

</div>

4. 切換專案設定的標籤到 Build Phases，在 Target Dependancy 裡面新增 CoreActionSheetPicker.framework。
<div style="width:65%;margin:0 auto;display:block;">

![ProjectSetting](images/Dropdwn3.png)

</div>

5. 一樣在 Build Phases 裡，新增一個 New Copy Files Phase，將其重新命名為 Copy Frameworks。
<div style="width:35%;margin:0 auto;display:block;">

![ProjectSetting](images/Dropdwn4.png)

</div>
<div style="width:35%;margin:0 auto;display:block;">

![ProjectSetting](images/Dropdwn5.png)

</div>

6. 在 Copy Frameworks 裡，選擇 Destination 為 Frameworks，然後新增 CoreActionSheetPicker.framework。
<div style="width:65%;margin:0 auto;display:block;">

![ProjectSetting](images/Dropdwn6.png)

</div>

7. 隨便新增一個 Objective-C 檔案到專案裡面，然後 Xcode 會詢問是不是要新增 Objective-C Bridge Header，請按「是」。
<div style="width:60%;margin:0 auto;display:block;">

![ProjectSetting](images/Dropdwn7.png)

</div>
<div style="width:60%;margin:0 auto;display:block;">

![ProjectSetting](images/Dropdwn8.png)

</div>

8. 在「專案名稱-Bridging-Header.h」裡新增下面這行：
```ObjectiveC
#import "CoreActionSheetPicker/CoreActionSheetPicker.h"
```

9. 在目標 ViewController 裡面新增 Reference Outlet 變數和資料容器，然後將 Reference Outlet 元件建立連結。
```Swift
class ViewController: UIViewController {
        //UITextField 的 Reference Outlet 變數
        @IBOutlet weak var textField: UITextField!

        //下拉選單的內容
        private var dataArray: NSArray!;
        。。。。。
}
```

10. 在 ViewController 的 viewDidLoad() 裡將改造 UITextField 改造成下拉選單。
```Swift
override func viewDidLoad() {
        。。。。。
        //指定下拉符號圖片
        var img = UIImage(named: "dwn")
        self.textField.rightView = UIImageView(image: img)

        //設定圖片填滿右視圖
        self.textField.rightView!.contentMode = UIViewContentMode.ScaleAspectFit
        self.textField.rightView!.clipsToBounds = true

        //顯示下拉符號
        self.textField.rightViewMode = UITextFieldViewMode.Always

        //建立委托
        self.textField.delegate = self

        //設定使用者點擊事件
        self.textField.addTarget(self, action: "selectClicked:", forControlEvents: UIControlEvents.TouchDown);
        。。。。
}
```

11. 最後建立使用者與 UITextField 的互動事件，以及設定 ActionSheetPicker。
```Swift
class ViewController: UIViewController, UITextFieldDelegate {
        。。。。。
        //設定 textField 唯讀
        func textFieldShouldBeginEditing(textField: UITextField) -> Bool {
              return false
        }

        //不讓使用者透過建盤輸入文字
        func textField(textField: UITextField, shouldChangedCharactersInRange range: NSRange, replacementString string: String) {
              return false
        }

        //使用者點擊事件
        func selectClicked(sender: UITextField) {
              ActionSheetStringPicker.showPickerWithTitle(nil, rows: self.dataArray, initialSelection: 0, doneBlock: {
                  picker, value, index in
                  self.textField.text = value
                  return
              }, cancelBlock: { 
                  ActionStringCancelBlock in
                  return
              }, origin: sender.superview)
        }
}
```
