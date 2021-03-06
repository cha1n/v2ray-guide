# VMess

VMess 协议是由 V2Ray 原创并使用于 V2Ray 的加密传输协议，如同 Shadowsocks 一样为了对抗墙的[深度包检测](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E5%8C%85%E6%A3%80%E6%B5%8B)而研发的。在 V2Ray 上客户端与服务器的通信主要是通过 VMess 协议通信。

本小节给出了 VMess 的配置文件，其实也就是服务器和客户端的基本配置文件，这是最简单的配置了。

V2Ray 使用 inbound 和 outbound 的概念，这个概念非常清晰地体现了数据包的流动方向，同时也使得 V2Ray 功能强大复杂的同时而不混乱，结构清晰明了。简单来说，V2Ray 就是一个盒子，这个盒子有出口和入口，我们将数据包通过某个入口放进这个盒子里，然后这个盒子以某各机制（这个机制其实就是路由，后面会讲到）决定这个数据包走哪个出口并将数据包发出去。建议选看一下 V2Ray 的[工作原理](https://www.v2ray.com/chapter_01/internal.html)。

-------
## 配置前的准备
实际上，根本不用准备什么，只要有一个文本编辑器(text editor)就可以修改配置了。但我还是打算啰嗦一些，因为我发现新手容易犯语法（格式）不正确的错误，这个很正常新手上路对路况总会不是很熟悉；另外一个就是不会使用工具，用了很多年电脑文本编辑还是 Windows 自带的记事本（在我身边有不少敲代码的，平常看某个代码文件不想打开 IDE 就直接用记事本看），用水果刀切菜可以吗？当然可以，建议你亲自体验一下。如果你会用工具，会非常高效的而且装有一些插件可以语法检查，将代码格式化。

文本编辑器有许多，比如说 sublime text、vs code、atom、notepad++，上面这些都是跨平台的，具体如何使用请自行 google 吧。这些软件都可以做到高亮显示、折叠、格式化等，建议使用，如果你不想安装软件，网上也有一些在线的 json 编辑器，还自动检查语法。如果你非要用 Windows 的记事本我也无话可说。

对于 Linux 有一个软件叫 jq，可以执行这样的指令检查配置文件的语法是否正确：
```
jq . config.json
```
这里的 config.json 是当前目录下的 config.json。特别注意命令中的点 . 不能省去。

![](/base_config/jqdemo.png)
当我把 "23ad6b10-8d1a-40f7-8ad0-e3e35cd38297" 后的逗号 , 删去时：

![](/base_config/jqerror.png)


有一点要注意的是，修改完配置之后要重启 V2Ray 才能使用新配置生效。

**另外再强调一遍，V2Ray 的认证是基于时间，请确保服务器和客户端的时间准确，误差一分钟内即可**

## 客户端
为了方便，就以 V2Ray 默认的 config.json 为例（为了简单说明，我将配置删去了大部分东西，只留下 inbound 和 outbound，inbound 和 outbound 是 V2Ray 的两个必不可少的配置项），如果你是要在 VPS 上部署，请将 address 改成你的服务器地址。请看 inbound，port 为 1080，V2Ray 监听了一个端口 1080，协议是 socks。之前我们已经把浏览器的代理设置好了（SOCKS Host: 127.0.0.1，Port: 1080），假如访问了 google.com，浏览器就会发出一个数据包打包成 socks 协议发送到本机（127.0.0.1指的本机，localhost）的 1080 端口，这个时候数据包就会被 V2Ray 接收到。再看 outbound，protocol 是 vmess，说明 V2Ray 接收到数据包之后要将数据包打包成 [VMess](https://www.v2ray.com/chapter_03/01_effective.html#vmess-%E5%8D%8F%E8%AE%AE) 协议并且使用预设的 id 加密（这个例子 id 是 23ad6b10-8d1a-40f7-8ad0-e3e35cd38297），然后发往服务器地址为 v2ray.cool 的 17173 端口。服务器地址 address 可以是域名也可以是 IP，只要正确就可以了。

客户端配置：
```javascript
{
  "inbound": {
    "port": 1080, // 监听端口
    "protocol": "socks", // 入口协议为 SOCKS 5
    "settings": {
      "auth": "noauth"  // 不认证
    }
  },
  "outbound": {
    "protocol": "vmess", // 出口协议
    "settings": {
      "vnext": [
        {
          "address": "v2ray.cool", // 服务器地址，请修改为你自己的服务器 ip 或域名
          "port": 10086,  // 服务器端口
          "users": [
            {
              "id": "23ad6b10-8d1a-40f7-8ad0-e3e35cd38297",  // 用户 ID，须与服务器端配置相同
              "alterId": 64
            }
          ]
        }
      ]
    }
  }
}
```

## 服务器
接着看服务器，服务器配置的 id 是 23ad6b10-8d1a-40f7-8ad0-e3e35cd38297，所以 V2Ray 服务器接收到客户端发来的数据包时就会尝试用 23ad6b10-8d1a-40f7-8ad0-e3e35cd38297 解密，如果解密成功再看一下时间对不对，对的话就把数据包发到 outbound 去，outbound.protocol 是 freedom（freedom 的中文意思是自由，在这里姑且将它理解成直连吧），数据包就直接发到 google.com 了。

配置中的 alterId 也是作为认证的，具体请看 [V2Ray 用户手册](https://www.v2ray.com/chapter_03/01_effective.html#alterid)。只要确保服务器和客户端配置文件的 alterId 相同就行了，但要注意 alterId 的值越大会使用 V2Ray 占用更多的内存。根据我的经验，对于一般用户来说，alterId 的值设为 50 到 150 之间应该是比较合适的。

服务器配置：
```javascript
{
  "inbound": {
    "port": 10086, // 主端口
    "protocol": "vmess",    // 主传入协议
    "settings": {
      "clients": [
        {
          "id": "23ad6b10-8d1a-40f7-8ad0-e3e35cd38297",  // 用户 ID，客户端须使用相同的 ID 才可以中转流量
          "alterId": 64
        }
      ]
    }
  },
  "outbound": {
    "protocol": "freedom",  // 主传出协议，参见协议列表
    "settings": {}
  }
}
```

简单来说就是 浏览器打包 ---> V2Ray 客户端接收 -> V2Ray 客户端发出 --->  V2Ray 服务器接收 -> V2Ray 服务器发出 ---> 目标网站
