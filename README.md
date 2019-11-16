# 配置npm以用于GitHub Packages

您可以将npm配置为将包发布到Github Packages，并将存储在Github Packages中的包用作npm项目中的依赖项。

{% hint style="info" %}
GitHub软件包可与GitHub免费版、GitHub专业版、GitHub团队和GitHub企业云一起使用。有关更多信息，请参见“GitHub的产品”
{% endhint %}

您需要一个access token来发布、安装和删除GitHub Packages中的包。您可以使用个人访问令牌personal access token以您的用户名直接向GitHub软件包或GitHub API进行身份验证。您可以使用GITHUB\_TOKEN使用 GitHub Actions 工作流进行身份验证。

要使用个人访问令牌personal access token进行身份验证，您必须使用具有适当作用域scope的个人访问令牌personal access token在Github Packages中发布和安装包。有关更多信息，请参见“关于GitHub Packages”。

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

## 发布包

默认情况下，GitHub Packages会在GitHub 仓库中发布一个包，您可以在
`package.json`文件的名称字段中指定该包。例如，您可以将一个名为
`@my-org/test`的包发布到`my-org/test`的GitHub仓库中。您可以通过在`package.json`文件中包含描述字段来为包列表页面添加摘要。有关更多信
息，请参见npm文档中的“使用软件包”和“如何创建节点模块”。

通过在`package.json`文件中包含一个`URL`字段，可以将多个包发布到同一个GitHub仓库中。有关更多信息，请参见“将多个包发布到同一个仓库”

如果要设置作用域映射的话，用本地项目中的 `.npmrc` 文件或者 
`package.json` 中的 `publishConfig` 选项来配置都是可以的。作用
域包的名称格式为`@owner/name`。作用域包总是以`@`符号开头。您可能需要新
包中的名称，json才能使用作用域名称。例如，`name: "@codertocat/hello-world-npm"`

发布包后，您可以在GitHub上查看该包。有关更多信息，请参见“查看仓库的包”

### 使用本地`.npmrc`文件发布包

你可以用`.npmrc`文件来配置项目的作用域映射。在`.npmrc`文件中，使用Github Packages的URL和帐户所有者，这样Github Packages就知道将包请求路由到哪里。使用`.npmrc`文件防止其他开发人员意外地将包发布到`npmjs.org`，而不是GitHub Packages。因为不支持大写字母，所以即使GitHub用户或组织名称包含大写字母，仓库所有者也必须使用小写字母。

1. 向GitHub Packages进行身份验证。更多信息请参见“向GitHub Packages进行身份验证”
2. 在与 `package.json` 文件相同的目录中，创建或编辑 `.npmrc` 文件，添加一行指定 GitHub Packages 的URL和帐户所有者。用拥有包含项目的仓库的用户或组织帐户的名称替换OWNER。
```
registry=https://npm.pkg.github.com/OWNER
```
3. 把`.npmrc`文件添加到GitHub Packages可以找到您的项目的仓库中。有关更多信息，请参见“使用命令行将文件添加到仓库中”
4. 请在项目包中验证包的名称。名称字段必须包含包的作用域和名称。例如，如果您的包被称为“test”，并且您正在发布到 `my-org` GitHub组织，则包中的名称字段应为`@my-org/test`。
4. 请验证项目包中的`repostiroy`字段。`repository`字段必须与GitHub 仓库的URL匹配。例如，如果您的仓库URL是`github.com/my-org/test`，那么`repository`字段应该是`git://github.com/my-org/test.git`。
5. 发布
```
$npm publish
```

### 在`package.json`文件中使用`publishConfig`发布包

您可以在`package.json`文件中使用publishConfig元素来指定要在其中发布包的注册表。有关更多信息，请参见npm文档中的“发布配置”。

1. 为您的包编辑`package.json`文件，并包含一个`publishConfig`条目。
```
"publishConfig":{
    "registry":" https://npm.pkg.github.com/"
}
```
2. 请验证项目包中的`repository`字段。`repository`字段必须与GitHub仓库的URL匹配。例如，如果您的仓库URL是`github.com/my-org/test`，那么`repository`字段应该是`git://github.com/my-org/test.git`。
3. 发布包:
```
$ npm publish
```

### 将多个包发布到同一个仓库中
要将多个包发布到同一个仓库中，可以在每个包的 `package.json` 文件的 `repository` 字段中包含GitHub仓库的URL。GitHub包将基于该字段匹配仓库，而不是基于包名。如果将 `package.json` 文件存储在仓库的根目录之外，可以使用 `directory` 字段指定GitHub包可以找到 `package.json` 文件的位置。

```
"repository" : {
    "type" : "git",
    "url": "ssh://git@github.com/OWNER/REPOSITORY.git",
    "directory": "packages/name"
  },
```

## 安装包

安装软件包
您可以通过将软件包作为依赖项添加到项目的软件 `package.json` 文件中，从GitHub软件包安装软件包。有关在项目中使用包的更多信息，请参见npm文档中的“使用包”。

默认情况下，您可以添加来自一个组织的包。有关更多信息，请参见安装其他组织的软件包

您还需要添加 `.npmrc` 文件，这样所有安装包的请求都将通过GitHub包。当您通过GitHub包路由所有包请求时，您可以同时使用来自npmjs.com的有作用域包和无作用域包。有关更多信息，请参见npm文档中的“npm作用域”。

向GitHub包进行身份验证。有关更多信息，请参见“向GitHub包进行身份验证”

在与 `package.json` 文件相同的目录中，创建或编辑 `.npmrc` 文件中包含一行指定GitHub包的网址和帐户所有者。用拥有包含项目的仓库的用户或组织帐户的名称替换OWNER。
```conf
registry=https://pkg.npm.github.com/OWNER
```
添加 `.npmrc` 文件放到GitHub包可以找到您的项目的仓库中。有关更多信息，请参见“使用命令行将文件添加到仓库中”

在项目中配置 `package.json` 以使用正在安装的包。要将包依赖项添加到GitHub包的 `package.json` 文件中，请指定完整作用域的包名，如`@my-org/server`。对于来自npmjs.com的包，请指定全名，例如`@babel/core`或`@lodash`。例如，下面这个包。json使用`@octo-org/octo-app`包作为依赖项。

```json
{
  "name": "@my-org/server",
  "version": "1.0.0",
  "description": "Server app that uses the @octo-org/octo-app package",
  "main": "index.js",
  "author": "",
  "license": "MIT",
  "dependencies": {
    "@octo-org/octo-app": "1.0.0"
  }
}
```
5. 安装软件包。
```
$ npm install
```