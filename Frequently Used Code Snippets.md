这里会记录一些Chris常用的Code Snippets🥐

### 为`String?`添加`extension`来处理`nil`和`""`

```swift
extension Optional where Wrapped == String {
    var nilIfEmpty: String? {
        guard let strongSelf = self else {
            return nil
        }
        return strongSelf.isEmpty ? nil : strongSelf
    }
}
```

详细请见 [为String?添加extension](https://github.com/objchris/ChrisDerSwiftCookBook/blob/master/Chris%20der%20Swift%20Cookbook.md#为string加extension)

正在努力搓面团中...