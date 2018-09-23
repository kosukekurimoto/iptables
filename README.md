# iptables

```bash
#!/bin/bash

# interface
INTERFACE=eth0

# 初期化
/etc/rc.d/init.d/iptables stop

# デフォルトルール
iptables -P INPUT   DROP   # 受信はすべて破棄
iptables -P OUTPUT  ACCEPT # 送信はすべて許可
iptables -P FORWARD DROP   # 通過はすべて破棄

# 自ホストからのアクセスをすべて許可
iptables -A INPUT -i lo -j ACCEPT

# 内部から行ったアクセスに対する外部からの返答アクセスを許可
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SYN Cookies を有効にする
# SYNフラッド攻撃( SYN Flood Attack )対策
sysctl -w net.ipv4.tcp_syncookies=1 > /dev/null
sed -i '/net.ipv4.tcp_syncookies/d' /etc/sysctl.conf
echo "net.ipv4.tcp_syncookies=1" >> /etc/sysctl.conf

# ブロードキャストアドレス宛 ping には応答しない
# ※Smurf 攻撃対策
sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1 > /dev/null
sed -i '/net.ipv4.icmp_echo_ignore_broadcasts/d' /etc/sysctl.conf
echo "net.ipv4.icmp_echo_ignore_broadcasts=1" >> /etc/sysctl.conf

# ICMP Redirect パケットは拒否
sed -i '/net.ipv4.conf.*.accept_redirects/d' /etc/sysctl.conf
for dev in `ls /proc/sys/net/ipv4/conf/`
do
    sysctl -w net.ipv4.conf.$dev.accept_redirects=0 > /dev/null
    echo "net.ipv4.conf.$dev.accept_redirects=0" >> /etc/sysctl.conf
done

# Source Routed パケットは拒否
sed -i '/net.ipv4.conf.*.accept_source_route/d' /etc/sysctl.conf
for dev in `ls /proc/sys/net/ipv4/conf/`
do
    sysctl -w net.ipv4.conf.$dev.accept_source_route=0 > /dev/null
    echo "net.ipv4.conf.$dev.accept_source_route=0" >> /etc/sysctl.conf
done

# 全ホスト(ブロードキャストアドレス、マルチキャストアドレス)宛パケットはログを記録せずに破棄
iptables -A INPUT -d 255.255.255.255 -j DROP
iptables -A INPUT -d 224.0.0.1 -j DROP

# 必要なポートは下記
# 外部からの TCP 22番ポート( SSH )へのアクセスを許可
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# サーバー再起動時にも上記設定が有効となるようにルールを保存
/etc/rc.d/init.d/iptables save

# ファイアウォール起動
/etc/rc.d/init.d/iptables start
```
