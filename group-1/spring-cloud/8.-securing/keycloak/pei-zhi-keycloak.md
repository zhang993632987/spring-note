# 配置 Keycloak

## Creating an realm

Realm（领域）是Keycloak中用来管理一组用户、凭据、角色和组的对象的概念。

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## 注册客户端应用

在Keycloak中，客户端是可以请求用户身份验证的实体。通常，客户端是我们希望通过提供单一登录（SSO）解决方案来保护的应用程序或服务。

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### 添加客户端角色

在“**Roles**”页面上，我们需要创建以下客户端角色：

* USER
* ADMIN

<figure><img src="../../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### Credentials page

既然我们已经完成了基本的客户端配置，让我们访问“**Credentials**”页面。该页面将显示身份验证过程所需的客户端密钥：

<figure><img src="../../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

## 创建 realm roles(optional)

这是一个可选步骤。如果你不想创建这些角色，可以直接创建用户。但是之后要识别和维护每个用户的角色可能会更困难。

<figure><img src="../../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## 创建用户

<figure><img src="../../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

保存这个表单后，点击“**Credentials**”选项卡。您需要输入用户的密码，禁用“**Temporary**”选项，然后点击“**Save Password**”按钮。密码设置好后，点击“**Role Mappings**”选项卡，为用户分配特定的角色。
