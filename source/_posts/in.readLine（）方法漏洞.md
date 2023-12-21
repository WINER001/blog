---
title: 修复漏洞拒绝服务（Denial of Service）
date: 2023/5/30 10:24
tags: 漏洞修复
categories: 漏洞修复
top_img: ./img/41.jpg
cover: ./img/31.jpg


---

## 



## in.readLine（）方法漏洞

```java
StringBuilder response = new StringBuilder();
try(BufferedReader in  = new BufferedReader(new InputStreamReader(connection.getInputStream(),"UTF-8"))){
       String line;
       while((line = in.readLine()) != null){
          response.append(line);
       }
}
```

在这段代码中，使用`readLine()`读取输入流的方式可能会导致攻击者使用恶意输入来造成程序崩溃或拒绝服务（Denial of Service）的情况。攻击者可以发送大量的换行符来使读取操作变得非常缓慢，最终耗尽系统资源。



### 优化方案：

1. #### 设置超时时间：

   使用`setReadTimeout()`方法设置读取超时时间，确保读取操作不会无限期地阻塞。可以通过以下代码示例设置超时时间为5秒：
```java
connection.setReadTimeout(5000);
```
这样，如果读取操作在指定的时间内没有完成，将会抛出`java.net.SocketTimeoutException`异常，你可以根据具体情况处理该异常。



2. #### 限制读取的最大字节数：（我选择的解决方式）

   可以使用`read()`方法，并限制每次读取的最大字节数，以防止恶意输入导致缓冲区溢出或消耗过多内存。例如，可以使用以下代码将每次读取的最大字节数限制为1024字节：
```java
StringBuilder response = new StringBuilder(1024);
try(BufferedReader in  = new BufferedReader(new InputStreamReader(connection.getInputStream(),"UTF-8"))){
		char[] buffer = new char[1024];
		int bytesRead;
		while ((bytesRead = in.read(buffer, 0, buffer.length)) != -1) {
    response.append(buffer, 0, bytesRead);
}
```
通过限制每次读取的最大字节数，可以有效地控制内存使用并防止缓冲区溢出的攻击。

3. #### 使用`InputStream`而不是`Reader`

   如果不需要处理字符数据，而是处理二进制数据，可以直接使用`InputStream`来读取输入流，并避免字符编码相关的问题。例如：
```java
try (InputStream in = connection.getInputStream()) {
    byte[] buffer = new byte[1024];
    int bytesRead;
    while ((bytesRead = in.read(buffer, 0, buffer.length)) != -1) {
        // 处理二进制数据
    }
}
```
通过使用`InputStream`，可以更有效地处理二进制数据，并避免字符编码相关的安全问题。



