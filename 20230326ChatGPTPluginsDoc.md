



<img src="https://p.ipic.vip/h1e5ve.png" alt="image-20230327174124277" style="zoom:150%;" />











[TOC]



### Chat Plugins 

学习如何构建一个插件，使 ChatGPT 能够智能地调用您的 API。

#### 简介

 OpenAI 插件将 ChatGPT 与第三方应用程序连接起来。这些插件使 ChatGPT 能够与开发人员定义的 API 进行交互，增强了 ChatGPT 的功能，使其能够执行各种操作。

插件可以让 ChatGPT 做这些事情： 获取实时信息；例如，体育比分、股票价格、最新新闻等。 检索知识库信息；例如，公司文档、个人笔记等。 代表用户执行操作；例如，预订航班、订购食物等。

> 插件处于有限的 alpha 阶段，您可能无法访问。请加入等待名单以获得访问权限。在 alpha 阶段，我们将与用户和开发者密切合作，对插件系统进行迭代，该系统可能会发生重大变化。

插件开发者公开一个或多个 API 端点，并附带一个标准化的清单文件和一个 OpenAPI规范。这些定义了插件的功能，使 ChatGPT 能够使用这些文件并向开发者定义的 API 发送调用。

AI 模型充当智能的API调用者。根据 API 规范和自然语言描述何时使用API，模型主动调用 API以执行操作。例如，如果用户询问，“在巴黎住几晚，我可以住在哪？”模型可能会选择调用酒店预订插件 API，接收 API 响应，并结合 API 数据和自然语言能力生成面向用户的答案。

随着时间的推移，我们预计系统将发展以适应更高级的用例。

#### 插件流程

要构建一个插件，了解端到端流程至关重要。

1.  在 yourdomain.com/.well-known/ai-plugin.json 上创建并托管一个清单文件
    - 该文件包括有关插件的元数据（名称、logo 等）、所需身份验证的详细信息（身份验证类型、OAuth URL 等）以及您想要公开的端点的 OpenAPI 规范。
    - 模型将查看 OpenAPI 描述字段，这些字段可用于为不同的字段提供自然语言描述。
    - 我们建议一开始仅公开 1-2 个端点，并将参数数量降到最低，以减少文本长度。插件描述、API 请求和 API 响应都将插入到与 ChatGPT 的对话中。这将计入模型的上下文限制。
2.  在 ChatGPT UI 中注册插件。
    - 从顶部下拉菜单中选择插件模型，然后选择“插件”、“插件商店”，最后选择“安装未经验证的插件”或“开发自己的插件”。
    - 如果需要身份验证，请提供 OAuth 2 客户端ID和客户端密钥或 API 密钥。
3.  用户激活插件
    - 用户必须在 ChatGPT UI 中手动激活插件。（ChatGPT 默认不会使用您的插件。）
    - 在 alpha 阶段，插件开发者将能够与其他 15 名用户分享插件（目前只有其他开发者可以安装未经验证的插件）。我们将逐步推出一种提交插件审查的方法，以使其暴露给所有 ChatGPT 用户。
    - 如果需要验证，用户将通过 OAuth 重定向到您的插件；您还可以在此处创建新帐户。
    - 将来，我们希望建立一些功能，帮助用户发现有用且受欢迎的插件。
4.  用户开始对话。
    - OpenAI 将在向 ChatGPT 发送消息时，将插件的简洁描述插入其中，对最终用户不可见。这将包括插件描述、端点和示例。
    - 当用户提出相关问题时，模型可能会选择从您的插件中调用 API（如果看起来相关的话）；对于 POST 请求，我们要求开发者构建一个用户确认流程。
    - 模型将 API 结果纳入其对用户的回应。
    - 模型可能会在其回应中包含从 API 调用返回的链接。这些将显示为丰富的预览（使用 OpenGraph 协议，我们将提取 site\_name、title、description、image 和 url 字段）。

目前，我们将在插件对话头中发送用户所在国家和州的信息（例如，如果您在加州，它看起来像是 {"openai-subdivision-1-iso-code": "US-CA"}）。对于其他数据来源，用户需要通过同意选择加入。这对购物、餐厅、天气等非常有用。您可以在我们的开发者使用条款中了解更多信息。

### 开始

创建插件分为三个步骤：

1.  构建 API
2.  以 OpenAPI yaml 或 JSON 格式记录 API
3.  创建一个 JSON 清单文件，为插件定义相关元数据

接下来的部分将通过定义 OpenAPI 规范和清单文件来创建一个待办事项列表插件。

#### 插件清单

