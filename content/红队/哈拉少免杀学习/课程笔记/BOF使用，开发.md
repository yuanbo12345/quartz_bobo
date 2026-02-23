# BOF原理
1. 文件内存执行，不落地磁盘，就不会有静态查杀
2. CS早期采用的时RDI的方式运行插件基于fork&run的加载方法：打开一个子进程将代码注入到子进程中（也是内存加载，不落地），360核晶直接拦截注入
3. bof时再beacon的进程内部运行的，内存加载不落地磁盘，不创建新进程，规避了进程检测，方便迁移小型的C开发工具，实现在不创建进程的情况下于内存中加载.NET程序集
4. bof不可以进行超长时间的任务处理，大于60s的都不太行
5. 注意BOF一旦开始运行，其他的beacon的任务就不能再运行了
![[1744335199370.jpg]]

6. 疑问：BOF引用的API也是来自于目标主机的dll文件中吗？
7. BOF的使用可以看哈拉少关于卡巴那一节，使用bof进行敏感行为？
8. 只要是进程注入，核晶全都拦截不论是不是利用BOF加载
9. 为什么核晶看到任何注入立刻就能拦截报毒？
10. 疑问：一个只是在一个BOF中使用patch ETW和ASMI那么后续通过该beacon执行的所有行为都可以绕过ETW和beacon吗？
11. 根据bof的原理后续再利用beacon执行其他操作的话就需要关闭掉这个bof，那么上一个bof带来的规避效果就会失效？这样的话想要再去执行其他敏感操作的话是不是还要使用其他的BOF才行？

# BOF项目推荐
1. BOFnet项目
2. https://github.com/trustedsec/CS-Situational-Awareness-BOF
3. https://github.com/trustedsec/CS-Remote-OPs-BOF
4. 上面两个github项目来自于同一个团队trustedsec很厉害，可以多看下他们的项目
5. 