# SSH折腾

[参考教程](https://testerhome.com/topics/17112)

```bash

# 法兰克福
ssh root@45.76.84.193
V2o]JXwZ?GT62u},
# 亚特兰大
149.28.186.178
6j[Ea5rWtLkvzm6e

## 安装shadowsocksR
wget --no-check-certificate  https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log

## 配置用户信息
cat  /etc/shadowsocks.json
vi /etc/shadowsocks.json

{
    "server":"0.0.0.0",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
         "443":"pwd000",
         "9001":"pwd123",
         "9002":"pwd234",
         "9003":"pwd345",
         "9004":"pwd456"                    
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}

# 查看已开放端口
firewall-cmd --list-ports
# 开启各个端口
firewall-cmd --zone=public --add-port=9004/tcp --permanent
# 重启防火墙
firewall-cmd --reload

#重启
ssserver -c/etc/shadowsocks.json -d restart 

#BBR加速
wget –no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh


Congratulations, Shadowsocks-python server install completed!
Your Server IP        :  45.76.84.193 
Your Server Port      :  207 
Your Password         :  201046003 
Your Encryption Method:  aes-256-gcm 




```



换用: http://devpn.space/

用户名: Yaoyouguang

密码:asasdsdfd

[靠谱VPN推荐](https://v2mm.tech/topic/485/%E8%B0%81%E8%83%BD%E6%8E%A8%E8%8D%90%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84vpn/25)

[中国VPN服务](https://www.chinavpnz.com/)

[NordVPN购买](https://join.nordforcn.site/zh/order/?utm_campaign=off153&utm_content&utm_medium=affiliate&utm_source=aff9842&utm_term)

[VyVPN](https://www.aqniu.com/tools-tech/23223.html)



https://www.justhost.com/

选择新西伯利亚的网址

