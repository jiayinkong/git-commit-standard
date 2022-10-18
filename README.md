# git commit 规范

在项目中实现 git commit 规范，可以带来以下好处：
- 结合 Eslint+Prettier 做代码格式化
- 把杂乱无章的代码提交信息约定成有统一的格式，这种格式还可以根据团队的需要，自行制定
- 使用 Commitizen 代码提交规范工具，能减少 git commit 信息描述的麻烦
- 结合git hooks 钩子，不符合约定式提交规范的时候，能阻止当前提交，抛出错误提示，或者自动修复格式化问题，保证上库代码是统一格式
- husky + commitlint 检查代码是否符合规范要求

## 制定代码提交规范
目前最为广泛使用的是以 Angular 团队规范延伸出的 Conventional Commits specification（约定式提交）。[具体了解 onventional Commits specification](https://www.conventionalcommits.org/zh-hans/v1.0.0/) 

其格式遵循以下规范：

```shell
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

但一般在项目中我们直接写 `<type>[optional scope]: <description>` 就可以了，body、footer 省略。


接下来需要做以下几件事

1. 安装 cz-customizable
```shell
pnpm i cz-customizable@6.3.0 -D
```

2. 配置 `package.json`
```json
{
    "config": {
        "commitizen": {
        "path": "node_modules/cz-customizable"
        }
    }
}
```

3. 根目录下新建 `.cz-config.js`
```js
module.exports = {
  // 可选类型
  types: [
    { value: 'feat', name: 'feat:     新功能' },
    { value: 'fix', name: 'fix:      修复' },
    { value: 'docs', name: 'docs:     文档变更' },
    { value: 'style', name: 'style:    代码格式(不影响代码运行的变动)' },
    {
      value: 'refactor',
      name: 'refactor: 重构(既不是增加feature，也不是修复bug)'
    },
    { value: 'perf', name: 'perf:     性能优化' },
    { value: 'test', name: 'test:     增加测试' },
    { value: 'chore', name: 'chore:    构建过程或辅助工具的变动' },
    { value: 'revert', name: 'revert:   回退' },
    { value: 'build', name: 'build:    打包' }
  ],
  // 消息步骤
  messages: {
    type: '请选择提交类型:',
    customScope: '请输入修改范围(可选):',
    subject: '请简要描述提交(必填):',
    body: '请输入详细描述(可选):',
    footer: '请输入要关闭的issue(可选):',
    confirmCommit: '确认使用以上信息提交？(y/n/e/h)'
  },
  // 跳过问题
  skipQuestions: ['body', 'footer'],
  // subject文字长度默认是72
  subjectLimit: 200
}
```

这部分内容可以根据团队的需要，自行制定规范内容。

4. 全局安装 Commitizen

`npm i commitizen@4.2.4 -g`

Commitizen 是一个代码提交规范工具，在终端输入 `git cz` 代替 `git commit -m`

在 windows 的 git base 可能会有问题，但可以直接在 vsCode 打开终端，输入 `git cz` 这样就可以根据刚才配置的 `.cz-config.js` 给出输入提示，我们可以根据提示快速完成 commit 的描述提交。

![git cz](./img/git_commit/git_cz.png)

## git hooks + husky + commitlint
通过以上的 Commitizen + cz-customizable 我们已经制定好了代码提交规范，但是还存在这样的问题，如果有人不小心使用 `git commit -m` 这样代码还是可以提交的，那有没有办法实现当不符合提交规范的时候阻止提交呢？有的，通过使用 git hooks 就可以做到。

### git hooks
git hooks 也叫 git 钩子、git 回调方法， git 在执行某个事件之前或之后进行一些其他额外的操作。

[了解gitHooks](https://git-scm.com/docs/githooks)


用得最多的是 `pre-commit`、`commit-msg`。

- `commit-msg` 可用于将消息规范化为某个项目标准格式，还可用于在检查消息文件后拒绝提交。

- `pre-commit` 不接受任何参数，在获取提交日志消息并进行提交之前被调用，脚本 `git commit` 以非零状态退出会导致命令在创建提交之前终止。

绕过 `pre-commit`、`commit-msg` 可使用命令：
```shell
git commit --no-verify
```

### husky
`husky` 是 git hooks 工具。使用 `husky` 需要做以下几件事

1. 安装 husky
```shell
npm install husky@7.0.1 --save-dev
```

2. 添加 `husky` 脚本到 `package.json`
```shell
npm set-script prepare "husky install"
```

3. 启动 hooks，生成 `.husky` 文件夹
```shell
pnpm prepare
```

4. 安装 commitlint 相关依赖
```shell
pnpm install --save-dev @commitlint/config-conventional@12.1.4 @commitlint/cli@12.1.4
```

根目录下新建 `commitlint.config.js`
```js
module.exports = {
  // 继承的规则
  extends: ['@commitlint/config-conventional'],
  // 定义规则类型
  rules: {
    // type 类型定义，表示 git 提交的 type 必须在以下类型范围内
    'type-enum': [
      2,
      'always',
      [
        'feat', // 新功能 feature
        'fix', // 修复 bug
        'docs', // 文档注释
        'style', // 代码格式(不影响代码运行的变动)
        'refactor', // 重构(既不增加新功能，也不是修复bug)
        'perf', // 性能优化
        'test', // 增加测试
        'chore', // 构建过程或辅助工具的变动
        'revert', // 回退
        'build' // 打包
      ]
    ],
    // subject 大小写不做校验
    'subject-case': [0]
  }
}
```

[commitlint 默认配置参考](https://github.com/conventional-changelog/commitlint/blob/master/@commitlint/config-conventional/index.js)

4. 添加生命周期钩子

- pre-commit

```shell
npx husky add .husky/pre-commit "pnpm lint && pnpm format"
```

终端输入 `git commit -m 'update'` 查看是否生效:

![pre commit](./img/git_commit/pre_commit_eslint.png)


- commit-msg

```shell
# 添加 commitlint 的 hooks 到 husky
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'
```

终端输入 `git commit -m 'update'` 查看是否生效

![commit msg](./img/git_commit/commit_msg.png)

接着试下成功提交的效果：

```shell
git add .
git cz
```

![commit success](./img/git_commit/commit_success.png)
