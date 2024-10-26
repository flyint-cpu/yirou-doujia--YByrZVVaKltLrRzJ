
作为一个前端，基本上每次初始化项目都会用到脚手架，通过一些脚手架可以快速的搭建一个前端的项目并集成一些所需的功能模块，避免自己每次都手动一个一个去安装。安装各个包的这个过程其实没啥营养，通过封装一个脚手架来跳过这个步骤，把精力聚焦到功能研发上。


由于最近自己在写项目都是相同的技术栈：`Nextjs + TailwindCSS + TypeScript + ShadcnUI` ，有时候如果忘记了 `ShadcnUI` 安装的命令的话，还得去 [ShadcnUI 官网](https://github.com) 去查看先关的文档。正好最近在看前端工程化相关的内容，简单封装一个 `cli` ，为了以后给 DevNow 来扩展一些内容模版做些基建。说 · 感觉就逼格就上来了😎。


顺便给自己的开源博客项目打个广告，欢迎大家体验、star：
[DevNow](https://github.com) 是一个精简的开源技术博客项目模版，支持 Vercel 一键部署，支持评论、搜索等功能，欢迎大家体验。


## 1\. 初始化


新建一个 cli 文件夹 用来存放我们相关的内容。



```
|-- cli
  |-- index.js

```

我们主要的 cli 内容都在 index.js 中，接下来我们来实现一下。先来看完整代码如下：



```
#!/usr/bin/env node

const { Command } = require('commander');
const { execSync } = require('child_process');
const prompts = require('prompts');

const program = new Command();
let projectPath = '';
async function initProject() {
  console.log('欢迎使用项目初始化工具！');

  const opts = program.opts();

  // 询问用户输入和选择
  const response = await prompts([
    {
      type: 'text',
      name: 'projectName',
      message: '请输入项目名称:',
      validate: (name) => (name ? true : '项目名称不能为空'),
      initial: projectPath
    },
    {
      type: 'select',
      name: 'template',
      message: '请选择项目模板:',
      choices: [
        { title: 'JavaScript', value: 'javascript' },
        { title: 'TypeScript', value: 'typescript' }
      ],
      initial: opts.ts ? 1 : 0 // 默认 TypeScript
    },
    {
      type: 'select',
      name: 'tailwindCSS',
      message: 'Would you like to use Tailwind CSS?',
      choices: [
        { title: 'No', value: false },
        { title: 'Yes', value: true }
      ],
      initial: opts.tailwind ? 1 : 0 // 默认 TypeScript
    },
    {
      type: 'select',
      name: 'eslint',
      message: 'Would you like to use eslint',
      choices: [
        { title: 'No', value: false },
        { title: 'Yes', value: true }
      ],
      initial: opts.eslint ? 1 : 0 // 默认 eslint
    },
    {
      type: 'select',
      name: 'shadcnUI',
      message: 'Would you like to use Shadcn/ui?',
      choices: [
        { title: 'No', value: false },
        { title: 'Yes', value: true }
      ],
      initial: opts.shadcnUI ? 1 : 0 // 默认 shadcnUI
    }
  ]);

  // 项目初始化命令拼接
  const templateFlag = (response.template || opts.ts) === 'typescript' ? '--typescript' : '';
  const eslintFlag = response.eslint || opts.eslint ? '--eslint' : '';
  const tailwindCSS = response.tailwindCSS || opts.tailwind ? '--tailwind' : '';

  try {
    // 执行 Next.js 初始化命令
    console.log(`正在创建项目：${response.projectName}...`);
    execSync(
      `pnpm create next-app@latest ${response.projectName} ${templateFlag} ${eslintFlag} ${tailwindCSS} --turbopack --app --src-dir --import-alias "@/*"`,
      { stdio: 'inherit' }
    );

    // 切换到项目目录
    process.chdir(response.projectName);

    // 安装 shadcn/ui
    if (response.shadcnUI || opts.shadcnUI) {
      console.log('安装 shadcn/ui...');
      execSync('pnpm dlx shadcn@latest init -d', { stdio: 'inherit' });
    }

    console.log('项目初始化完成！');
    console.log(`请运行以下命令进入项目并启动开发服务器：`);
    console.log(`  cd ${response.projectName}`);
    console.log(`  pnpm dev`);
  } catch (error) {
    console.error('项目创建失败:', error.message);
  }
}

// 定义 CLI 命令
program
  .name('create-next-app')
  .description('创建一个 Next.js 项目')
  .argument('', '项目名称')
  .option('--ts, --typescript', 'Initialize as a TypeScript project. (default)')
  .option('--tailwind', 'Initialize with Tailwind CSS config. (default)')
  .option('--eslint', 'Initialize with ESLint config.')
  .option('--shadcnUI', 'Enable ShadcnUi.')
  .action((name) => {
    projectPath = name;
    initProject();
  });

// 解析命令行参数
program.parse(process.argv);

```

### 1\.1 [execSync](https://github.com)


`execSync` 是 `Node.js` 的内置模块 `child_process` 中的一个同步执行命令的方法。它会在当前进程中运行指定的系统命令，并返回结果。


### 1\.2 [Commadner.js](https://github.com):[蓝猫机场加速器](https://dahelaoshi.com)


Commander.js 完整的 node.js 命令行解决方案。编写代码来描述你的命令行界面。 Commander 负责将参数解析为选项和命令参数，为问题显示使用错误，并实现一个有帮助的系统。



```
// 定义 CLI 命令
program
  .name('create-next-app')
  .description('创建一个 Next.js 项目')
  .argument('', '项目名称')
  .option('--ts, --typescript', 'Initialize as a TypeScript project. (default)')
  .option('--tailwind', 'Initialize with Tailwind CSS config. (default)')
  .option('--eslint', 'Initialize with ESLint config.')
  .option('--shadcnUI', 'Enable ShadcnUi.')
  .action((name) => {
    projectPath = name;
    initProject();
  });

// 解析命令行参数
program.parse(process.argv);

```

这段代码通过 `commander.js` 定义了一个功能丰富的 `CLI` 工具，允许用户通过不同的选项初始化一个定制化的 `Next.js` 项目。 `parse(process.argv)` 是核心步骤，用来解析命令行中的参数并执行对应的逻辑。
我们通过 `--help` 可以看到一些可选的配置参数等等内容。


![](https://r2.laughingzhu.cn/05e7668fec0572c41f20271f38631253-93e991.webp)


### 1\.3 [prompts](https://github.com)


`prompts` 是一个轻量级、用户友好的交互式 `CLI` 库，用于在命令行中向用户提出问题并收集输入。它支持多种类型的提示，比如文本输入、选择、多选、确认等，允许开发者灵活地设计命令行应用的交互体验。


通过使用 `prompts` 库，我们可以在 `CLI` 中实现类似 `Next.js` 的交互体验，这样可以在初始化项目时根据需要选择不同的选项，体验会更好一点。



```
const response = await prompts([
  {
    type: 'text',
    name: 'projectName',
    message: '请输入项目名称:',
    validate: (name) => (name ? true : '项目名称不能为空'),
    initial: projectPath
  },
  {
    type: 'select',
    name: 'template',
    message: '请选择项目模板:',
    choices: [
      { title: 'JavaScript', value: 'javascript' },
      { title: 'TypeScript', value: 'typescript' }
    ],
    initial: opts.ts ? 1 : 0 // 默认 TypeScript
  },
  {
    type: 'select',
    name: 'tailwindCSS',
    message: 'Would you like to use Tailwind CSS?',
    choices: [
      { title: 'No', value: false },
      { title: 'Yes', value: true }
    ],
    initial: opts.tailwind ? 1 : 0 // 默认 TypeScript
  },
  {
    type: 'select',
    name: 'eslint',
    message: 'Would you like to use eslint',
    choices: [
      { title: 'No', value: false },
      { title: 'Yes', value: true }
    ],
    initial: opts.eslint ? 1 : 0 // 默认 eslint
  },
  {
    type: 'select',
    name: 'shadcnUI',
    message: 'Would you like to use Shadcn/ui?',
    choices: [
      { title: 'No', value: false },
      { title: 'Yes', value: true }
    ],
    initial: opts.shadcnUI ? 1 : 0 // 默认 shadcnUI
  }
]);

```

这里通过配置了一些选项，结合了 command 初始化命令行的默认选项，效果如下：


![](https://r2.laughingzhu.cn/cff35e1653e99fbf178b98c7eccaee02-c20b80.webp)


### 1\.4 执行代码



```
try {
  // 执行 Next.js 初始化命令
  console.log(`正在创建项目：${response.projectName}...`);
  execSync(
    `pnpm create next-app@latest ${response.projectName} ${templateFlag} ${eslintFlag} ${tailwindCSS} --turbopack --app --src-dir --import-alias "@/*"`,
    { stdio: 'inherit' }
  );

  // 切换到项目目录
  process.chdir(response.projectName);

  // 安装 shadcn/ui
  if (response.shadcnUI || opts.shadcnUI) {
    console.log('安装 shadcn/ui...');
    execSync('pnpm dlx shadcn@latest init -d', { stdio: 'inherit' });
  }

  console.log('项目初始化完成！');
  console.log(`请运行以下命令进入项目并启动开发服务器：`);
  console.log(`  cd ${response.projectName}`);
  console.log(`  pnpm dev`);
} catch (error) {
  console.error('项目创建失败:', error.message);
}

```

这里就是主要的执行内容，通过前面的配置参数来定制化执行，首先是 `Nextjs` 相关内容，可以看到我默认了一些配置，这些看个人的需求了，感觉这些东西默认的会更好。最后就是根据配置参数判断是否要安装 `ShadcnUI` .


其实到这里 CLI 的基本内容就完事了，我们可以在本地执行 `node index.js cli-test --tailwind` 去测试下。



> 注意:
> 本地 Node 环境中需要安装 `commander` 和 `prompts` 两个库，不然会报错。
> 
> 
> 当通过 npx create\-next\-app 等命令初始化项目时，你不需要担心 commander 等依赖的安装。工具本身已经打包好了这些依赖，并通过 npx 或 pnpm dlx 临时加载执行，简化了使用流程。


## 2\. 发布


这里简单记录下吧，这个应该很多人已经会了，


### 2\.1 npm init


首先通过 `npm init` 初始化，通过提示可以生成一些配置文件，这个时候项目结构应该是:



```
|-- cli
  |-- index.js
  |-- package.json

```

package.json 主要内容如下:



```
{
  "dependencies": {
    "commander": "^12.1.0",
    "prompts": "^2.4.2"
  },
  "name": "create-devnow-app",
  "version": "0.0.6",
  "description": "create devnow app",
  "main": "./index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "LaughingZhu",
  "license": "MIT",
  "bin": {
    "create-devnow-app": "./index.js"
  }
}

```

确保你的 package.json 文件中定义了正确的 bin 配置。这样才能执行。


## 2\.2 登录



```
npm login

```

## 2\.3 发布



```
npm publish

```

### 使用自己专属的 CLI 😏



```
pnpm dlx create-devnow-app@latest my-app

```

![](https://r2.laughingzhu.cn/d4e8fa681d7d9103d7953bae920a9f95-8ce943.webp)


大功告成，这样在后续就可以直接使用了，如果业务中需要其他的一些配置的话，可以通过相同的方式集成，比如 [T3](https://github.com) 这个项目，就集成了 `TRPC` 等内容到脚手架中，可以方便快速构建一个全栈的项目。


