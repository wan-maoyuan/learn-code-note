## I/O
> 将操作系统中的文件，目录（文件夹），路径，封装成File对像

> 提供方法，操作系统中的内容,File与系统无关的类

> 文件：file  目录：directory  路径：path

- 路径：绝对路径、相对路径      

### File类的构造：
```java
/**
 * File类的构造方法
 * 三种重载形式
 */
public class FileDemo01 {
    public static void main(String[] args){
        function_1();
    }
    /**
     * File(File parent, String child)
     * 传递路径，传递File类型的父路径，字符串路径
     */
    public static void function_2(){
        File parent = new File("/Users/wanmaoyuan/Downloads");
        File file = new File(parent, "IMG_1755.JPG");
        System.out.println(file);
    }
    /**
     * File(String parent, String child)
     * 传递路径，传递字符串的父路径，字符串的子路径
     */
    public static void function_1(){
        File file = new File("Downloads", "IMG_1755.JPG");
        System.out.println(file);
    }
    /**
     * File(String pathname)
     * 传递路径名，可以写到文件夹，也可以写到文件
     */
    public static void function(){
        File file = new File("/Users/wanmaoyuan/Downloads/IMG_1755.JPG");
        System.out.println(file);
    }
}
```

### 创建文件
```java
    /**
     * File创建文件的功能
     * boolean creaNewFile()
     * 创建的文件路径和文件名，在File构造方法中给出
     * 文件已经存在，不再创建
     */
    public static void function()throws IOException {
        File file = new File("/Users/wanmaoyuan/Downloads : a.txt");
        boolean b = file.createNewFile();
        System.out.println(b);
```

### 创建文件夹

```java
 * boolean mkdir()
 * 创建路径也在File构造方法中给出
 * 文件夹存在，不再创建
 */
public static void function_1(){
    File file = new File("万茂源");
    boolean b = file.mkdir();（创建单级目录） 		boolean b = file.mkdirs();（创建多级目录）
    System.out.println(b);
}
```

### 删除文件

```java
 * boolean delete()
 * 删除的文件或者是文件夹，在File构造方法中给出
 * 删除成功返回true，删除失败返回false
 */
public static void function_2(){
    File file = new File("文件路径");
    boolean b = file.delete();
    System.out.println(b);
}

文件字节数获取：
File类的获取功能
 * long length()
 * 返回路径中表示的文件的字节数
 */
public static void function_3(){
    File file = new File("/Users/wanmaoyuan/Downloads/jdk api 1.8.CHM");
    long length = file.length();
    System.out.println(length);
}

获取文件的绝对路径：
File类获取功能
 * String getAbsolutePath()
 * File   getAbsoluteFile()
 * 获取绝对路径
 */
public static void function_4(){
    File file = new File("文件路径");
    File absolute = file.getAbsoluteFile();
    System.out.println(absolute);
}
```

### 获取父级路径
```java
 * String getParent()
 * File   getParentFile()
 * 获取父级路径
 */
public static void function_5(){
    File file = new File("文件路径");
    File parent = file.getParentFile();
    System.out.println(parent);
}

判断文件路径是否存在：
File 判断功能
 * boolean exists()
 * 判断File构造方法中封装路径是否存在
 * 存在返回true，不存在返回false
 */
public static void function_6(){
    File file = new File("文件路径");
    boolean b = file.exists();
    System.out.println(b);
}
```

### 判断路径是不是文件夹
```java
 * boolean isDirection()
 * 判断File构造方法中封装的路径是不是文件夹
 * 如果是返回true，如果不是返回false
 */
public static void function(){
    File file = new File("路径");
    boolean b = file.isDirectory();
    System.out.println(b);
}
```

### 判断路径是不是文件：
```java
 * boolean isFile()
 * 判断File构造方法中封装的路径是不是文件夹
 * 如果是返回true，如果不是返回false
 */
public static void function(){
    File file = new File("路径");
    boolean b = file.isFile();
    System.out.println(b);
}
```

### 文件夹的便利
```java
 * String[] list()
 * 获取到File构造方法中封装的路径中的文件和文件夹名
 */
public static void function(){
    File file = new File("/Users/wanmaoyuan/Downloads");
    String[] strArr = file.list();
    for (String str : strArr){
        System.out.println(str);
    }
}
```

### 文件获取的过滤器
```java
 * 实现FileFilter接口，重写抽象方法
 */
public class MyFilter implements FileFilter {
    public boolean accept(File pathname) {
        String name = pathname.getName();
        return name.endsWith(".m4a");
    }


File file = new File("/Users/wanmaoyuan/Downloads");
File[] fileArr = file.listFiles(new MyFilter());
for (File f : fileArr){
    System.out.println(f);
}
```

