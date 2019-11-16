# 配置npm以用于GitHub Packages

您可以将npm配置为将包发布到GitHub包，并将存储在GitHub包中的包用作npm项目中的依赖项。

{% hint style="info" %}
GitHub软件包可与GitHub免费版、GitHub专业版、GitHub团队和GitHub企业云一起使用。有关更多信息，请参见“GitHub的产品”
{% endhint %}

您需要一个access token来发布、安装和删除GitHub Packages中的包。您可以使用个人访问令牌personal access token以您的用户名直接向GitHub软件包或GitHub API进行身份验证。您可以使用GITHUB\_TOKEN使用 GitHub Actions 工作流进行身份验证。

要使用个人访问令牌personal access token进行身份验证，您必须使用具有适当作用域scope的个人访问令牌personal access token在GitHub包中发布和安装包。有关更多信息，请参见“关于GitHub Packages”。

将您的个人访问令牌personal access token添加到您的`~/.npmrc`来进行身份验证，或者在 npm 命令行上用用户名和个人访问令牌登陆都是可以的。

在 `~/.npmrc` 上添加以下行，将 `TOKEN` 换为你自己的个人访问令牌 personal access token。如果你没有 `~/.npmrc` 这个文件的话你可以自己创建。

```text
//npm.pkg.github.com/:_authToken=TOKEN
```

要通过登录npm进行身份验证，请使用npm登录命令，用您的GitHub用户名替换USERNAME，用您的个人访问令牌替换TOKEN，用您的电子邮件地址替换PUBLIC-EMAIL-ADDRESS。

```text
$ npm login --registry=https://npm.pkg.github.com
> Username: USERNAME> Password: TOKEN> Email: PUBLIC-EMAIL-ADDRESS
```

使用GITHUB\_TOKEN进行身份验证 如果您正在使用GitHub Actions工作流，您可以使用GITHUB\_TOKEN在Github Packages中发布和使用包，而不需要存储和管理个人访问令牌。有关更多信息，请参见“使用GITHUB\_TOKEN进行身份验证”。

### 发布包

默认情况下，GitHub包会在GitHub存储库中发布一个包，您可以在package.json文件的名称字段中指定该包。例如，您可以将一个名为@my-org/test的包发布到my-org/test GitHub仓库中。您可以通过在package.json文件中包含描述字段来为包列表页面添加摘要。有关更多信息，请参见npm文档中的“使用软件包”和“如何创建节点模块”。

通过在package.json文件中包含一个网址字段，可以将多个包发布到同一个GitHub仓库中。有关更多信息，请参见“将多个包发布到同一个仓库”

您可以使用本地。或者在包中使用publishConfig选项。作用域包的名称格式为@owner/name。作用域包总是以@符号开头。您可能需要更新包中的名称，json才能使用作用域名称。例如，“名称”:@codertocat/hello-world-npm。

发布包后，您可以在GitHub上查看该包。有关更多信息，请参见“查看存储库的包”

使用本地发布包。npmrc文件 你可以用。npmrc文件来配置项目的范围映射。在……npmrc文件中，使用GitHub包的网址和帐户所有者，这样GitHub包就知道将包请求路由到哪里。使用。npmrc文件防止其他开发人员意外地将包发布到npmjs.org，而不是GitHub包。因为不支持大写字母，所以即使GitHub用户或组织名称包含大写字母，存储库所有者也必须使用小写字母。

