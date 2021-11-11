# 获取项目当前路径四种方式

> ​	注意run，build，test 运行获取的路径会有不同

> ​	正式的开发环境使用os.Executable()配合filepath.EvalSymlinks()效果更搭

- os.Args[0]

- os.Getwd()

  ```go
  pwd, err := os.Getwd()
  println("Getwd: ", pwd, err)
  ```

- os.Executable()

  ```go
  exePath, _ := os.Executable()
  res, _ := filepath.EvalSymlinks(filepath.Dir(exePath))
  println(res)
  ```

- runtime.Caller(0)

  ```go
  var  string
  _, filename, _, ok := runtime.Caller(0)
  if ok {
      abPath = path.Dir(filename)
  }
  println(abPath)
  ```