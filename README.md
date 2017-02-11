# Vagrant + CentOS 6.6でロードバランサの環境構築

## 各モジュールなどのバージョン
* Vagrant 1.8.7
* CentOS 6.6
 *  <https://github.com/tommy-muehle/puppet-vagrant-boxes/releases/download/1.0.0/centos-6.6-x86_64.box>
* ロードバランサのサーバ （2台構成）
 * keepalived 1.2.13-5.el6_6
 * ipvsadm 1.26-4.el6
* Webサーバ（2台構成）
 * httpd 2.2.15-56.el6.centos.3

## ネットワークの構成
以下のイメージ参照

![ネットワーク構成](/images/image1.png)

※この構成によって、LVSが1台死んでも、Webサーバが1台死んでもOK！

## ロードバランサの環境構築手順

Gitで管理されているVagrantfileをC:\VM\lvs-sampleに配置した状態で手順を進める。<br>
※`CentOS66`は上記記載のモジュールでCentOS6.6をvagrant box add した名前。

1. `C:\VM\lvs-sample>vagrant up`

1. TeraTermなどを使用して lvs1 へアクセスし、keepalivedとipvsadmをyumでインストールする<br>
`yum -y install keepalived ipvsadm`

1. /etc/keepalived/keepalived.confを以下の内容に修正する。<br>
※lvs1もlvs2もどちらも同じ内容で問題ない。
<pre>
<code>
  global_defs {
     router_id LVS_SAMPLE
  }
  vrrp_instance VI_1 {
      state BACKUP
      interface eth0
      virtual_router_id 1
      priority 100
      advert_int 5
      nopreempt
      authentication {
          auth_type PASS
          auth_pass sansou
      }
      virtual_ipaddress {
          192.168.33.254
      }
  }
  virtual_server 192.168.33.254 80 {
      delay_loop 1
      lb_algo rr
      lb_kind DR
      nat_mask 255.255.255.0
      protocol TCP
      real_server 192.168.33.101 80 {
          weight 1
          HTTP_GET {
              url {
                path /
              }
              connect_timeout 3
              nb_get_retry 3
              delay_before_retry 3
          }
      }
      real_server 192.168.33.102 80 {
          weight 1
          HTTP_GET {
              url {
                path /
              }
              connect_timeout 3
              nb_get_retry 3
              delay_before_retry 3
          }
      }
  }
</code>
</pre>

1. /etc/sysctl.confの修正を行う。<br>
 1. net.ipv4.ip_forwardの値を1に修正／2つの定義を追加<br>
<pre><code>net.ipv4.ip_forward = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 =1</code></pre>

 1. sysctl.confの修正を反映する。
 `/sbin/sysctl -p`

1. iptablesの設定
<pre><code>iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -I INPUT -p vrrp -j ACCEPT
service iptables save</code></pre>

1. keepalivedのサービス起動
<pre><code>service keepalived start
chkconfig keepalived on</code></pre>

1. 上記がlvs1/lvs2の手順で、あとはweb1/web2にhttpdをyumでインストールしてサービス起動すれば準備OK<br>
一応 /var/www/html/index.html を作成して、web1とweb2のどちらのindex.htmlにアクセスされているか判別できるようにしておくとよい。

1. あと、web1/web2のサーバで以下のコマンドを発行してiptablesの設定をしておくこと
<pre><code>iptables -t nat -I PREROUTING -d 192.168.33.254 -j REDIRECT
iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
service iptables save</code></pre>


## 動作確認方法

1. lvs1/lvs2/web1/web2が起動した状態にする

1. 以下のURLに何度もアクセスする<br>
http://192.168.33.254/

1. ロードバランサの設定`lb_algo rr`でラウンドロビンにしているため、毎回アクセスするたびに接続先のWebサーバが変化していることに気付く。

1. あとは、片方のlvsサーバを停止`vagrant halt`したり、webサーバを停止したりしても正常に動作することを確認するなどいろいろ。

## 今後

play2フレームワークを使って、クライアントサイドセッション管理が正常に動作し、どちらのサーバへ接続している状態であってもセッションの中身が取得できるかどうか検証したいと思います。