每个插件都需要一个 ai-plugin.json 文件，该文件需要托管在 API 的域上。例如，一个名为 example.com 的公司将通过 [https://example.com](https://example.com) 域让插件 JSON 文件可访问，因为这是他们的 API 托管的地方。当你通过 ChatGPT UI 安装插件时，后端会查找位于 /.well-known/ai-plugin.json 的文件。/.well-known 文件夹是必需的，必须存在于您的域上，以便 ChatGPT 与您的插件建立连接。如果找不到文件，插件将无法安装。对于本地开发，您可以使用 HTTP，但如果指向远程服务器，则需要 HTTPS。

ai-plugin.json文件的最小定义如下所示：

```json
{
  "schema_version": "v1",
  "name_for_human": "TODO Plugin",
  "name_for_model": "todo",
  "description_for_human": "Plugin for managing a TODO list. You can add, remove and view your TODOs.",
  "description_for_model": "Plugin for managing a TODO list. You can add, remove and view your TODOs.",
  "auth": {
    "type": "none"
  },
  "api": {
    "type": "openapi",
    "url": "http://localhost:3333/openapi.yaml",
    "is_user_authenticated": false
  },
  "logo_url": "http://localhost:3333/logo.png",
  "contact_email": "support@example.com",
  "legal_info_url": "http://www.example.com/legal"
}
```

如果您想查看插件文件的所有可能选项，可以参考以下定义：

| 字段                  | 类型                  | 描述/选项                                                    |
| --------------------- | --------------------- | ------------------------------------------------------------ |
| schema_version        | String                | 清单模式版本                                                 |
| name_for_model        | String                | 模型用于定位插件的名称                                       |
| name_for_human        | String                | 人类可读名称，例如完整的公司名称                             |
| description_for_model | String                | 更适合模型的描述，例如令牌上下文长度考虑或关键字用法以改善插件提示。 |
| description_for_human | String                | 插件的人类可读描述                                           |
| auth                  | ManifestAuth          | 身份验证模式                                                 |
| api                   | Object                | API 规范                                                     |
| logo_url              | String                | 用于获取插件 logo 的 URL                                     |
| contact_email         | String                | 安全/审核联系方式，支持和停用的电子邮件                      |
| legal_info_url        | String                | 用户查看插件信息的重定向 URL                                 |
| HttpAuthorizationType | HttpAuthorizationType | "bearer" 或 "basic"                                          |
| ManifestAuthType      | ManifestAuthType      | "none"、"user_http"、"service_http" 或 "oauth"               |
| interface             | BaseManifestAuth      | 类型：ManifestAuthType；说明：字符串                         |
| ManifestNoAuth        | ManifestNoAuth        | 不需要身份验证：BaseManifestAuth & { type: 'none', }         |
| ManifestAuth          | ManifestAuth          | ManifestNoAuth, ManifestServiceHttpAuth, ManifestUserHttpAuth, ManifestOAuthAuth |

下面是不同认证方法的示例：

```
# App-level API keys

type ManifestServiceHttpAuth  = BaseManifestAuth & {
  type: 'service_http';
  authorization_type: HttpAuthorizationType;
  verification_tokens: {
    [service: string]?: string;
  };
}

# User-level HTTP authentication

type ManifestUserHttpAuth  = BaseManifestAuth & {
  type: 'user_http';
  authorization_type: HttpAuthorizationType;
}

type ManifestOAuthAuth  = BaseManifestAuth & {
  type: 'oauth';

  # OAuth URL where a user is directed to for the OAuth authentication flow to begin.

  client_url: string;

  # OAuth scopes required to accomplish operations on the user's behalf.

  scope: string;

  # Endpoint used to exchange OAuth code with access token.

  authorization_url: string;

  # When exchanging OAuth code with access token, the expected header 'content-type'. For example: 'content-type: application/json'

  authorization_content_type: string;

  # When registering the OAuth client ID and secrets, the plugin service will surface a unique token. 

  verification_tokens: {
    [service: string]?: string;
  };
}
```

清单文件中某些字段的长度也受到限制，并可能随时间变化：

- `name_for_human` 最多50 个字符

- `name_for_model`最多50 个字符
- `description_for_human`最多120 个字符 
- `description_for_model`最多8000 个字符（随着时间的推移会减少） 

另外，我们还对 API 响应正文长度设有100k 字符限制（随着时间的推移会减少），这也可能发生变化。

#### OpenAPI 定义

下一步是构建 OpenAPI 规范来记录API。除了在 OpenAPI规范和清单文件中定义的内容之外，ChatGPT 中的模型对您的 API 一无所知。这意味着，如果您有一个庞大的 API，您不需要将所有功能暴露给模型，可以选择特定的端点。例如，如果您有一个社交媒体 API，您可能希望让模型通过 GET 请求访问站点上的内容，但阻止模型评论用户帖子，以减少冗余信息出现几率。

OpenAPI 规范是覆盖在您 API 之上的包装。一个基本的 OpenAPI 规范如下所示：

```
openapi: 3.0.1
info:
  title: TODO Plugin
  description: A plugin that allows the user to create and manage a TODO list using ChatGPT.
  version: 'v1'
servers:

  - url: http://localhost:3333
    paths:
      /todos:
    get:
      operationId: getTodos
      summary: Get the list of todos
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/getTodosResponse'
    components:
      schemas:
    getTodosResponse:
      type: object
      properties:
        todos:
          type: array
          items:
            type: string
          description: The list of todos.
```

我们首先定义规范版本、标题、描述和版本号。当在 ChatGPT 中运行查询时，它将查看 info 部分中定义的描述，以确定插件是否与用户查询相关。您可以在编写描述部分中阅读更多关于提示的内容。

请注意 OpenAPI 规范中以下可能发生变化的限制：

*   每个 API 端点描述/摘要字段在 API 规范中最多 200 个字符
*   每个 API 参数描述字段在 API 规范中最多 200 个字符

由于我们在本地运行此示例，我们希望将服务器设置为指向您的本地主机 URL。OpenAPI 规范的其余部分遵循传统的 OpenAPI 格式，您可以通过各种在线资源了解有关 OpenAPI 格式的更多信息。还有许多工具可以根据您的底层 API 代码自动生成 OpenAPI 规范。

#### 运行插件

创建 API、清单文件和 API 的 OpenAPI 规范后，您现在可以通过ChatGPT UI连接插件。您的插件可能在两个不同的位置运行，要么在本地开发环境中，要么在远程服务器上。

如果您的 API有一个本地版本正在运行，您可以将插件界面指向该本地设置。要将插件与 ChatGPT 连接，您可以导航到插件商店，然后选择“安装未验证的插件”。

如果插件在远程服务器上运行，您需要首先选择“开发自己的插件”，然后选择“安装未验证的插件”。您可以将插件清单文件简单地添加到 ./well-known路径并开始测试您的 API。但是，对于后续对清单文件的更改，您将必须将新更改部署到您的公共站点，这可能需要很长时间。在这种情况下，我们建议设置一个本地服务器作为您 API 的代理。这样可以快速原型化您的OpenAPI规范和清单文件的更改。

##### 设置公开API 的本地代理 

```python
import requests
import os

import yaml
from flask import Flask, jsonify, Response, request, send_from_directory
from flask_cors import CORS

app = Flask(__name__)

PORT = 3333
CORS(app, origins=[f"http://localhost:{PORT}", "https://chat.openai.com"])

api_url = 'https://example'


@app.route('/.well-known/ai-plugin.json')
def serve_manifest():
    return send_from_directory(os.path.dirname(__file__), 'ai-plugin.json')


@app.route('/openapi.yaml')
def serve_openapi_yaml():
    with open(os.path.join(os.path.dirname(__file__), 'openapi.yaml'), 'r') as f:
        yaml_data = f.read()
    yaml_data = yaml.load(yaml_data, Loader=yaml.FullLoader)
    return jsonify(yaml_data)


@app.route('/openapi.json')
def serve_openapi_json():
    return send_from_directory(os.path.dirname(__file__), 'openapi.json')


@app.route('/<path:path>', methods=['GET', 'POST'])
def wrapper(path):

    headers = {
    'Content-Type': 'application/json',
    }
    
    url = f'{api_url}/{path}'
    print(f'Forwarding call: {request.method} {path} -> {url}')
    
    if request.method == 'GET':
        response = requests.get(url, headers=headers, params=request.args)
    elif request.method == 'POST':
        print(request.headers)
        response = requests.post(url, headers=headers, params=request.args, json=request.json)
    else:
        raise NotImplementedError(f'Method {request.method} not implemented in wrapper for {path=}')
    return response.content

if __name__ == '__main__':
    app.run(port=PORT)
```

#### 编写描述

当用户发出可能是插件请求的查询时，模型会查看 OpenAPI 规范中端点的描述以及清单文件中的 description_for_model。就像提示其他语言模型一样，您需要测试多个提示和描述以找到最佳效果。

OpenAPI规范本身是提供有关您API不同细节的绝佳场所 - 可用的功能、参数等。除了为每个字段使用富有表现力、信息量充足的名称外，规范中还可以包含每个属性的“描述”字段。这些可用于提供关于功能的作用或查询字段所需信息的自然语言描述。模型将能够看到这些描述，并在使用 API 时被指导。如果一个字段仅限于某些值，您还可以提供带有描述性类别名称的“枚举”。

“description_for_model”属性让您可以指导模型如何使用您的插件。总的来说，ChatGPT 背后的语言模型非常擅长理解自然语言并遵循指示。因此，这是放入有关插件功能及如何正确使用模型的一般说明的好地方。请使用自然语言，最好简洁且具有描述性和客观。您可以查看一些示例以了解这应该是什么样子的。针对”description_for_model“，我们建议从“插件用于...”开始，然后列举您的API 提供的所有功能。

#### 最佳实践 

编写描述模型和OpenAPI规范中的描述以及设计API响应时，请遵循以下最佳实践：

1. 您的描述不应试图控制ChatGPT的情绪、个性或确切回应。ChatGPT旨在为插件编写合适的响应。

   错误示例：

   > 当用户要求查看他们的待办事项列表时，请始终回复“我找到了你的待办事项列表！你有\[x\]个待办事项：\[在此列出待办事项\]。如果你愿意，我可以添加更多待办事项！”

​		正确示例：

​		 [对此无需说明\]

2. 您的描述不应在用户没有要求使用特定类别服务的情况下鼓励ChatGPT使用插件。

   错误示例：

   > 每当用户提及任何类型的任务或计划时，请询问他们是否想使用TODO插件将某事添加到待办事项列表中。

​	正确示例：

> 待办事项列表可以添加、删除和查看用户的待办事项。

3. 您的描述不应为ChatGPT使用插件规定特定的触发器。ChatGPT在适当时自动使用您的插件。

   错误示例：

   > 当用户提到任务时，回复“您想让我将其添加到您的待办事项列表中吗？请说'是'以继续。"

​	正确示例：

​	[对此无需说明\]

4. 插件API响应应返回原始数据而不是自然语言响应，除非有必要。ChatGPT将使用返回的数据提供自己的自然语言响应。

   错误示例：

   > 我找到了你的待办事项列表！你有2个待办事项：购买杂货和遛狗。如果你愿意，我可以添加更多待办事项！

​	正确示例：

> { "todos": \[ "get groceries", "walk the dog" \] }

#### 调试 

默认情况下，聊天不会显示插件调用和其他未展示给用户的信息。为了更完整地了解模型与插件的交互，您可以单击屏幕左下方的“调试”按钮以打开调试窗格。这将打开包含插件调用和响应的到目前为止的对话的原始文本表示。

模型对插件的调用通常包括模型（“助手”）发送的包含类似JSON的参数的消息，然后是插件（“工具”）的响应，最后是模型利用插件返回的信息的消息。

在某些情况下，例如在插件安装过程中，错误可能会在浏览器的JavaScript控制台上显示。

### 插件认证 

插件提供了许多认证方案，以适应各种用例。要为您的插件指定认证方案，请使用清单文件。我们的插件域策略概述了我们解决域安全问题的策略。有关可用认证选项的示例，请参考示例部分，该部分展示了所有不同的选择。

#### 无需认证

对于不需要认证的应用程序，我们支持无认证流程，用户可以直接向您的API发送请求，无需任何限制。如果您有一个开放的API，希望建立一个可供所有人使用的API，这将非常有用，因为它允许除了OpenAI插件请求之外的其他来源的流量。

```
"auth": {
  "type": "none"
},
```

#### 服务级别

如果您希望特定启用OpenAI插件与您的API协同工作，您可以在插件安装流程中提供客户端密钥。这意味着来自OpenAI插件的所有流量都将进行认证，但不是在用户级别上。这种流程受益于简单的最终用户体验，但从API的角度来看控制较少。

首先，开发者粘贴他们的访问令牌（全局密钥） 然后，他们必须将验证令牌添加到他们的清单文件中 我们存储令牌的加密版本 在安装插件时，用户不需要执行任何操作 最后，我们在向插件发出请求时将其传递到Authorization标头中（“Authorization”：“\[Bearer/Basic\]\[用户的令牌\]”）

```
"auth": {
  "type": "service_http",
  "authorization_type": "bearer",
  "verification_tokens": {
    "openai": "cb7cdfb8a57e45bc8ad7dea5bc2f8324"
  }
},
```

#### 用户级别

就像用户可能已经在使用您的API一样，我们通过允许最终用户在插件安装期间将他们的秘密API密钥复制并粘贴到ChatGPT UI中来实现用户级别认证。虽然我们在将密钥存储到数据库时会对其进行加密，但我们不建议使用这种方法，因为用户体验很差。

首先，用户在安装插件时粘贴他们的访问令牌 我们存储令牌的加密版本 然后，我们在向插件发出请求时将其传递到Authorization标头中（“Authorization”：“\[Bearer/Basic\]\[用户的令牌\]”）

```
"auth": {
  "type": "user_http",
  "authorization_type": "bearer",
},
```

#### OAuth

插件协议与OAuth兼容。我们在清单中期望的OAuth流程的一个简单示例如下：

- 首先，开发者粘贴他们的OAuth客户端ID和客户端密钥
  - 然后，他们必须将验证令牌添加到他们的清单文件中
- 我们存储客户端密钥的加密版本
- 用户在安装插件时通过插件的网站登录
  - 这将为我们提供一个用户的OAuth访问令牌（以及可选的刷新令牌），我们对其进行加密存储 
- 最后，我们在向插件发出请求时将该用户的令牌传递到Authorization标头中

```
"auth": {
  "type": "oauth",
  "client_url": "https://my_server.com/authorize",
  "scope": "",
  "authorization_url": "https://my_server.com/token",
  "authorization_content_type": "application/json",
  "verification_tokens": {
    "openai": "abc123456"
  }
},
```

为了更好地理解OAuth的URL结构，这里对字段进行了简短的描述：

- 当您将插件与ChatGPT设置时，您将被要求提供您的OAuth client\_id和client\_secret

- 当用户登录插件时，ChatGPT将引导用户的浏览器转到

  ```
  "\[client\_url\]?response\_type=code&client\_id=\[client\_id\]&scope=\[scope\]&redirect\_uri=https%3A%2F%2Fchat.openai.com%2Faip%2F\[plugin\_id\]%2Foauth%2Fcallback" 
  ```

- 在您的插件将用户重定向回给定的redirect\_uri之后，ChatGPT将通过使用内容类型`authorization\_content\_type`和参数

  ```
  { “grant\_type”: “authorization\_code”, “client\_id”: \[client\_id\], “client\_secret”: \[client\_secret\], “code”: \[the code that was returned with the redirect\], “redirect\_uri”: \[the same redirect uri as before\] }
  ```

  向authorization\_url发出POST请求以完成OAuth流程。

这样，您的插件将能够根据用户的特定需求和权限为其提供服务。通过使用OAuth，您可以确保用户的数据安全并提供更个性化的体验。在设计和实现插件时，请确保遵循最佳实践和安全准则以保护用户的隐私和数据。

### 示例插件 

开始开发，我们提供了一组简单的插件，涵盖了不同的认证方案和用例。从简单的无认证待办事项列表插件到更强大的检索插件，这些示例展示了我们希望通过插件实现的可能性。

在开发过程中，您可以在本地计算机上运行插件，也可以通过云开发环境（如GitHub Codespaces，Replit或CodeSandbox）运行。

#### 学习如何构建一个简单的无认证待办事项列表插件 

首先，定义一个带有以下字段的manifest.json文件：

```
{
  "schema_version": "v1",
  "name_for_human": "TODO Plugin (no auth)",
  "name_for_model": "todo",
  "description_for_human": "Plugin for managing a TODO list, you can add, remove and view your TODOs.",
  "description_for_model": "Plugin for managing a TODO list, you can add, remove and view your TODOs.",
  "auth": {
    "type": "none"
  },
  "api": {
    "type": "openapi",
    "url": "PLUGIN_HOSTNAME/openapi.yaml",
    "is_user_authenticated": false
  },
  "logo_url": "PLUGIN_HOSTNAME/logo.png",
  "contact_email": "dummy@email.com",
  "legal_info_url": "http://www.example.com/legal"
}
```

接下来，我们可以定义一些简单的Python端点，为特定用户创建、删除和获取待办事项列表。

```
import json

import quart
import quart_cors
from quart import request

app = quart_cors.cors(quart.Quart(__name__), allow_origin="https://chat.openai.com")

_TODOS = {}


@app.post("/todos/<string:username>")
async def add_todo(username):
    request = await quart.request.get_json(force=True)
    if username not in _TODOS:
        _TODOS[username] = []
    _TODOS[username].append(request["todo"])
    return quart.Response(response='OK', status=200)


@app.get("/todos/<string:username>")
async def get_todos(username):
    return quart.Response(response=json.dumps(_TODOS.get(username, [])), status=200)


@app.delete("/todos/<string:username>")
async def delete_todo(username):
    request = await quart.request.get_json(force=True)
    todo_idx = request["todo_idx"]
    # fail silently, it's a simple plugin
    if 0 <= todo_idx < len(_TODOS[username]):
        _TODOS[username].pop(todo_idx)
    return quart.Response(response='OK', status=200)


@app.get("/logo.png")
async def plugin_logo():
    filename = 'logo.png'
    return await quart.send_file(filename, mimetype='image/png')


@app.get("/.well-known/ai-plugin.json")
async def plugin_manifest():
    host = request.headers['Host']
    with open("manifest.json") as f:
        text = f.read()
        text = text.replace("PLUGIN_HOSTNAME", f"https://{host}")
        return quart.Response(text, mimetype="text/json")


@app.get("/openapi.yaml")
async def openapi_spec():
    host = request.headers['Host']
    with open("openapi.yaml") as f:
        text = f.read()
        text = text.replace("PLUGIN_HOSTNAME", f"https://{host}")
        return quart.Response(text, mimetype="text/yaml")


def main():
    app.run(debug=True, host="0.0.0.0", port=5002)

if __name__ == "__main__":
    main()
```

最后，我们需要设置并定义一个与本地或远程服务器上定义的端点相匹配的OpenAPI规范。您无需通过规范暴露API的全部功能，而可以选择让ChatGPT仅访问某些功能。

还有许多工具可以自动将您的服务器定义代码转换为OpenAPI规范，因此无需手动执行此操作。对于上面的Python代码，OpenAPI规范看起来如下：

```
openapi: 3.0.1
info:
  title: TODO Plugin
  description: A plugin that allows the user to create and manage a TODO list using ChatGPT. If you do not know the user's username, ask them first before making queries to the plugin. Otherwise, use the username "global".
  version: 'v1'
servers:

  - url: PLUGIN_HOSTNAME
    paths:
      /todos/{username}:
    get:
      operationId: getTodos
      summary: Get the list of todos
      parameters:
      - in: path
        name: username
        schema:
            type: string
        required: true
        description: The name of the user.
          responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/getTodosResponse'
        post:
          operationId: addTodo
          summary: Add a todo to the list
          parameters:
      - in: path
        name: username
        schema:
            type: string
        required: true
        description: The name of the user.
          requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/addTodoRequest'
          responses:
        "200":
          description: OK
        delete:
          operationId: deleteTodo
          summary: Delete a todo from the list
          parameters:
      - in: path
        name: username
        schema:
            type: string
        required: true
        description: The name of the user.
          requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/deleteTodoRequest'
          responses:
        "200":
          description: OK

components:
  schemas:
    getTodosResponse:
      type: object
      properties:
        todos:
          type: array
          items:
            type: string
          description: The list of todos.
    addTodoRequest:
      type: object
      required:

   - todo
     operties:
     todo:
       type: string
       description: The todo to add to the list.
       required: true
         deleteTodoRequest:
           type: object
           required:
   - todo_idx
     operties:
     todo_idx:
       type: integer
       description: The index of the todo to delete.
       required: true
```

#### 学习如何构建一个具有服务级别认证的简单待办事项插件

首先，定义一个带有以下字段的manifest.json文件：

```
{
  "schema_version": "v1",
  "name_for_human": "TODO Plugin (service http)",
  "name_for_model": "todo",
  "description_for_human": "Plugin for managing a TODO list, you can add, remove and view your TODOs.",
  "description_for_model": "Plugin for managing a TODO list, you can add, remove and view your TODOs.",
  "auth": {
    "type": "service_http",
    "authorization_type": "bearer",
    "verification_tokens": {
      "openai": "758e9ef7984b415688972d749f8aa58e"
    }
  },
   "api": {
    "type": "openapi",
    "url": "https://example.com/openapi.yaml",
    "is_user_authenticated": false
  },
  "logo_url": "https://example.com/logo.png",
  "contact_email": "dummy@email.com",
  "legal_info_url": "http://www.example.com/legal"
}
```

请注意，服务级认证插件需要验证令牌。在ChatGPT Web UI中的插件安装过程中会生成令牌。

接下来，我们可以定义一些简单的Python端点，为特定用户创建、删除和获取待办事项列表。端点还需要进行简单的认证检查，因为这是必需的。

```
import json

import quart
import quart_cors
from quart import request

app = quart_cors.cors(quart.Quart(__name__), allow_origin="https://chat.openai.com")

_SERVICE_AUTH_KEY = "TEST"
_TODOS = {}


def assert_auth_header(req):
    assert req.headers.get(
        "Authorization", None) == f"Bearer {_SERVICE_AUTH_KEY}"


@app.post("/todos/<string:username>")
async def add_todo(username):
    assert_auth_header(quart.request)
    request = await quart.request.get_json(force=True)
    if username not in _TODOS:
        _TODOS[username] = []
    _TODOS[username].append(request["todo"])
    return quart.Response(response='OK', status=200)


@app.get("/todos/<string:username>")
async def get_todos(username):
    assert_auth_header(quart.request)
    return quart.Response(response=json.dumps(_TODOS.get(username, [])), status=200)


@app.delete("/todos/<string:username>")
async def delete_todo(username):
    assert_auth_header(quart.request)
    request = await quart.request.get_json(force=True)
    todo_idx = request["todo_idx"]
    # fail silently, it's a simple plugin
    if 0 <= todo_idx < len(_TODOS[username]):
        _TODOS[username].pop(todo_idx)
    return quart.Response(response='OK', status=200)


@app.get("/logo.png")
async def plugin_logo():
    filename = 'logo.png'
    return await quart.send_file(filename, mimetype='image/png')


@app.get("/.well-known/ai-plugin.json")
async def plugin_manifest():
    host = request.headers['Host']
    with open("manifest.json") as f:
        text = f.read()
        text = text.replace("PLUGIN_HOSTNAME", f"https://{host}")
        return quart.Response(text, mimetype="text/json")


@app.get("/openapi.yaml")
async def openapi_spec():
    host = request.headers['Host']
    with open("openapi.yaml") as f:
        text = f.read()
        text = text.replace("PLUGIN_HOSTNAME", f"https://{host}")
        return quart.Response(text, mimetype="text/yaml")


def main():
    app.run(debug=True, host="0.0.0.0", port=5002)

if __name__ == "__main__":
    main()
```

最后，我们需要设置并定义一个与本地或远程服务器上定义的端点相匹配的OpenAPI规范。通常情况下，OpenAPI规范的外观与认证方法无关。使用自动OpenAPI生成器在创建OpenAPI规范时减少错误的机会，因此值得探讨选项。

```
openapi: 3.0.1
info:
  title: TODO Plugin
  description: A plugin that allows the user to create and manage a TODO list using ChatGPT. If you do not know the user's username, ask them first before making queries to the plugin. Otherwise, use the username "global".
  version: 'v1'
servers:

  - url: https://example.com
    paths:
      /todos/{username}:
    get:
      operationId: getTodos
      summary: Get the list of todos
      parameters:
      - in: path
        name: username
        schema:
            type: string
        required: true
        description: The name of the user.
          responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/getTodosResponse'
        post:
          operationId: addTodo
          summary: Add a todo to the list
          parameters:
      - in: path
        name: username
        schema:
            type: string
        required: true
        description: The name of the user.
          requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/addTodoRequest'
          responses:
        "200":
          description: OK
        delete:
          operationId: deleteTodo
          summary: Delete a todo from the list
          parameters:
      - in: path
        name: username
        schema:
            type: string
        required: true
        description: The name of the user.
          requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/deleteTodoRequest'
          responses:
        "200":
          description: OK

components:
  schemas:
    getTodosResponse:
      type: object
      properties:
        todos:
          type: array
          items:
            type: string
          description: The list of todos.
    addTodoRequest:
      type: object
      required:

   - todo
     operties:
     todo:
       type: string
       description: The todo to add to the list.
       required: true
         deleteTodoRequest:
           type: object
           required:
   - todo_idx
     operties:
     todo_idx:
       type: integer
       description: The index of the todo to delete.
       required: true
```

#### 学习如何构建一个简单的体育统计插件

此插件是一个简单的体育统计API示例。在考虑要构建什么时，请注意我们的域名规定和使用规定。

首先，我们可以定义一个manifest.json文件

```
{
  "schema_version": "v1",
  "name_for_human": "Sport Stats",
  "name_for_model": "sportStats",
  "description_for_human": "Get current and historical stats for sport players and games.",
  "description_for_model": "Get current and historical stats for sport players and games. Always display results using markdown tables.",
  "auth": {
    "type": "none"
  },
  "api": {
    "type": "openapi",
    "url": "PLUGIN_HOSTNAME/openapi.yaml",
    "is_user_authenticated": false
  },
  "logo_url": "PLUGIN_HOSTNAME/logo.png",
  "contact_email": "dummy@email.com",
  "legal_info_url": "http://www.example.com/legal"
}
```

接下来，我们为一个简单的体育服务插件定义一个模拟API。

```
import json
import requests
import urllib.parse

import quart
import quart_cors
from quart import request

app = quart_cors.cors(quart.Quart(__name__), allow_origin="https://chat.openai.com")
HOST_URL = "https://example.com"

@app.get("/players")
async def get_players():
    query = request.args.get("query")
    res = requests.get(
        f"{HOST_URL}/api/v1/players?search={query}&page=0&per_page=100")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/teams")
async def get_teams():
    res = requests.get(
        "{HOST_URL}/api/v1/teams?page=0&per_page=100")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/games")
async def get_games():
    query_params = [("page", "0")]
    limit = request.args.get("limit")
    query_params.append(("per_page", limit or "100"))
    start_date = request.args.get("start_date")
    if start_date:
        query_params.append(("start_date", start_date))
    end_date = request.args.get("end_date")
    

    if end_date:
        query_params.append(("end_date", end_date))
    seasons = request.args.getlist("seasons")
    
    for season in seasons:
        query_params.append(("seasons[]", str(season)))
    team_ids = request.args.getlist("team_ids")
    
    for team_id in team_ids:
        query_params.append(("team_ids[]", str(team_id)))
    print(query_params)
    
    res = requests.get(
        f"{HOST_URL}/api/v1/games?{urllib.parse.urlencode(query_params)}")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/stats")
async def get_stats():
    query_params = [("page", "0")]
    limit = request.args.get("limit")
    query_params.append(("per_page", limit or "100"))
    start_date = request.args.get("start_date")
    if start_date:
        query_params.append(("start_date", start_date))
    end_date = request.args.get("end_date")
    

    if end_date:
        query_params.append(("end_date", end_date))
    player_ids = request.args.getlist("player_ids")
    
    for player_id in player_ids:
        query_params.append(("player_ids[]", str(player_id)))
    game_ids = request.args.getlist("game_ids")
    
    for game_id in game_ids:
        query_params.append(("game_ids[]", str(game_id)))
    res = requests.get(
        f"{HOST_URL}/api/v1/stats?{urllib.parse.urlencode(query_params)}")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/season_averages")
async def get_season_averages():
    query_params = []
    season = request.args.get("season")
    if season:
        query_params.append(("season", str(season)))
    player_ids = request.args.getlist("player_ids")
    

    for player_id in player_ids:
        query_params.append(("player_ids[]", str(player_id)))
    res = requests.get(
        f"{HOST_URL}/api/v1/season_averages?{urllib.parse.urlencode(query_params)}")
    body = res.json()
    return quart.Response(response=json.dumps(body), status=200)


@app.get("/logo.png")
async def plugin_logo():
    filename = 'logo.png'
    return await quart.send_file(filename, mimetype='image/png')


@app.get("/.well-known/ai-plugin.json")
async def plugin_manifest():
    host = request.headers['Host']
    with open("manifest.json") as f:
        text = f.read()
        text = text.replace("PLUGIN_HOSTNAME", f"https://{host}")
        return quart.Response(text, mimetype="text/json")


@app.get("/openapi.yaml")
async def openapi_spec():
    host = request.headers['Host']
    with open("openapi.yaml") as f:
        text = f.read()
        text = text.replace("PLUGIN_HOSTNAME", f"https://{host}")
        return quart.Response(text, mimetype="text/yaml")


def main():
    app.run(debug=True, host="0.0.0.0", port=5001)

if __name__ == "__main__":
    main()
```

最后，我们定义一个OpenAPI规范。

```
openapi: 3.0.1
info:
  title: Sport Stats
  description: Get current and historical stats for sport players and games.
  version: 'v1'
servers:

  - url: PLUGIN_HOSTNAME
    paths:
      /players:
    get:
      operationId: getPlayers
      summary: Retrieves all players from all seasons whose names match the query string.
      parameters:
      - in: query
        name: query
        schema:
            type: string
        description: Used to filter players based on their name. For example, ?query=davis will return players that have 'davis' in their first or last name.
          responses:
        "200":
          description: OK
          /teams:
        get:
          operationId: getTeams
          summary: Retrieves all teams for the current season.
          responses:
        "200":
          description: OK
          /games:
        get:
          operationId: getGames
          summary: Retrieves all games that match the filters specified by the args. Display results using markdown tables.
          parameters:
      - in: query
        name: limit
        schema:
            type: string
        description: The max number of results to return.
      - in: query
        name: seasons
        schema:
            type: array
            items:
              type: string
        description: Filter by seasons. Seasons are represented by the year they began. For example, 2018 represents season 2018-2019.
      - in: query
        name: team_ids
        schema:
            type: array
            items:
              type: string
        description: Filter by team ids. Team ids can be determined using the getTeams function.
      - in: query
        name: start_date
        schema:
            type: string
        description: A single date in 'YYYY-MM-DD' format. This is used to select games that occur on or after this date.
      - in: query
        name: end_date
        schema:
            type: string
        description: A single date in 'YYYY-MM-DD' format. This is used to select games that occur on or before this date.
          responses:
        "200":
          description: OK
          /stats:
        get:
          operationId: getStats
          summary: Retrieves stats that match the filters specified by the args. Display results using markdown tables.
          parameters:
      - in: query
        name: limit
        schema:
            type: string
        description: The max number of results to return.
      - in: query
        name: player_ids
        schema:
            type: array
            items:
              type: string
        description: Filter by player ids. Player ids can be determined using the getPlayers function.
      - in: query
        name: game_ids
        schema:
            type: array
            items:
              type: string
        description: Filter by game ids. Game ids can be determined using the getGames function.
      - in: query
        name: start_date
        schema:
            type: string
        description: A single date in 'YYYY-MM-DD' format. This is used to select games that occur on or after this date.
      - in: query
        name: end_date
        schema:
            type: string
        description: A single date in 'YYYY-MM-DD' format. This is used to select games that occur on or before this date.
          responses:
        "200":
          description: OK
          /season_averages:
        get:
          operationId: getSeasonAverages
          summary: Retrieves regular season averages for the given players. Display results using markdown tables.
          parameters:
      - in: query
        name: season
        schema:
            type: string
        description: Defaults to the current season. A season is represented by the year it began. For example, 2018 represents season 2018-2019.
      - in: query
        name: player_ids
        schema:
            type: array
            items:
              type: string
        description: Filter by player ids. Player ids can be determined using the getPlayers function.
          responses:
        "200":
          description: OK
```



#### 学习如何构建一个语义搜索和检索插件

ChatGPT检索插件是我们提供的最详细代码示例。考虑到插件范围，我们鼓励您浏览代码库并阅读代码以更好地了解可能与一些简单示例不同的地方。

检索插件示例包括：

*   支持多个向量数据库提供商
*   所有4种不同认证方法
*   多种不同的功能

### 生产环境中的插件 

#### 速率限制 

考虑在您公开的 API 端点上实施速率限制。尽管当前的规模有限，但 ChatGPT 得到了广泛使用，您应该预料到大量请求。您可以监控请求的数量并相应地设置限制。

#### 更新插件

插件被批准并对 ChatGPT 用户可用后，您可能需要随着时间的推移更新清单文件。当发出请求时，ChatGPT 将定期获取 OpenAPI 规范。要手动更新清单文件，请导航到插件商店，然后再次执行“开发您自己的插件”流程。

#### 插件条款 

要注册插件，您必须同意插件条款。

#### 域名验证和安全性

为确保插件只能对其控制的资源执行操作，OpenAI 对插件的清单和 API 规范强制执行要求。

##### 定义插件的根域名

清单文件定义了向用户显示的信息（如logo和联系信息）以及托管插件的 OpenAPI 规范的 URL。获取清单时，根据以下规则建立插件的根域名：

- 如果域名以 [www](http://www). 作为子域名，则根域名将从托管清单的域名中去掉 [www.。](http://www.%E3%80%82)

- 否则，根域名与托管清单的域名相同。

关于重定向的注意事项：在解析清单过程中，如果有任何重定向，只允许子域名重定向。唯一的例外是从带有 www 的子域名重定向到没有 www 的子域名。

以下是根域名的示例：

- ✅ [https://example.com/.well-known/ai-plugin.json](https://example.com/.well-known/ai-plugin.json) 
  - 根域名：example.com

- ✅ [https://www.example.com/.well-known/ai-plugin.json](https://www.example.com/.well-known/ai-plugin.json) 
  - 根域名：example.com
- ✅ [https://www.example.com/.well-known/ai-plugin.json](https://www.example.com/.well-known/ai-plugin.json) → 重定向到 [https://example.com/.well-known/ai-plugin.json](https://example.com/.well-known/ai-plugin.json) 
  - 根域名：example.com 
- ✅ [https://foo.example.com/.well-known/ai-plugin.json](https://foo.example.com/.well-known/ai-plugin.json) → 重定向到 [https://bar.foo.example.com/.well-known/ai-plugin.json](https://bar.foo.example.com/.well-known/ai-plugin.json) 
  - 根域名：bar.foo.example.com 
- ✅ [https://foo.example.com/.well-known/ai-plugin.json](https://foo.example.com/.well-known/ai-plugin.json) → 重定向到 [https://bar.foo.example.com/baz/ai-plugin.json](https://bar.foo.example.com/baz/ai-plugin.json) 
  - 根域名：bar.foo.example.com 
- ❌ [https://foo.example.com/.well-known/ai-plugin.json](https://foo.example.com/.well-known/ai-plugin.json) → 重定向到 [https://example.com/.well-known/ai-plugin.json](https://example.com/.well-known/ai-plugin.json) 
  - 不允许将子域名重定向到父级域名 
- ❌ [https://foo.example.com/.well-known/ai-plugin.json](https://foo.example.com/.well-known/ai-plugin.json) → 重定向到 [https://bar.example.com/.well-known/ai-plugin.json](https://bar.example.com/.well-known/ai-plugin.json) 
  - 不允许将子域名重定向到同级子域名 
- ❌ [https://example.com/.well-known/ai-plugin.json](https://example.com/.well-known/ai-plugin.json) -> 重定向向 [https://example2.com/.well-known/ai-plugin.json](https://example2.com/.well-known/ai-plugin.json) 的
  - 重定向另一个域名是不允许的 

##### 清单验证

 清单本身中的特定字段必须满足以下要求：

- `api.url` - 提供给 OpenAPI 规范的 URL 必须托管在根域名的同一级别或子域名上。 
- `legal_info` - 提供的 URL 的二级域名必须与根域名的二级域名相同。 
- `contact_info` - 电子邮件地址的二级域名应与根域名的二级域名相同。 

##### 解析 API 规范 

清单中的 api.url 字段提供了一个链接到 OpenAPI 规范的链接，该规范定义了插件可以调用的 API。OpenAPI 允许指定多个服务器基本 URL。选择服务器 URL 时使用以下逻辑：

- 如果有一个与根域名完全匹配的服务器 URL，那么使用该 URL。 

- 否则，请使用第一个根域名的子域名的服务器 URL。

- 如果上述两种情况都不适用，则默认使用托管 API 规范的域名。例如，如果规范托管在 api.example.com 上，则 api.example.com 将用作 OpenAPI 规范中路由的基本 URL。 

注意：请避免为托管 API规范和任何 API 端点使用重定向，因为不能保证重定向始终会被遵循。

##### 使用 TLS 和 HTTPS 

与插件的所有流量（例如，获取 ai-plugin.json 文件、OpenAPI 规范、API 调用）都必须使用 TLS 1.2 或更高版本的有效公共证书。

##### IP 出口范围

ChatGPT 将从CIDR 块 23.102.140.112/28 的 IP 地址向您的插件发起调用。您可能希望将这些 IP 地址明确列入白名单。

对于 OpenAI 的网络浏览插件，将从 23.98.142.176/28 的 IP 地址块向网站发起调用。

#### 常见问题解答 

##### 插件数据如何使用？

插件将 ChatGPT 连接到外部应用。如果用户启用插件，ChatGPT 可能会将其对话的部分内容和所在国家或州发送到您的插件。

##### 如果我向 API 发送的请求失败会发生什么？

如果 API 请求失败，模型可能会在通知用户无法从该插件获取响应之前重试请求多达 10 次。

##### 我可以邀请人们尝试我的插件吗？

是的，所有未经验证的插件最多可以由 15 位用户安装。在发布时，只有拥有访问权限的其他开发人员才能安装插件。我们计划随着时间的推移扩大访问权限，并最终推出一种提交插件审核的流程，然后向所有用户提供。

### 插件政策 

除了上述关于我们模型禁止使用的详细说明外，我们还为开发插件的开发者提供了额外要求：

- 插件清单必须有一个明确的描述，与暴露给模型的 API 功能相匹配。 
- 不要在插件清单、OpenAPI 端点描述或插件响应消息中包含无关、不必要或欺诈性的术语或说明。这包括避免使用其他插件的说明，或试图引导或设置模型行为的说明。
-  不要使用插件绕过或干扰 OpenAI 的安全系统。 不要使用插件与真实人物进行自动化对话，无论是通过模拟类似人类的回应还是通过回复预先编程的消息。
- 分发由 ChatGPT 生成的个人通信或内容（如电子邮件、消息或其他内容）的插件必须表明该内容是由 AI 生成的。

与我们的其他使用政策一样，我们预计插件政策会随着我们对插件使用和滥用的了解而发生变化。