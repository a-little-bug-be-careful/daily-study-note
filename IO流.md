**IO流**

1. 分类

   > - 流向分类（以程序为参照物，将文件中的数据读取到程序中为输入流，将程序中的数据写入到文件中为输出流）
   >
   >   输入流、输出流
   >
   > - 流数据类型分类
   >
   >   字节流、字符流
   >
   > - 常用IO流类
   >
   >   字节流适合处理图片、视频等文件，一般纯文本文件不太适合
   >
   >   > 字节输入流：顶层父类InputStream ，常用子类FileInputStream、BufferedInputStream
   >   >
   >   > 字节输出流：顶层父类OutputStream ，常用子类FileOutputStream、BufferedOutputStream
   >   >
   >   > FileInputStream常用方法：
   >   >
   >   > - read()：读取一个字节数据
   >   >
   >   > - read(byte[] bytes)：从输入流读取指定长度字节数组的数据，返回值为实际读取到的字节数，尽可能读满整个字节数组，但是有时候由于文件长度问题，比如字节数组大小指定为10，最后文件读取时只剩3个字节，则只会读取到3个字节，数组中其他位置的数据可能为上次读取的数据或者默认值，所以在使用该方法时，要结合着读取到的实际字节数也就是方法返回值一起用
   >   >
   >   >   ~~~java
   >   >   try (InputStream inputStream = new FileInputStream("C:\\Users\\tongxin\\Desktop\\R-C.jpg");
   >   >                OutputStream outputStream = new FileOutputStream("C:\\Users\\tongxin\\Desktop\\copy.jpg")){
   >   >               int length = 0;
   >   >               LocalDateTime localDateTime1 = LocalDateTime.now();
   >   >               byte[] bytes = new byte[100];
   >   >               while ((length = inputStream.read(bytes)) != -1) {
   >   >                   //使用时，要指定字节数组中实际读取的字节数，防止产生垃圾数据
   >   >                   outputStream.write(bytes, 0, length);
   >   >               }
   >   >               System.out.println("time: " + Duration.between(localDateTime1, LocalDateTime.now()).toMillis());
   >   >           } catch (IOException ioException) {
   >   >   
   >   >           }
   >   >   ~~~
   >   >
   >   > - read(byte[] b, int off, int len)：从输入流读取指定字节数的数据到该字节数组b中，位置从off开始（初始下标为0），实际上read(byte[] bytes)方法调用了read(byte[] b, int off, int len)
   >   >
   >   >   ```java
   >   >   public int read(byte[] b) throws IOException {
   >   >       return read(b, 0, b.length);
   >   >   }
   >   >   ```
   >   >
   >   > - write(int b)：写一个字节的数据
   >   >
   >   > - write(byte[] b)：写一个指定字节数组的数据到文件中
   >   >
   >   > - write(byte[] b, int off, int len)：从一个指定字节数组b中指定off位置写指定长度len的数据到文件中
   >
   >   字符流适合处理纯文本文件（字符流基于字节流和字符集）
   >
   >   > 字符输入流：顶层父类Reader，常用子类FileReader、BufferedReader，基于字节读取（utf-8编码格式下，英文占一个字节，中文占3个字节），读取后需要对字节进行解码（将二进制字节码转换为对应的unicode编码），有缓冲区（8192字节数组）
   >   >
   >   > 字符输出流：顶层父类Writer，常用子类FileWriter、BufferedWriter，基于字节写入。将数据编码后（将unicode码转为二进制字节码）先写入缓冲区，缓冲区满（8192字节数组）或者字符输出流关闭或者手动调用flush之后，数据写入文件。
   >
   >   