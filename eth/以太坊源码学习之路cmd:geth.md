疑问待解决

+ utils包中有一个方法，是打印错误并退出

  ```
  // Fatalf formats a message to standard error and exits the program.
  // The message is also printed to standard output if standard error
  // is redirected to a different file.
  func Fatalf(format string, args ...interface{}) {
  	w := io.MultiWriter(os.Stdout, os.Stderr)
  	if runtime.GOOS == "windows" {
  		// The SameFile check below doesn't work on Windows.
  		// stdout is unlikely to get redirected though, so just print there.
  		w = os.Stdout
  	} else {
  		outf, _ := os.Stdout.Stat()
  		errf, _ := os.Stderr.Stat()
  		if outf != nil && errf != nil && os.SameFile(outf, errf) {
  			w = os.Stderr
  		}
  	}
  	fmt.Fprintf(w, "Fatal: "+format+"\n", args...)
  	os.Exit(1)
  }
  ```

  用了一大堆都是设置write流，是为了，如注释中所言，当标准输出出错的时候能重定向到文件中？什么时候标准输出出错？

+ 复制实例

  ```
  conf *Config

  confCopy := *conf //confCopy为类型值，conf指向的地址类型
  conf = &confCopy //类型取地址 = = 新地址就这么生成了
  ```

  就这么两行代码就完成了对象值的复制。要不要这么方便

  复制map,services的copy -> services_copy

  ```
  services_copy := make(map[reflect.Type]Service),
  for kind, s := range services { // copy needed for threaded access,range循环是取值的副本
  	services_copy[kind] = s   
  }
  ```

+ 还是log相关，eth中单独写了一个log包

  ```
  github.com/inconshreveable/log15
  ```

  应该是这个log包的本地化。

  emm,  具备log等级(Info, Debug, Error)等，打印方式主要是key-value等等特性

  ​



