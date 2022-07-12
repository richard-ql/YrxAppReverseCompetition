\[原创\]猿人学-APP逆向2022对抗赛第八题-upx

2022-5-17 12:02 2815

### \[原创\]猿人学-APP逆向2022对抗赛第八题-upx

![](https://passport.kanxue.com/pc/view/img/star.gif)
![](https://passport.kanxue.com/pc/view/img/star.gif)

1、抓包请求-根据抓包查看请求，参数只有一个s，直接使用jadx搜索app8看看是否能够定位到实现位置  
![](https://bbs.pediy.com/upload/attach/202205/947138_ZXNPEDMYGMH5Y35.png)
  
2、可以看到根据app8搜索并定位到接口请求处，点击调用函数并查找用例  
![](https://bbs.pediy.com/upload/attach/202205/947138_SFVD2ZT2JGZ9CND.png)
  
![](https://bbs.pediy.com/upload/attach/202205/947138_AUU965RUKPAH3NP.png)
  
3、通过分析调用函数可以推断出，f5491OooO0O0为页数，在每次滑动时进行+1操作，通过data函数将页数当作参数传入，返回data值，最终通过OooO0oo函数进行调用

```

```

4、点击data函数，跳转声明，发现data是一个native函数  
![](https://bbs.pediy.com/upload/attach/202205/947138_3Z4V7TNCPNWASGU.png)
  
5、使用frida尝试调用data函数,根据返回值与请求值进行对比发现格式完全一致

```

```

![](https://bbs.pediy.com/upload/attach/202205/947138_WBBBFU7ANPYBXEU.png)
  
6、由于比赛规则，所以使用unidbg进行so调用，发现调用失败，通过ida进行so文件分析,发现找不到JNI\_OnLoad，突然意识到题目是upx  
![](https://bbs.pediy.com/upload/attach/202205/947138_N8MGYDUZBQUN4GM.png)
  
7、先确保so文件是经过upx压缩过的，使用ExeinfoPe查看文件信息,可以看到so文件确实经过了upx压缩处理  
![](https://bbs.pediy.com/upload/attach/202205/947138_UCNVMKMEH7AB5AJ.png)
  
8、那么第一个想法就是解压缩，直接github下载upx工具进行解压缩，使用方法非常简单  
![](https://bbs.pediy.com/upload/attach/202205/947138_QUNDVN433MRN8HY.png)

```

```

![](https://bbs.pediy.com/upload/attach/202205/947138_WFDYQ8NJ87BF3SJ.png)
  
9、可以看到so文件解压缩成功了，文件由原来的约2M变成了4M左右，那么直接再次尝试通过unidbg调用data函数  
![](https://bbs.pediy.com/upload/attach/202205/947138_CNT475D3F6KJZNP.png)
  
10、可以请求到接口数据了，下面是部分udb代码

```
public String getSign(Integer allParams) {
    DvmObject<?> context = vm.resolveClass("com/yuanrenxue/match2022/fragment/challenge/ChallengeEightFragment").newObject(null);
    DvmObject<?> strRc = context.callJniMethodObject(emulator, "data(I)Ljava/lang/String;", allParams);
    System.out.println(strRc.getValue());
    return (String) strRc.getValue();
}
```

比赛链接：https://appmatch.yuanrenxue.com/