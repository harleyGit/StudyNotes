<br/>

***
<br/>

**RxTheme** 可以很轻松的实现换肤，切换主题的功能。

<br/>

**ThemeManager.swift**

```
import Foundation
import RxTheme
import SwifterSwift
import RxSwift
import RxCocoa

//BehaviorRelay使用：https://www.hangge.com/blog/cache/detail_1929.html
let globalStatusBarStyle = BehaviorRelay<UIStatusBarStyle>(value: .default)

//获得主题服务
let themeService = ThemeType.service(initial: ThemeType.currentTheme())

protocol Theme {
    var primary: UIColor { get }
    var primaryDark: UIColor { get }
    var secondary: UIColor { get }
    var secondaryDark: UIColor { get }
    var separator: UIColor { get }
    var text: UIColor { get }
    var textGray: UIColor { get }
    var background: UIColor { get }
    var statusBarStyle: UIStatusBarStyle { get }
    var barStyle: UIBarStyle { get }
    var keyboardAppearance: UIKeyboardAppearance { get }
    var blurStyle: UIBlurEffect.Style { get }
    
    init(colorTheme: ColorTheme)
}

enum ColorTheme: Int {
    case red, pink, purple, deepPurple, indigo, blue
    
    var color: UIColor {
        switch self {
        case .red: return UIColor.Material.red
        case .pink: return UIColor.Material.pink
        case .purple: return UIColor.Material.purple
        case .deepPurple: return UIColor.Material.deepPurple
        case .indigo: return UIColor.Material.indigo
        case .blue: return UIColor.Material.blue
        }
    }
    
    var colorDark: UIColor {
        switch self {
        case .red: return UIColor.Material.red900
        case .pink: return UIColor.Material.pink900
        case .purple: return UIColor.Material.purple900
        case .deepPurple: return UIColor.Material.deepPurple900
        case .indigo: return UIColor.Material.indigo900
        case .blue: return UIColor.Material.blue900
        }
    }
    
}

struct LightTheme: Theme {
    let primary = UIColor.white
    let primaryDark = UIColor.gray
    var secondary = UIColor.red
    var secondaryDark = UIColor.red
    let separator = UIColor.gray
    let text = UIColor.gray
    let textGray = UIColor.gray
    let background = UIColor.white
    let statusBarStyle = UIStatusBarStyle.default
    let barStyle = UIBarStyle.default
    let keyboardAppearance = UIKeyboardAppearance.light
    let blurStyle = UIBlurEffect.Style.extraLight
    
    init(colorTheme: ColorTheme) {
        secondary = colorTheme.color
        secondaryDark = colorTheme.colorDark
    }
}

struct DarkTheme: Theme {
    let primary = UIColor.Material.grey800
    let primaryDark = UIColor.Material.grey900
    var secondary = UIColor.Material.red
    var secondaryDark = UIColor.Material.red900
    let separator = UIColor.Material.grey900
    let text = UIColor.Material.grey50
    let textGray = UIColor.Material.grey
    let background = UIColor.Material.grey800
    let statusBarStyle = UIStatusBarStyle.lightContent
    let barStyle = UIBarStyle.black
    let keyboardAppearance = UIKeyboardAppearance.dark
    let blurStyle = UIBlurEffect.Style.dark
    
    init(colorTheme: ColorTheme) {
        secondary = colorTheme.color
        secondaryDark = colorTheme.colorDark
    }
}

enum ThemeType: ThemeProvider {
    case light(color: ColorTheme)
    case dark(color: ColorTheme)
    
    var associatedObject: Theme {
        switch self {
        case .light(let color): return LightTheme(colorTheme: color)
        case .dark(let color): return DarkTheme(colorTheme: color)
        }
    }
    
    var isDark: Bool {
        switch self {
        case .dark: return true
        default: return false
        }
    }
}


extension ThemeType {
    
    static func currentTheme() -> ThemeType {
        let defaults = UserDefaults.standard
        //是否为深夜模式
        let isDark = false//defaults.bool(forKey: "IsDarkKey")
        let colorTheme = ColorTheme(rawValue: defaults.integer(forKey: "ThemeKey")) ?? ColorTheme.red
        let theme = isDark ? ThemeType.dark(color: colorTheme) : ThemeType.light(color: colorTheme)
        theme.save()
        return theme
    }
    
    func save() {
        let defaults = UserDefaults.standard
        defaults.set(self.isDark, forKey: "IsDarkKey")
        switch self {
        case .light(let color): defaults.set(color.rawValue, forKey: "ThemeKey")
        case .dark(let color): defaults.set(color.rawValue, forKey: "ThemeKey")
        }
    }
    
}



//https://www.hangge.com/blog/cache/detail_1946.html
//状态栏属性的rx扩展
extension Reactive where Base: UIApplication {
    
    var statusBarStyle: Binder<UIStatusBarStyle> {
        return Binder(self.base) { view, attr in
            globalStatusBarStyle.accept(attr)
        }
    }
}


```


<br/>
**AppDelegate.swift**

```

class AppDelegate: UIResponder, UIApplicationDelegate {


    let disposeBag = DisposeBag()

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        
        self.setUpThemeService()
        
        return true
    }
    
    //设置状态栏的颜色
    func setUpThemeService() {
        themeService.rx
            .bind({ $0.statusBarStyle }, to: UIApplication.shared.rx.statusBarStyle)
            .disposed(by: disposeBag)
        
    }
}
```


<br/>

调用

```
import UIKit
import RxTheme
import RxSwift

class ViewController: UIViewController {
    let disposeBag = DisposeBag()

    lazy var btn: UIButton = {
        let view = UIButton.init(frame: CGRect.init(x: 100, y: 100, width: 150, height: 100))
        view.addTarget(self, action: #selector(changeTheme), for: .touchUpInside)
        view.setTitle("主题颜色改变", for: .normal)
        //view.backgroundColor = UIColor.purple
        /**
         *这里把return和参数都省略了
         let multipyClosure = {
         (color: UIColor) in
          return color
         }
         */
        themeService.rx.bind({$0.secondary}, to: view.rx.backgroundColor).disposed(by: disposeBag)
   
        return view
        
    }()
    
    
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        
        self.view.addSubview(btn)
        
        
    }
    
    //状态栏颜色的改变，否则状态栏无法改变
    override var preferredStatusBarStyle: UIStatusBarStyle {
        return globalStatusBarStyle.value
    }

    
    
    @objc func  changeTheme() {
        
        themeService.switch(ThemeType.light(color: .pink))
        
    }
   
}

```

效果：

![png](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z44.png)

点击后效果

![45](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z45.png)