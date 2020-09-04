# SwiftUI[简单构造一个App]


- Demo的 Code
```
//
//  ContentView.swift
//  SwfitUITest
//
//  Created by Harley Huang on 3/9/2020.
//  Copyright © 2020 Harley Huang. All rights reserved.
//

import SwiftUI


struct ChatMessage : Hashable {
    var message: String
    var avatar: String
}

struct ProgrammingBook: Identifiable {
    var id = UUID()
    var name: String
    var publisher: String
    var content: String
    var imageName: String
}

let programmingBooks: [ProgrammingBook] = [
    ProgrammingBook(name: "Objective-C", publisher: "歐萊禮",   content: "Objective-C is a general-purpose, object-oriented programming language that adds Smalltalk-style messaging to the C programming language. It was the main programming language supported by Apple for the macOS, iOS and iPadOS operating systems, and their respective application programming interfaces (APIs) Cocoa and Cocoa Touch until the introduction of Swift in 2014", imageName: "ouLaiNi"),
    ProgrammingBook(name: "SwiftUI", publisher: "aPress", content: "SwiftUI is an innovative, exceptionally simple way to build user interfaces across all Apple platforms with the power of Swift. Build user interfaces for any Apple device using just one set of tools and APIs. With a declarative Swift syntax that’s easy to read and natural to write, SwiftUI works seamlessly with new Xcode design tools to keep your code and design perfectly in sync. Automatic support for Dynamic Type, Dark Mode, localization, and accessibility means your first line of SwiftUI code is already the most powerful UI code you’ve ever written.", imageName: "huoBan"),
    ProgrammingBook(name: "Swift", publisher: "歐萊禮", content: "This article is about the Apple programming language. For the scripting language, see Swift (parallel scripting language).", imageName: "aPress"),
    ProgrammingBook(name: "Python", publisher: "歐萊禮", content: "Python is an interpreted, high-level, general-purpose programming language. Created by Guido van Rossum and first released in 1991, Python's design philosophy emphasizes code readability with its notable use of significant whitespace. Its language constructs and object-oriented approach aim to help programmers write clear, logical code for small and large-scale projects", imageName: "ouLaiNi"),
    ProgrammingBook(name: "Java", publisher: "歐萊禮", content: "This article is about a programming language. For the software platform, see Java (software platform). For the software package downloaded from java.com, see Java Platform, Standard Edition. For other uses, see Java (disambiguation)", imageName: "ouLaiNi")
]


struct ContentView: View {
    
    var programmingBooks: [ProgrammingBook] = []
    
    var body: some View {
        
        return VStack {
            NavigationView {
                List(programmingBooks) { programmingBook in
                    
                    NavigationLink(destination: DetailView(name: programmingBook.name, content: programmingBook.content, imageName: programmingBook.imageName)) {
                        Image(programmingBook.imageName)
                            .clipShape(Circle())
                        VStack (alignment: .leading){
                            VStack {
                                Text(programmingBook.name)
                                    .font(.title)
                                Text(programmingBook.publisher)
                                    .foregroundColor(.red)
                                    .font(.subheadline)
                            }
                        }
                    }
                }
                .navigationBarTitle(Text("programmingBooks"))
            }
            
        }
        
        
    }
}

//#if DEBUG
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView(programmingBooks: programmingBooks)
    }
}
//#endif

```
效果图：
<br/>
![programmingBooks 在Resume下的效果图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/iOS开发/Swift开发/Pictures/swiftUI‘sDemo.png)
<br/>
初学SwiftUI，不知道为什么在Simulator无法出现这种效果，追踪原因是因为没有加载上数据。
上述的代码和效果从[这里获取(需要vpn)](https://medium.com/@mikru168/swiftui-淺玩-swiftui-用其建構一簡單的-app-2f2477bd49d7)查看。

```
//
//  DetailView.swift
//  SwfitUITest
//
//  Created by Harley Huang on 4/9/2020.
//  Copyright © 2020 Harley Huang. All rights reserved.
//

import SwiftUI

struct DetailView: View {
    var name: String
    var content: String
    var imageName: String
    
    var body: some View {
        VStack {
            Image(imageName)
            Text(name)
                .font(.largeTitle)
            Text(content)
                .font(.headline)
            }.padding()
        .navigationBarTitle(Text("Detail"))
    }
}

struct DetailView_Previews: PreviewProvider {
    static var previews: some View {
        DetailView(name: "Test", content: "Test DetailView", imageName: "aPress")
    }
}


```
效果图：
![详情图](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/iOS开发/Swift开发/Pictures/swiftUI'sDemo2.png)






















