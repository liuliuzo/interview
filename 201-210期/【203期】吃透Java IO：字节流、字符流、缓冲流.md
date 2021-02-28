1、FileInputStream、FileOutputStream（字节流）
字节流的方式效率较低，不建议使用

2、BufferedInputStream、BufferedOutputStream（缓冲字节流）
缓冲字节流是为高效率而设计的，真正的读写操作还是靠FileOutputStream和FileInputStream，所以其构造方法入参是这两个类的对象也就不奇怪了。

3、InputStreamReader、OutputStreamWriter（字符流）
字符流适用于文本文件的读写，OutputStreamWriter类其实也是借助FileOutputStream类实现的，故其构造方法是FileOutputStream的对象

4.字符流便捷类
Java提供了FileWriter和FileReader简化字符流的读写，new FileWriter等同于new OutputStreamWriter(new FileOutputStream(file, true))

5.BufferedReader、BufferedWriter（字符缓冲流）

```
 BufferedWriter bw = new BufferedWriter(new FileWriter(file, true));

  // 要写入的字符串
  String string = "松下问童子，言师采药去。只在此山中，云深不知处。";
  bw.write(string);
  bw.close();
```
