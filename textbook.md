# config.yaml教材

```yaml
# 必须开启、模块赖以工作的条件
tproxy-port: 7893
bind-address: '*'
# HTTP 代理端口
port: 7890
# SOCKS5 代理端口
socks-port: 7891
# 是否允许局域网的连接，如关闭会导致热点没用网络
allow-lan: false
# 工作模式 rule（根据规则分流） global（直接走GLOBAL分组） direct（直连，不通过代理）
mode: rule
# 日志显示级别 debug info warning error silent
log-level: warning
# 如无必要不建议启用IPV6
ipv6: false
# 网页面板监听地址，没事别改
external-controller: 0.0.0.0:9090
# 本地网页面板的的相对路径（相对/data/clash）
external-ui: yacd
# 网页面板的密码
secret: 'clash'

dns:
    # 必须为true
    enable: true
    # 不要改变端口，若allow-lan为true则必须为0.0.0.0:1053
    listen: 0.0.0.0:1053
    # 如无必要不建议启用IPV6
    ipv6: false
    # 字面意思
    default-nameserver:
        - 114.114.114.114
        - 223.5.5.5
    # 两种 fake-ip  redir-host，若要修改先阅读教程
    enhanced-mode: redir-host
    # 如果是fake-ip模式，则哪些url不走fake-ip
    fake-ip-filter:
        - localhost.ptlogin2.qq.com
    # 国内的dns解析服务
    nameserver:
        - https://rubyfish.cn/dns-query
        - https://dns.alidns.com/dns-query
        - https://doh.pub/dns-query
    # 国外的dns解析服务
    fallback:
        - https://doh.opendns.com/dns-query#Proxy
        - https://doh.dns.sb/dns-query#Proxy
        - https://dns64.dns.google/dns-query#Proxy
        - https://dns.google/dns-query#Proxy
        #
        # 特别说明，上面的#Proxy不是注释，yaml语法，注释#前必须为空格或换行符
        # Proxy是下方代理组的名字，意思是这些DNS查询也走这个代理组
        #

# 即使不用也要保留，模块合并文件时的锚点
proxies:
# 在这配置订阅
proxy-providers:
  # 随便的机场名字
  SpeederOne:
    # 订阅类型 file http；用订阅链接的用http，本地文件用file
    type: http
    # 机场订阅链接
    url: "机场订阅链接"
    # 下载好的文件的存储相对位置（相对/data/clash）
    path: ./proxy_providers/SpeederOne.yaml
    # 自动更新订阅间隔，秒为单位
    interval: 3600
    # 健康检查:检查节点可用性
    health-check:
      # 开启健康检查
      enable: true
      # 健康检查用的网址
      url: http://www.gstatic.com/generate_204
      # 定时健康检查
      interval: 300
    # 过滤部分节点，请自行学习正则表达式
    filter: "香港|美国|HK|US"
  BaiPiao:
    # 此处用的file类型，
    type: file
    path: ./proxy_providers/BaiPiao.yaml
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 300

# 设置代理分组，
proxy-groups:
-
  # 组名
  name: Proxy
  # 行为，有多种，具体看教程，如自动选延迟低的、负载均衡啥的
  type: select
  use:
    # 把前面的机场节点放到组里面 前面proxy-providers下的机场名字
    - SpeederOne
    - BaiPiao
  # 过滤部分节点，请自行学习正则表达式
  filter: "香港|美国|HK|US"
-
  # 国内流量分组
  name: Domestic
  type: select
  use:
    # 同上  机场得放在use下
    - BaiPiao
  proxies:
    # 可以引用其他的组，但是得放在proxies下
    # DIRECT（直连）为自带的分组
    - DIRECT
    # 引用上一个分组
    - Proxy
-
  name: Google
  type: select
  proxies:
    - Domestic
    - Proxy
  use:
    - SpeederOne
-
  name: Others
  type: select
  proxies:
    - Proxy
    - Domestic

# 规则配置
rule-providers:
  # 规则名
  Ad:
    # 规则类型 file为文件保存在本地，要更新得手动， http是在网上的规则
    type: file
    # 行为看文档
    behavior: classical
    # 规则保存的相对位置
    path: ./rule_providers/Ad.yaml
  #这里是http类型的，必须有网页地址 即url
  GoogleDrive:
    type: http
    behavior: classical
    # 规则的链接
    url: "https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/GoogleDrive/GoogleDrive.yaml"
    # 下载下来存哪儿
    path: ./rule_providers/GoogleDrive.yaml

# 分流规则 使用规则，按顺序匹配
#
# 重点(敲黑板)
# 规则从上往下顺序匹配，前面匹配了就不会继续向后匹配
#
rules:
  # 防止回环
  - IP-CIDR,127.0.0.1/32,REJECT,no-resolve
  # RULE-SET使用上面的rule-providers的规则
  # 语法：RULE-SET,规则名,匹配后的行为
  
  # 此处行为为拒绝，即没网
  - RULE-SET,Ad,REJECT
  # 此处行为为使用Google分组（proxy-groups）
  - RULE-SET,GoogleDrive,Google
  # 匹配国内的流量,此处为直连，不走代理
  - GEOIP,CN,Domestic
  # MATCH为全匹配，意思就是其他没匹配到的走Others分组（proxy-groups）
  - MATCH,Others
```
