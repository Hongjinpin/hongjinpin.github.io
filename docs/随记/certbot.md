# certbot 使用介绍

原文地址：https://www.seanzhao.me/2018/06/21/certbot-manual.html

## 介绍

certbot是专门为Let’s encrypt制作的一个管理证书工具，可以通过它来生成证书管理更新Let’s encrypt证书。`它们都是免费的。`大多数情况下，需要在服务器上面有root权限才可以允许certbot管理证书。

### 安装

直接访问[certbot.eff.org](https://certbot.eff.org/)的首页选择服务器环境和系统就可以显示出安装说明

#### 系统要求

Certbot目前需要在像UNIX的操作系统上运行Python 2.7或3.4+。默认情况下，它需要以写root访问权限 `/etc/letsencrypt`，`/var/log/letsencrypt`，`/var/lib/letsencrypt`，绑定到端口80和443（如果您使用的standalone插件），并读取和修改Web服务器配置（如果使用apache或nginx 插件）。如果这些都不适用于您，理论上可以在没有root权限的情况下运行，但对于大多数希望避免以root身份运行ACME客户端的用户，[letsencrypt-nosudo](https://github.com/diafygi/letsencrypt-nosudo)或[simp_le](https://github.com/zenhack/simp_le)是更合适的选择。

#### 离线安装

    user @ webserver：〜$ wget https://dl.eff.org/certbot-auto
    user @ webserver：〜$ chmod a + x ./certbot-auto
    user @ webserver：〜$ ./certbot-auto --help


在很多情况下，您可以运行`certbot-auto`或者`certbot`，客户端将引导您完成交互式获取和安装证书的过程。

如果需要帮助可以输入
`./certbot-auto --help all`
您还可以从命令行准确地告诉它要做什么。例如，如果您想获得example.com、www.example.com和other.example.net的证书，可以使用Apache插件获取和安装证书，您可以这样做:

例如，如果想要获得example.com、www.example.com 和 other.example.net的证书，使用certbot自带的apache工具获取和安装：
`certbot --apache -d example.com -d www.example.com -d other.example.net`
如果第一次允许该命令，将会在执行过程中要求输入邮箱等信息。该命令会自动查找apache的配置文件并且修改配置文件支持https。

如果想要收到配置apache设置可以使用`certonly`子命令
`sudo certbot --apache certonly`

注意：具有certonly的apache插件会执行以下操作

1. 进行临时配置更改
2. 执行重新加载
3. 恢复所有更改
4. 执行另一个重新加载
这似乎是一个可靠的过程，但是如果您不希望Certbot以任何方式触摸您的Apache进程或文件，则可以改为使用 [webroot插件](https://certbot.eff.org/docs/using.html#webroot)。
  
## 命令介绍

提示：如果您的系统使用较旧的软件包，或者您使用了备用安装方法，则certbotWeb服务器上的脚本可能会被命名。在整个文档中，只要你看到，根据需要交换正确的名称。`letsencrypt``certbot-auto``certbot`

| Plugin      | Auth | Inst | 描述                                                         | 类型和端口\(challenge\-type\)                        |
| ----------- | ---- | ---- | ------------------------------------------------------------ | ---------------------------------------------------- |
| apache      | y    | y    | 使用Apache自动获取和安装证书2\.4在1\.0以上的操作系统上libaugeas0。 | tls\-sni\-01（443）                                  |
| webroot     | y    | n    | 过写入到webroot目录获得证书个已经运行的web服务器。           | http\-01（80）                                       |
| nginx       | y    | y    | 使用Nginx自动获取和安装证书。用Certbot 0\.9\.0发货。         | tls\-sni\-01（443）                                  |
| standalone  | y    | n    | 用“独立”网络服务器获取证书。求端口80或443可用。这很有用有网络服务器的系统，或直接与系统集成地网络服务器不受支持或不受欢迎。 | http\-01（80）或 tls\-sni\-01（443）                 |
| DNS plugins | y    | n    | 类插件可以通过自动获取证书改DNS记录以证明您有控制权。以这种方式进行域验证是Let's获得通配符证书的唯一方法密。 | dns\-01（53）                                        |
| manual      | y    | n    | 过向您提供说明帮助您获得证书己执行域验证。另外允许你定脚本以自动执行a中的验证任务制的方式。 | http\-01（80）， dns\-01（53）或 tls\-sni\-01（443） |


插件使用几种ACME协议验证你是否有该域名的所有权。选项是http-01（使用端口80）， tls-sni-01（端口443）和dns-01（需要在端口53上配置DNS服务器，尽管这通常与Web服务器不同）。少数插件支持多种验证类型，在这种情况下，您可以选择一种`--preferred-challenges`。

也有许多[第三方插件](https://certbot.eff.org/docs/using.html#third-party-plugins)可用。下面我们更详细地描述每个插件如何使用以及使用情况。

### apache

Apache插件目前需要安装有augeas 1.0版的操作系统; 目前它支持 基于Debian，Fedora，SUSE，Gentoo和Darwin的现代操作系统。这可以在Apache Web服务器上自动获取和安装证书。要在命令行上指定使用此插件，只需指定 `--apache`如`certbot --apache`。

### Webroot

如果你的服务器正在运行，而您必须修改配置，你不希望在制作证书过程中停止Web服务器，您可以使用Web根目录插件通过包括获取证书`certonly`和`--webroot`在命令行上。并且需要指定`--webroot-path` 或`-w`参数包含网站的根目录。例如， 或者是两个常见的webroot路径。`--webroot-path /var/www/html``--webroot-path /usr/share/nginx/html`
示例：
`certbot certonly --webroot -w /var/www/example -d www.example.com -d example.com -w /var/www/other -d other.example.net -d another.other.example.net`
前面两个域名使用`/var/www/example`目录后面两个域名使用`/var/www/other`目录

webroot工作的原理就是在每个网站根目录创建一个临时文件`${webroot-path}/.well-known/acme-challenge`，然后Let’s encrypt服务器使用HTTP请求验证。类似google、百度的站长验证一样

### nginx

我们建议在使用之前备份Nginx配置（尽管您也可以将配置恢复为配置）。您可以通过在命令行上提供标志来使用它。`certbot --nginx rollback--nginx`
`certbot --nginx`

### Standalone

如果您不想使用（或当前没有）现有服务器软件，请使用独立模式获取证书。独立模式不依赖获取证书的服务器和服务器软件。
如果使用`Standalone`来获得证书需要加上`certonly`和`--standalone`并且`80端口`或者`443端口`所有这两个端口必须有一个是空闲可占用的才行。

如果要指定使用那一个端口，可以通过命令：
`--preferred-challenges http` 使用端口80
`--preferred-challenges tls-sni` 使用端口443

默认情况下，Certbot首先尝试绑定到IPv6的端口，然后绑定到IPv4该端口; 只要至少有一个绑定成功，Certbot就会继续。在大多数Linux系统上，IPv4流量将被路由重定向到IPv6端口，并且期望在第二次绑定期间出现故障。
使用`--<challenge-type>-address`明确地告诉Certbot接口（和协议）。

### DNS Plugins

如果您希望从Let’s Encrypt获取通配符证书或certbot在除目标Web服务器之外的计算机上运行 ，则可以使用Certbot的一个DNS插件。

这些插件仍处于被许多发行版打包的过程中，目前无法安装certbot-auto。但是，如果您愿意自己安装证书，则可以使用Docker运行这些插件。

安装完成后，您可以在以下网址找到有关如何使用每个插件的文档：

[certbot-DNS-cloudflare](https://certbot-dns-cloudflare.readthedocs.io/)

[certbot-DNS-cloudxns](https://certbot-dns-cloudxns.readthedocs.io/)

[certbot-DNS-digitalocean](https://certbot-dns-digitalocean.readthedocs.io/)

[certbot-DNS-dnsimple](https://certbot-dns-dnsimple.readthedocs.io/)

[certbot-DNS-dnsmadeeasy](https://certbot-dns-dnsmadeeasy.readthedocs.io/)

[certbot-DNS-google](https://certbot-dns-google.readthedocs.io/)

[certbot-dns-luadns](https://certbot-dns-luadns.readthedocs.io/)

[certbot-dns-nsone](https://certbot-dns-nsone.readthedocs.io/)

[certbot-dns-rfc2136](https://certbot-dns-rfc2136.readthedocs.io/)

[certbot-dns-route53](https://certbot-dns-route53.readthedocs.io/)

### Manual

如果需要获得在目标服务器之外允许的证书或者自己执行验证步骤则可以使用手动插件`Manual`通过指定`certonly`和`--manual`获取证书。

Manual验证可以使用`http`,`dns`或者`tls-sni`验证。可以使用该`--preferred-challenges`选项来选择验证方式。
该`http`验证要求您在网站的根目录放置在一个特定的名称和具体内容的文件`/.well-known/acme-challenge/`。本质上它与`webroot`插件相同，但不是自动的。

使用`dns`验证时，`certbot`需要在颁发证书的域名的DNS上面添加一条TXT记录，前缀为`_acme-challenge`。

例如：域名为`example.com`

TXT记录为`_acme-challenge.example.com. 300 IN TXT "gfj9Xq...Rg85nM"`

使用`tls-sni`验证时，certbot将为您准备一份自签名的SSL证书，并将检测的验证编码到`subjectAlternatNames`条目中。您将需要配置您的SSL服务器以使用SNI将此验证SSL证书呈现给ACME服务器。

### 结合插件使用

有时您可能想要指定不同的验证方式和安装程序插件的组合。要做到这一点，说明与认证插件 `--authenticator`或`-a`与安装插件`--installer`或 `-i`。

例如，您可能希望使用`webroot`插件进行身份验证并使用`apache`插件来安装证书，这可能是因为您使用代理或CDN进行SSL，并且只想确保它们与原始服务器之间的连接不能使用由于中间代理的tls-sni-01验证。
`certbot run -a webroot -i apache -w /var/www/html -d example.com
`

### 管理证书

要查看Certbot知道的证书列表，请运行certificates子命令：
`certbot certificates`
`Certificate Name`显示证书的名称。使用`--cert-name`子命令来指定一个特定的证书进行操作`run`， `certonly`，`certificates`，`renew`，和`delete`命令。例：
`certbot certonly --cert-name example.com`

#### 重新创建和更新现有证书

即使您已经拥有一些证书，也可以使用子命令`certonly`或`run`子命令来请求创建单个新证书。

如果请求证书`run`或`certonly`指定已存在的证书名称，Certbot会更新现有证书。否则，将创建一个新证书并分配指定的名称。`一句话存在就更新，不存在就创建`。

使用`--force-renewal`，`--duplicate`和`--expand`选项在重新创建已经存在的证书时，控制Certbot的行为。如果你没有指定请求的行为，Certbot可能会问你你想要的。该选项默认包含了 –expand 选项的功能。(默认: False)
`--force-renewal`告诉Certbot请求更新已经存在的证书时。每个域名必须通过指定`-d`。如果成功，此证书将与较早的证书一起保存，并且通过符号链接将更新为指向新证书。这是更新特定单个证书的有效方法(这是直接更新，不过是否将要过期)。
`--duplicate`告诉Certbot创建一个已经存在的证书时。该证书与前一份完全分开保存。大多数用户在正常情况下不需要发出这个命令。
`--expand`告诉Certbot使用包含所有旧域名和一个或多个额外新域的新证书更新现有证书。使用该`--expand`选项，使用该`-d`选项可以指定所有现有域名和一个或多个新域名。(默认: 询问)

例如：`certbot --expand -d existing.com，example.com，newdomain.com`也可以`certbot --expand -d existing.com -d example.com -d newdomain.com`

推荐使用`--cert-name`而不是`--expand`，因为它可以更好地控制哪个证书被修改，它可以让你删除域名以及添加它们。
`--allow-subset-of-names`如果只有部分指定的域名可以获得授权，则告诉Certbot继续生成证书。如果证书中指定的某些域不再指向该服务器，这可能很有用。

不管什么时候，采用这些方式获取的新的证书后就的证书不会被删除掉。

### 更改证书的域名

使用`--cert-name`选项还可用于通过使用`-d`或`--domains`选项指定新域名来修改或者移除证书所包含的域名。如果证书`example.com` 以前包含`example.com`和`www.example.com`，它可以被修改仅包含`example.com`通过仅指定`example.com`与`-d`或`--domains`标志。例：
`certbot certonly --cert-name example.com -d example.com`

可以使用如下格式来扩展该证书的域名集或者替换该证书的域名集
`certbot certonly --cert-name example.com -d example.org,www.example.org`

#### 注销证书

如果您的账户密钥已被盗用或您需要撤销证书，请使用该`revoke`命令来执行此操作。请注意，该`revoke`命令采用证书路径（以结尾cert.pem），而不是证书名称或域名。例：
`certbot revoke --cert-path /etc/letsencrypt/live/CERTNAME/cert.pem`

您还可以使用该`reason`标志指定撤消证书的原因。原因包括：`unspecified`这是默认的，以及`keycompromise`， `affiliationchanged`，`superseded`，和`cessationofoperation`：
`certbot revoke --cert-path /etc/letsencrypt/live/CERTNAME/cert.pem --reason keycompromise`

此外，如果证书是通过`--staging`或`--test-cert`选项获得的测试证书，则该选项必须传递给 `revoke`子命令。
`certbot revoke --cert-path /etc/letsencrypt/live/CERTNAME/cert.pem --staging`

或者：
`certbot revoke --cert-path /etc/letsencrypt/live/CERTNAME/cert.pem --test-cert`
一旦证书被吊销（或其他证书管理任务），可以使用`delete`子命令从系统中删除所有证书的相关文件：
`certbot delete --cert-name example.com`

如果您不使用`delete`完全删除证书，它将在证书更新时自动更新。
撤销证书不会影响Let’s Encrypt服务器施加的速率限制。

#### 更新证书

让我们加密CA发布短期证书（90天）。确保您在3个月内至少续签一次证书。
从版本0.10.0开始，Certbot支持`renew`检查所有已安装的证书是否即将到期并尝试对其进行更新。最简单的形式很简单
`certbot renew``renew`命令会检查所有的证书并更新`30天内过期`的证书（如果不指定使用的插件和选项，例如申请证书时使用`apache`那么更新时还说该参数，如果想要改变在后面跟上其它参数即可），由于`renew`只更新快过期的证书，所以可以随意的使用。
`renew`命令可以使用hook钩子，例如：如果使用`webroot`方式，那么在更新前需要关闭服务器释放80端口在更新后需要启动服务器。

certbot renew –pre-hook “service nginx stop” –post-hook “service nginx start”
`--pre-hook`在尝试更新前调用
`--post-hook`在更新之后调用，不论成功或者失败
`--deploy-hook`在更新成功后调用

例子：在更新成功后调用脚本 
`certbot renew --deploy-hook /path/to/deploy-hook-script`

如果存在一个守护程序不是以`root`的身份读取证书，那么可以使用`--deploy-hook`钩子将其复制到正确的位置并应用适当的文件权限。
示例脚本：

```shell
#!/bin/sh

set -e

for domain in $RENEWED_DOMAINS; do
        case $domain in
        example.com)
                daemon_cert_root=/etc/some-daemon/certs

                # Make sure the certificate and private key files are
                # never world readable, even just for an instant while
                # we're copying them into daemon_cert_root.
                umask 077

                cp "$RENEWED_LINEAGE/fullchain.pem" "$daemon_cert_root/$domain.cert"
                cp "$RENEWED_LINEAGE/privkey.pem" "$daemon_cert_root/$domain.key"

                # Apply the proper file ownership and permissions for
                # the daemon to read its certificate and key.
                chown some-daemon "$daemon_cert_root/$domain.cert" \
                        "$daemon_cert_root/$domain.key"
                chmod 400 "$daemon_cert_root/$domain.cert" \
                        "$daemon_cert_root/$domain.key"

                service some-daemon restart >/dev/null
                ;;
        esac
done
```

如果在某些钩子下有多个需要执行的脚本，还可以通过将文件放在Certbot的配置目录的子目录中来执行挂钩。假设您的配置目录是 `/etc/letsencrypt`，当使用子命令更新任何证书时，将分别在更新前执行`pre`目录中的脚本`/etc/letsencrypt/renewal-hooks/pre`，和更新成功后`deploy`目录中的脚本` /etc/letsencrypt/renewal-hooks/deploy`这些钩子按字母顺序运行，不会为其他子命令运行。（挂钩运行的顺序由其文件名中字符的字节值决定，并且不依赖于您的语言环境。）

有关钩子的更多信息可以通过运行`certbot --help renew`找到。
如果执行过程中出现错误（返回非0的状态码）那么错误会被输出到`stderr`。可以在后面加上`-q`或者`--quiet`来使除错误以外的所有输出保持沉默。
注意，在更新证书中指定的选项会应用到说以的证书上面，如果更新成功将会保存该选项并将在下次更新时再次使用。`certbot renew``certbot renew --rsa-key-size 4096`

一般情况下更新证书使用的cron计划任务定时更新，所以在更新过程中不想出现任何需要用户输入的参数，可以使用`-n`或者`--noninteractive`
`certbot certonly -n -d example.com -d www.example.com`

这种情况下必须指定证书包涵的所有域名，以便更新证书，如果域名不一致将会替换证书。

如果证书快到期并且没有更新CA将会给提供的邮箱发电子邮件。

#### 修改更新配置文件

在颁发证书时，默认情况下会创建一个续订的配置文件，用于续订时使用和颁发证书时相同的选项，配置文件位于`/etc/letsencrypt/renewal/CERTNAME`

如果不是必要不建议修改续订配置文件，如果手动修改后，建议通过`certbot renew --dry-run`进行测试配置文件的有效性

如果将续订配置文件`/etc/letsencrypt/archive/CERTNAME`移动到新文件夹，请先在续订配置文件中指定新文件夹的名称，然后运行以将符号链接指向新文件夹。
`certbot update_symlinks /etc/letsencrypt/live/CERTNAME`
如果需要修改生成证书的目录或者修改过期证书的目录等等需要调用`certbot update_symlinks`更新

例如：证书的续订配置文件包含以下指令

```shell
archive_dir = /etc/letsencrypt/archive/example.com  
cert = /etc/letsencrypt/live/example.com/cert.pem  
privkey = /etc/letsencrypt/live/example.com/privkey.pem  
chain = /etc/letsencrypt/live/example.com/chain.pem  
fullchain = /etc/letsencrypt/live/example.com/fullchain.pem  
```


更改位置：

```shell
mv /etc/letsencrypt/archive/example.com /home/user/me/certbot/example_archive
sed -i 's,/etc/letsencrypt/archive/example.com,/home/user/me/certbot/example_archive,' /etc/letsencrypt/renewal/example.com.conf
mv /etc/letsencrypt/live/example.com/*.pem /home/user/me/certbot/
sed -i 's,/etc/letsencrypt/live/example.com,/home/user/me/certbot,g' /etc/letsencrypt/renewal/example.com.conf
certbot update_symlinks
```


#### 自动续订

使用cron计划任务定时执行

### 证书位置

所有生成的密钥和颁发的证书都可以在中找到 /etc/letsencrypt/live/$domain。请勿将您的（网络）服务器配置直接指向这些文件（或创建符号链接），而不要复制。更新期间，/etc/letsencrypt/live会更新最新的必要文件。
`/etc/letsencrypt/archive`和`/etc/letsencrypt/keys`包含所有以前的密钥和证书，而 `/etc/letsencrypt/live`符号链接到最新版本。

#### 生成的文件

`privkey.pem` 私钥文件，必须严密保存，这就是Apache需要的[SSLCertificateKeyFile](https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatekeyfile)，以及Nginx需要的[ssl_certificate_key](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key)。
`fullchain.pem` 证书，这就是Apache> = 2.4.8所需的[SSLCertificateFile](https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatefile)，以及Nginx需要的[ssl_certificate](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate)。
`cert.pem`和`chain.pem`（不太常见）
`cert.pem`包含服务器证书和`chain.pem`中间证书（证书链）必须同时使用才有效
Apache <2.4.8需要这些用于[SSLCertificateFile](https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatefile)。和[SSLCertificateChainFile](https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatechainfile)。如果使用Nginx> = 1.3.7的OCSP装订，`chain.pem`应该提供[ssl_trusted_certificate](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_trusted_certificate) 来验证OCSP响应。

所有文件都是PEM编码的。如果您需要其他格式，例如DER或PFX，那么您可以使用转换`openssl`。在更新完成后通过触发钩子`--deploy-hook`调用`openssl`进行转化。

### manual验证前后钩子

在`Manual`模式运行时有两个钩子验证前和验证后会触发`--manual-auth-hook`和`--manual-cleanup-hook`
`certbot certonly --manual --manual-auth-hook /path/to/http/authenticator.sh --manual-cleanup-hook /path/to/http/cleanup.sh -d secure.example.com`
将会传递以下变量给脚本：
`CERTBOT_DOMAIN`: 被认证的域名
`CERTBOT_VALIDATION `:验证字符串（仅限HTTP-01和DNS-01）
`CERTBOT_TOKEN`: HTTP-01验证的资源名称部分（仅限HTTP-01）
`CERTBOT_CERT_PATH`: 挑战SSL证书（仅限TLS-SNI-01）
`CERTBOT_KEY_PATH`: 与上述SSL证书关联的私钥（仅限TLS-SNI-01）
`CERTBOT_SNI_DOMAIN`: ACME服务器希望为其提供位于`$CERTBOT_CERT_PATH`（仅限TLS-SNI-01）的自签名证书的SNI名称

cleanup 输出：
`CERTBOT_AUTH_OUTPUT`: 无论如何auth脚本写入标准输出
`certbot certonly --manual --preferred-challenges=http --manual-auth-hook /path/to/http/authenticator.sh --manual-cleanup-hook /path/to/http/cleanup.sh -d secure.example.com`

HTTP-01的示例用法：
`certbot certonly --manual --preferred-challenges=http --manual-auth-hook /path/to/http/authenticator.sh --manual-cleanup-hook /path/to/http/cleanup.sh -d secure.example.com`

/path/to/http/authenticator.sh

```shell
#!/bin/bashecho$CERTBOT_VALIDATION> /var/www/htdocs/.well-known/acme-challenge/$CERTBOT_TOKEN
```

/path/to/http/cleanup.sh

```shell
#!/bin/bash
rm -f /var/www/htdocs/.well-known/acme-challenge/$CERTBOT_TOKEN
```

DNS-01（Cloudflare API v4）的示例用法（仅用于示例目的，请勿按原样使用）
`certbot certonly --manual --preferred-challenges=dns --manual-auth-hook /path/to/dns/authenticator.sh --manual-cleanup-hook /path/to/dns/cleanup.sh -d secure.example.com`
/path/to/dns/authenticator.sh

```shell
#!/bin/bash

# Get your API key from https://www.cloudflare.com/a/account/my-account
API_KEY="your-api-key"
EMAIL="your.email@example.com"

# Strip only the top domain to get the zone id
DOMAIN=$(expr match "$CERTBOT_DOMAIN" '.*\.\(.*\..*\)')

# Get the Cloudflare zone id
ZONE_EXTRA_PARAMS="status=active&page=1&per_page=20&order=status&direction=desc&match=all"
ZONE_ID=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones?name=$DOMAIN&$ZONE_EXTRA_PARAMS" \
     -H     "X-Auth-Email: $EMAIL" \
     -H     "X-Auth-Key: $API_KEY" \
     -H     "Content-Type: application/json" | python -c "import sys,json;print(json.load(sys.stdin)['result'][0]['id'])")

# Create TXT record
CREATE_DOMAIN="_acme-challenge.$CERTBOT_DOMAIN"
RECORD_ID=$(curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
     -H     "X-Auth-Email: $EMAIL" \
     -H     "X-Auth-Key: $API_KEY" \
     -H     "Content-Type: application/json" \
     --data '{"type":"TXT","name":"'"$CREATE_DOMAIN"'","content":"'"$CERTBOT_VALIDATION"'","ttl":120}' \
             | python -c "import sys,json;print(json.load(sys.stdin)['result']['id'])")
# Save info for cleanup
if [ ! -d /tmp/CERTBOT_$CERTBOT_DOMAIN ];then
        mkdir -m 0700 /tmp/CERTBOT_$CERTBOT_DOMAIN
fi
echo $ZONE_ID > /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID
echo $RECORD_ID > /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID

# Sleep to make sure the change has time to propagate over to DNS
sleep 25
```


/path/to/dns/cleanup.sh

```shell
#!/bin/bash

# Get your API key from https://www.cloudflare.com/a/account/my-account
API_KEY="your-api-key"
EMAIL="your.email@example.com"

if [ -f /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID ]; then
        ZONE_ID=$(cat /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID)
        rm -f /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID
fi

if [ -f /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID ]; then
        RECORD_ID=$(cat /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID)
        rm -f /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID
fi

# Remove the challenge TXT record from the zone
if [ -n "${ZONE_ID}" ]; then
    if [ -n "${RECORD_ID}" ]; then
        curl -s -X DELETE "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID" \
                -H "X-Auth-Email: $EMAIL" \
                -H "X-Auth-Key: $API_KEY" \
                -H "Content-Type: application/json"
    fi
fi
```

### 更改ACME服务器

默认情况下，Certbot使用在加密的初始生产服务器 [https://acme-v01.api.letsencrypt.org/](https://acme-v01.api.letsencrypt.org/)。您可以通过`--server`在命令行或[配置文件](https://certbot.eff.org/docs/using.html#config-file)提供服务器的ACME目录的URL 来告诉Certbot使用不同的CA服务器. 例如，如果您想使用Let’s Encrypt的新ACMEv2服务器，则可以使用命令行执行`--server https://acme-v02.api.letsencrypt.org/directory`。Certbot将根据URL提供的内容自动选择要使用的ACME协议版本。

如果您使用`--server`指定实现新版本规范的ACME CA，则可以获得通配符域的证书。有些CA（例如Let’s Encrypt）要求通配符域的域验证必须通过修改DNS记录来完成，这意味着必须使用[dns-01](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-7.4)验证类型。要查看支持该挑战类型的Certbot插件列表以及如何使用它们，请参阅[插件](https://certbot.eff.org/docs/using.html#plugins)。

### 锁定文件

处理验证时，Certbot会在您的系统上写入多个锁定文件，以防止多个实例覆盖彼此的更改。这意味着默认情况下，Certbot的两个实例将不能并行运行。 

由于Certbot使用的目录是可配置的，Certbot将为它使用的所有目录编写一个锁定文件。这包括Certbot的 `--work-dir`，`--logs-dir`和`--config-dir`。默认情况下，这些都是 `/var/lib/letsencrypt`，`/var/logs/letsencrypt`和`/etc/letsencrypt` 。另外，如果您使用带有Apache或nginx的Certbot，它将锁定该程序的配置文件夹，这通常也位于 `/etc`目录中。

请注意，这些锁定文件只会阻止其他Certbot实例使用这些目录，而不会阻止其他进程。如果你想同时运行多个Certbot实例应为每个Certbot指定不同的目录中`--work-dir`，`--logs-dir`和`--config-dir`。

### 配置文件

Certbot接受全局配置文件，将其选项应用于Certbot的所有调用。应在`.conf`里面找到的文件中设置证书特定的配置选项`/etc/letsencrypt/renewal`。

默认情况下，不会创建`cli.ini`文件，创建一个文件后可以指定该配置文件的位置）。示例配置文件如下所示：`certbot --config`或者 `certbot cli.ini-c cli.ini`

```ini
 This is an example of the kind of things you can do in a configuration file.
# All flags used by the client can be configured here. Run Certbot with
# "--help" to learn more about the available options.
# 客户端使用的所有标志都可以在这里配置 --help 了解更多信息
#
# Note that these options apply automatically to all use of Certbot for
# obtaining or renewing certificates, so options specific to a single
# certificate on a system with several certificates should not be placed
# here.
# 这些选项会应用于所有的certbot来获取或者更新证书。因此，不应在这里放置特定于具有多个证书的系统上的单个
＃证书的选项

# Use a 4096 bit RSA key instead of 2048
# 使用4096位RSA密钥而不是2048
rsa-key-size = 4096

# Uncomment and update to register with the specified e-mail address
# 取消注册并更新以注册指定的电子邮件地址
# email = foo@example.com

# Uncomment to use the standalone authenticator on port 443
# 取消在端口443上使用独立验证器的注释
# authenticator = standalone
# standalone-supported-challenges = tls-sni-01

# Uncomment to use the webroot authenticator. Replace webroot-path with the
# path to the public_html / webroot folder being served by your web server.
# 取消注释以使用webroot身份验证器。将webroot-path替换为由您的Web服务器提供的public_html / webroot文件夹
的＃路径
# authenticator = webroot
# webroot-path = /usr/share/nginx/html
```

默认情况下，搜索以下位置：
`/etc/letsencrypt/cli.ini`
`$XDG_CONFIG_HOME/letsencrypt/cli.ini`（如果$XDG_CONFIG_HOME没有设置：`~/.config/letsencrypt/cli.ini`）。
由于此配置文件适用于certbot的所有调用，所有不能在其中列出域名。在cli.ini中列出域名可能会阻止续订工作。

### 日志循环

默认情况下，certbot存储状态日志`/var/log/letsencrypt`。默认情况下，一旦日志目录中有1000个日志，certbot就会开始旋转日志。这意味着一旦有1000个文件在`/var/log/letsencryptCertbot`中，将删除最旧的文件，为新日志腾出空间。通过`--max-log-backups`更改日志数量。

包括Debian和Ubuntu在内的一些发行版会禁用certbot的内部日志循环，以支持更传统的logrotate脚本。如果您使用的是发行包，并且想要更改日志轮转，请检查/etc/logrotate.d/certbot轮播脚本。