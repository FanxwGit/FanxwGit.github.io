在Web开发初学的时候，常常会遇到web的中文乱码问题，非常痛苦...![image-20210610123629687](C:\Users\37802\Desktop\image-20210610123629687.png)

![image-20210610123715202](C:\Users\37802\Desktop\image-20210610123715202.png)

当时我就在网上胡搜，就不断复制黏贴，就搞定了，也没搞清楚个明白。![image-20210610123939965](C:\Users\37802\Desktop\image-20210610123939965.png)

## 乱码原因的本质

Tomcat服务器默认用ISO-8859-1进行编码，我们写了中文以后，ISO-8859-1编码后，发送给客户端（浏览器），客户端（浏览器）一脸懵逼用**GBK的解码规则**进行解码，结果就**乱码**了...![image-20210610123334221](C:\Users\37802\Desktop\image-20210610123334221.png)



## 解决方案

从难到简，一步一步剖析后面背后的有趣过程。

### 解决方案1.0

Tomcat用ISO-8859进行编码，浏览器用GBK解码，我们让其统一就可以了，第一个方法就是让**Tomcat用GBK进行编码**

![image-20210610124219961](C:\Users\37802\Desktop\image-20210610124219961.png)

![image-20210610124305662](C:\Users\37802\Desktop\image-20210610124305662.png)

#### 代码实现

```
@WebServlet("/Test01")
public class Test01 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setCharacterEncoding("GBK");//设置编码格式！！！
        final PrintWriter writer = resp.getWriter();
        writer.write("你好");
    }
}
```



解决方案1让服务器去妥协成GBK解码,万一浏览器用UFT-8编码岂不是很尴尬？

### 解决方案2.0

所以我们得告诉浏览器，我用的是UTF-8，还是GBK,这样就不会出现不一致的情况，那么如何实现呢？![image-20210610125613373](C:\Users\37802\Desktop\image-20210610125613373.png)

HTTP协议的响应头里有这个规定：

![image-20210610125436816](C:\Users\37802\Desktop\image-20210610125436816.png)

这个请求头信息决定浏览器将以什么形式、什么编码读取这个文件。



所以我们就可以调用setHeader（设置响应头）的函数，来指定Content-Type的内容，**同时还能指定字符流的编码格式**，这样服务器和客户端都能统一。

```
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //resp.setCharacterEncoding("UTF-8");下面一行，就可以告诉字符流，我们的编码格式是UTF-8,这行可以省略
        resp.setHeader("Content-type","text/html;charset=UTF-8");//第一个是响应头的名称，第二个是值
        final PrintWriter writer = resp.getWriter();
        writer.write("你好!!");
    }
```

![image-20210610125850494](C:\Users\37802\Desktop\image-20210610125850494.png)



Tomcat还封装了一个函数，resp.setContentType()，可以直接修改ContentType，懒人就可以少写一个参数了。

```
resp.setContentType("text/html;charset=UTF-8");
```

### 最终方案

```
 protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        final PrintWriter writer = resp.getWriter();
        writer.write("你好!!");
 }
```



