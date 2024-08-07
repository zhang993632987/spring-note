# Git

## 1. 修改 application.yml

<pre class="language-yaml"><code class="lang-yaml">spring:
  application:
    name: config-server
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri:
            https://gitee.com/zhang993632987/config.git
          search-paths: '{application}'
          default-label: master
<strong>          refresh-rate: 0
</strong><strong>          deleteUntrackedBranches: true
</strong><strong>          basedir: file:///F:/tmp
</strong></code></pre>

*   **spring.profiles.active=git**

    <mark style="color:blue;">**使用 git 作为后端**</mark>，存储配置信息。
*   spring.cloud.config.server.git.search-locations={application}

    <mark style="color:blue;">**指定搜索路径**</mark>，可以使用占位符 **{application}**、**{profile}** 和 **{label}**，其中：

    * **application** 表示客户端应用的名称（**spring.application.name**）
    * **profile** 为客户端的环境（**spring.profiles.active**）
    * **label** 指示分支版本（**spring.cloud.config.label**）
*   **spring.cloud.config.server.git.uri=https://gitee.com/zhang993632987/config**&#x20;

    <mark style="color:blue;">**仓库地址**</mark>

    {% hint style="info" %}
    可以使用本地仓库，例如：file:///F:\self-learning\config-local，该地址必须是一个本地 git 仓库，需要经过下述命令处理：

    ```bash
    git init
    git add *
    git commit -m "message"
    ```

    注意：只有 commit 后的文件才能被获取！
    {% endhint %}
*   spring.cloud.config.server.git.default-label=master

    <mark style="color:blue;">**默认分支（branch）**</mark>
*   spring.cloud.config.server.git.refresh-rate=0

    <mark style="color:blue;">**刷新本地配置的频率**</mark>，单位为秒。“0” 代表每次客户端请求配置信息时，均会请求 git 仓库。
*   &#x20;spring.cloud.config.server.git.deleteUntrackedBranches=true

    在 Config Server 启动时，会将指定的 git 仓库克隆到本地，本地仓库会一直存在直到 Config Server 重启。deleteUntrackedBranches 表示 Config Server 会<mark style="color:blue;">**强制删除 untracked 的分支**</mark>。
*   spring.cloud.config.server.git.basedir=file:///F:/tmp

    <mark style="color:blue;">**远程仓库克隆下来后的存储地址**</mark>，默认存在临时文件夹中，可以指定也可以不指定。

## 2. git 仓库

<figure><img src="../../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

## 3. 认证

对于私有的远程仓库，需要添加认证信息才能连接上，此时可以采用“**私人令牌**”的方式进行用户认证。使用“私人令牌”可以控制令牌拥有的权限，防止账户密码泄露：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri:
            https://gitee.com/zhang993632987/config
          default-label: master
          refresh-rate: 0
          search-paths: '{application}'
          deleteUntrackedBranches: true
          username: '17336538193'
          password: 'bea83aa8df4a3355d06ab2837c97fa4c'
```
