# 环境变量

## 使用方法

首先导入PerfectLib：

``` swift
import PerfectLib
```

现在可以在程序中直接调用`Env`类对象操作环境变量：

### 设置

- 设置一个环境变量

以下操作等同于 bash 命令 "export foo=bar"

``` swift
Env.set("foo", value: "bar")
```

- 设置一组环境变量

同样可以使用字典方式设置一组环境变量

``` swift
Env.set(["foo":"bar", "koo":"kar"])
// 结果等同于 bash 命令 "export foo=bar && export koo=kar"
```

### 读取

- 查询单个变量：
	
``` swift
guard let foo = Env.get("foo") else {
	// 查询失败
}
```

- 查询单个变量，并附加默认值（如果不存在这个变量就用默认值代替）

``` swift
guard let foo = Env.get("foo", defaultValue: "bar") else {
	// 既然有默认值，则查询应该不会失败
}
```

- 查询所有系统变量

``` swift
let all = Env.get()
// 结果是一个字典 [String: String]
```

### 删除

- 删除一个环境变量:


``` swift
Env.del("foo")
```