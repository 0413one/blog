昨日（*Nov3,2025的昨日）把me.htm单独弄了一个子域名me.karina.xin
没想到居然要这么多步骤，但不过所幸素可以的，这就非常好了

Cloudflare里面的我的域名新建一个二级域名分配给我的Github中的存储库中的一个文件
那个存储库它已经分配了一个主域名了，它里面的一个文件再分配一个子域名
透过那个二级域名直接访问，后面不用加上文件名。

没想到移动端不能在创建的时候直接编辑代码啊啊
尝试了一下从 Safari 浏览器切换到了 Chrome 浏览器，并且都尝试打开了桌面端访问，但是网站好像并没有响应…..
后来把无痕访问也试了一下…依然没有

索性直接创建了，顶部的 "部署" 标签 → 进入 "编辑代码" 界面。


话说这个移动端的代码编辑器好难用，硬着头皮改了，在微信里面编辑了代码然后Copy到这里了

`export default {
  async fetch(request, env) {
    const CUSTOM_DOMAIN = "me.karina.xin";
    const TARGET_FILE_URL = "https://raw.githubusercontent.com/0413one/karina.github.io/main/me.htm";
    const MIME_TYPE = "text/html";

    const url = new URL(request.url);

    if (url.hostname === CUSTOM_DOMAIN && url.pathname === "/") {
      try {
        const response = await fetch(TARGET_FILE_URL);
      
        return new Response(response.body, {
          headers: {
            "Content-Type": MIME_TYPE + ";charset=utf-8",
            "Access-Control-Allow-Origin": "*"
          }
        });
      } catch (e) {
        return new Response("访问失败：" + e.message, { status: 500 });
      }
    }
   
    return new Response("Not Found", { status: 404 });
  }
};`

这个Raw文件还一开始不知道是什么，后面看了一下实例说是

`https://raw.githubusercontent.com/用户名/仓库名/分支名/文件路径

https://raw.githubusercontent.com/0413one/karina.github.io/main/me.htm`

也是一次性速通好吧（
后面就简单了，创建并且绑定了一下域名me.karina.xin也是顺利完成

还以为到这里就结束了，但是透过Safari浏览器查看的时候发现CSS没掉了，然后去问了一下怎么加CSS，果然跟我想的一样很麻烦

说什么路径问题也有，我在me.htm这个文件当中引用的CSS文件的路径是相对路径然后在Github Raw中是无法直接访问的。
然后还说GitHub Raw 文件的 Content-Type 可能被浏览器拦截因为安全策略限制，这段就看不懂了

提供的解决方法是
`1. 需要把所有外部资源的路径改为完整的 Raw 地址（如 https://raw.githubusercontent.com/0413one/karina.github.io/main/css/style.css）。
2. 在 Workers 代理中添加 Access-Control-Allow-Origin: * 头，允许跨域加载。
3. 修改Workers代码

export default {
  async fetch(request, env) {
    const CUSTOM_DOMAIN = "file.karina.xin";
    const GITHUB_RAW_URL = "https://raw.githubusercontent.com/0413one/karina.github.io/main";
    const url = new URL(request.url);

    if (url.hostname === CUSTOM_DOMAIN) {
      try {
        // 映射路径到 GitHub Raw
        const path = url.pathname === "/" ? "/me.htm" : url.pathname;
        const targetUrl = new URL(GITHUB_RAW_URL + path);
        const response = await fetch(targetUrl);

        // 设置正确的 MIME 类型和跨域头
        return new Response(response.body, {
          headers: {
            "Content-Type": response.headers.get("Content-Type") || "text/plain;charset=utf-8",
            "Access-Control-Allow-Origin": "*"
          }
        });
      } catch (e) {
        return new Response("资源加载失败：" + e.message, { status: 500 });
      }
    }
    return new Response("Not Found", { status: 404 });
  }
};`

看得头都大了 我直接把 css 文件写进那个 html 文件 简单粗暴
确实冗余了，我不是一个好码农2333
但确切不想弄了，谁知道又有多少的麻烦事，索性就这样子吧～

原来的链接很长，加上前面的加密协议就变成了https://karina.xin/me.htm实在是不好打，现在只需https://me.karina.xin即可，去掉前面协议就是me.karina.xin

总之这一次的经历还是非常不错的，非常有意义，让链接大大缩短，并且这是一个非常宝贵的经验！