---
title: 获取托管仓库token
categories:
  - 笔记
excerpt: '获取 github、gitee、gitlab 的 token'
description: ''
date: 2023-12-11 09:50:44
---


# Gitee

## 获取到用户 Token

### 创建应用

参考[文档](https://gitee.com/api/v5/oauth_doc#/list-item-3)创建一个应用

### 获取用户 token

> 参考链接：[文档链接](https://gitee.com/api/v5/oauth_doc#/list-item-2)

1. 跳转页面：https://gitee.com/oauth/authorize?client_id=${**Client ID**}&redirect_uri=${**应用回调地址\*\*}&response_type=code【注意： redirect_uri 必须完全和配置的一致】
1. 跳转到上面的 url，会进入到下面的这个页面

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/145998/1630996474251-bd209a40-a60d-424e-bd88-ceb5547b6e36.png#clientId=uc88975a1-01c9-4&from=paste&height=235&id=uf3601d7a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=938&originWidth=1188&originalType=binary&ratio=1&size=377494&status=done&style=none&taskId=u8e815496-a633-4ad4-b481-4692b46df37&width=297)
a. 如果用户选择了拒绝，返回参数

```javascript
{ error: 'access_denied', error_description: '用户或服务器拒绝了请求' }
```

b. 如果用户选择了同意，返回参数

```javascript
{
  code: 'xxxxxxxxxxxxxxxxxxxxx';
}
```

3. 调用 post 请求 `https://gitee.com/oauth/token`，params 参数：

```javascript
{
    code: '第二步返回的 code',
    grant_type: 'authorization_code',
    client_id: 'Client ID',
    redirect_uri: '应用回调地址，需要和第一步的地址完全一致',
    client_secret: 'Client Secret',
 }
```

返回的结构：

```javascript
{
  statusCode: 200,
  body: '{"access_token":"xxxxxxxxxxxxx","token_type":"bearer","expires_in":86400,"refresh_token":"xxxxxxxxxxxxx","scope":"user_info projects hook groups gists","created_at":1630995493}'
}
```

4. 调用 post 请求 `https://gitee.com/oauth/token?grant_type=refresh_token&refresh_token={上次返回的 refresh_token}`，返回结构和步骤三一致

## fork 仓库

1. 先调用 [fork 仓库](https://gitee.com/api/v5/swagger#/postV5ReposOwnerRepoForks)接口
1. 然后再[修改仓库配置](https://gitee.com/api/v5/swagger#/patchV5ReposOwnerRepo)接口
1. [开通 gitee go](https://gitee.com/api/v5/swagger#/postV5ReposOwnerRepoOpen)接口

# Github

> [参考链接](https://docs.github.com/en/free-pro-team@latest/rest/reference/repos#create-a-repository-using-a-template)

## 获取用户 Token

### 创建 GitHub Apps

1. 登录 github -> 进入个人中心 -> Developer settings -> 创建 app
2. 记录下 Client ID、Public link、Client secret 后面获取 token 需要这些参数
3. 通过回调接收 code 兑换 token

### 获取 code & token

1. [参考文档](https://docs.github.com/cn/free-pro-team@latest/developers/apps/identifying-and-authorizing-users-for-github-apps)
1. 具体步骤

获取 code ：跳转到这个链接带上 client_id https://github.com/login/oauth/authorize?client_id=\${client_id} ，github 会重定向到你 CallbackURL 地址中并带上 code 信息
获取 token： 调用这个链接在响应的参数里面找到 token 信息，带上 client_id，client_secret，code https://github.com/login/oauth/access_token?client_id=\${client_id}&client_secret=\${client_secret}&code=\${code}

```javascript
function getToken(fcContext, params, callback) {
  let token = '';
  return new Promise((resolve, reject) => {
    request.get(
      {
        url: `https://github.com/login/oauth/access_token?client_id=${client_id}&client_secret=${client_secret}&code=${params.code}`,
      },
      (err, res, body) => {
        token = qs.parse(body).access_token;
        resolve(token);
      },
    );
  }).then((token) => {
    const res = {
      data: { token },
      success: true,
      message: 'success',
    };
    callback(null, res);
  });
}
```

## 根据模版创建仓库

### 前置条件

示例代码的仓库勾选 Template repository [文档](https://docs.github.com/cn/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/creating-a-repository-from-a-template)
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/145998/1608789846948-a53d0130-2a48-42af-b6f8-5e739321b701.png#height=307&id=ZIVm0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=614&originWidth=2406&originalType=binary&ratio=1&size=287744&status=done&style=none&width=1203)

### 调用接口

```javascript
// 下载依赖
npm install @octokit/core

// 创建客户端
const { Octokit } = require("@octokit/core");
const octokit = new Octokit({ auth: '用户的 token' });

// 创建
async function generate () {
  try {
    const a = await octokit.request('POST /repos/{template_owner}/{template_repo}/generate', {
      template_owner: '示例代码的仓库拥有者', // wss-git
      template_repo: '示例代码的仓库名称', // upload-oss-regions
      owner: '目标仓库拥有者',
      name: '目标仓库仓库名称',
      description: '仓库说明',
      private: true, // true 私有， false 公开
      mediaType: {
        previews: [
          'baptiste'
        ]
      }
    });
    console.log(a);
  } catch (e) {
    console.error(e);
  }
}


```

## 创建仓库

> // TODO
>
> [https://docs.github.com/cn/rest/reference/repos#autolinks](https://docs.github.com/cn/rest/reference/repos#autolinks)
> POST /user/repos

```javascript
async function createRepo({ name, description }) {
  const res = await octokit.request('POST /user/repos', {
    name,
    description,
  });
}
```

## 创建单文件

```javascript
await octokit.request('PUT /repos/{owner}/{repo}/contents/{path}', {
  owner,
  repo: name,
  path: '.github/workflows/cicd.yml',
  message: 'init cicd',
  content: Buffer.from('文件内容').toString('base64'),
});
```

# Gitlab

> 示例仓库拥有者：serverlessfans

## 获取到用户的 Token

### 创建应用

1. 进入到账号设置，跳转到[创建](https://gitlab.com/-/profile/applications)应用
1. 创建应用，获取到 Application ID、Secret、Callback URL 三个字段

### 获取 token

1. 跳转获取 token 的 url： `https://gitlab.com/oauth/authorize`，参数 client_id（Application ID）、redirect_uri（Callback URL）、response_type（固定值 code）、state（可选参数，用于确认请求和回调的状态，OAuth 建议以此来防止 [CSRF 攻击](https://owasp.org/www-community/attacks/csrf)）

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/145998/1630389521267-ed95ce1c-fa9e-4759-bafc-b0e5bc3b0a0e.png#clientId=u64622b97-05b1-4&from=paste&height=57&id=u5501caf6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=114&originWidth=1558&originalType=binary&ratio=1&size=180190&status=done&style=none&taskId=ud1e66e40-b383-4959-a14f-e4a596b36bc&width=779)

2. 跳转到上面的 url，会进入到下面的这个页面

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/145998/1630389820517-7c9ac0a5-f7a8-4f50-acaf-a1100516f2a8.png#clientId=u64622b97-05b1-4&from=paste&height=319&id=swSm5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1054&originWidth=2032&originalType=binary&ratio=1&size=317020&status=done&style=none&taskId=ubaa1965e-3b12-4645-9834-6e6be377d2a&width=615)

1.  如果用户点击了 Deny，回调地址 query 参数解析如下，**此时需要抛出异常**

```javascript
{
  error: 'access_denied',
  error_description: 'The resource owner or authorization server denied the request.',
  state: '0.17789379632421753'
}
```

2.  如果用户点击了 Authorize，回调地址 query 参数解析：

```javascript
{
  code: 'xxxxxxxxxxxxx',
  state: '0.17789379632421753'
}
```

3. 调用 post 请求 `https://gitlab.com/oauth/token`，params 参数：

state: 上一步返回的参数 state
code：上一步返回的参数 code
client_id：Application ID
client_secret：Secret
redirect_uri: Callback URL
grant_type: 'authorization_code'

4. 返回值

```javascript
{
  "access_token": "*****",
  "token_type": "Bearer",
  "refresh_token": "******",
  "scope": "api",
  "created_at": 1630390334
}
```

```javascript
如果身份验证信息无效或丢失，GitLab 会返回一条错误消息，状态代码为401：
令牌撤销： {"error":"invalid_token","error_description":"Token was revoked. You have to re-authorize from the user."}
令牌过期： 暂未捕捉到

token 过期或者刷新 token：
parameters = 'client_id=APP_ID&client_secret=APP_SECRET&refresh_token=REFRESH_TOKEN&grant_type=refresh_token&redirect_uri=REDIRECT_URI&code_verifier=CODE_VERIFIER'
RestClient.post 'https://gitlab.example.com/oauth/token', parameters
```

## fork 仓库代码

### 前置条件

1. 拥有用户的 token
1. 拥有示例仓库的 token，示例仓库的名称

### 代码调用

```javascript
const request = require('request');
const gitlabUrl = 'https://gitlab.com/api/v4';

async function httpRequest({ url, method = 'GET', headers, form, body }) {
  return await new Promise((resolve, reject) => {
    request({ url, method, headers, body, form }, (error, response, b) => {
      if (error) {
        reject(error);
      }
      resolve({ statusCode: response.statusCode, body: b });
    });
  });
}

export async function forkRepo({
  template_owner, // 模版仓库的拥有者
  template_repo, // 模版仓库的 repo 路径
  access_token, // 用户授权给程序的密钥
  token_type = 'Bearer', // 授权给程序一般都是给 Bearer

  name, //  分配给结果项目的名称
  path, // 分配给结果项目的路径
}) {
  const encodeUri = encodeURIComponent(`${template_owner}/${template_repo}`);
  const createProjectRes = await httpRequest({
    url: `${gitlabUrl}/projects/${encodeUri}/fork?access_token=${access_token}&token_type=${token_type}`,
    method: 'POST',
    form: {
      name,
      path,
    },
  });
  console.log(encodeUri, '/fork:: ', JSON.stringify(createProjectRes));

  if (![200, 201].includes(createProjectRes.statusCode)) {
    throw new Error(createProjectRes.body);
  }
  const { id } = JSON.parse(createProjectRes.body);
  console.log('id: ', id);

  if (id) {
    let timer = setInterval(async () => {
      const getRepoUri = `${gitlabUrl}/projects/${id}?access_token=${access_token}&token_type=${token_type}`;
      const projectRes = await httpRequest({ url: getRepoUri });

      if (![200, 201].includes(createProjectRes.statusCode)) {
        throw new Error(createProjectRes.body);
      }

      const { import_status, import_error } = JSON.parse(projectRes.body);
      console.log('import_status:: ', import_status);
      // import_status 可取值：
      // none
      // queued 状态表示已接收到导出请求，并且当前处于要处理的队列中.
      // started 状态表示导出过程已开始并且当前正在进行中. 它包括导出过程，对生成的文件执行的操作，例如发送电子邮件通知用户下载文件，将导出的文件上传到 Web 服务器等.
      // finished 状态是在导出过程完成并且已通知用户之后.
      // failed 如果状态failed ，它将在import_error下包含导入错误消息.
      // regeneration_in_progress 是可以下载导出文件且正在处理生成新导出文件的请求.
      if (import_status === 'finished') {
        clearInterval(timer);
      } else if (import_status === 'failed') {
        throw new Error(import_error);
        clearInterval(timer);
      }
    }, 2 * 1000);
  }
}
```

