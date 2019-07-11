# zw-cli
通过交互式命令从github中下载代码生成新项目-学习记

#### 使用姿势
- 全局安装npm i zw-vue-cli -g
- 在项目目录下运行命令zwcli
- 更加提示输入项目名称和版本号即可


#### 参考文档

- [Node.js 命令行程序开发教程](https://www.kancloud.cn/kancloud/command-line-with-node/48657)
- [download-git-repo包从远程(GitHub, GitLab, Bitbucket)拉取文件到本地](https://www.npmjs.com/package/download-git-repo)
- [commander.js包 在命令行中显示](https://github.com/tj/commander.js/blob/master/Readme_zh-CN.md)
- [Ora 在控制台显示当前加载状态的](https://github.com/sindresorhus/ora)
- [shelljs 模块重新包装了 child_process，调用系统命令](https://www.npmjs.com/package/shelljs)
- [yargs 获取命令行参数](https://www.kancloud.cn/kancloud/command-line-with-node/48652)
- [chalk 控制控制台输出文字的颜色](https://github.com/chalk/chalk)
- [NodeJs交互式命令行工具inquirer](https://www.npmjs.com/package/inquirer)

#### 为什么要学习这个

看vue-cli可以用交互式的命令来生成一个项目感觉很好玩，所以我也来学学如何做。一下实现的功能其实一行命令就能解决(git clone --depth=1 https://github.com/ant-design/ant-design-pro.git my-project)

#### 我要实现什么功能

通过在命令行交互式输入项目名、版本号、选择模板地址后远程(github/gitlab)拉取代码到本地并创建项目.

#### 什么都不懂，如何开始

搜索到了一篇[Node.js 命令行程序开发教程](https://www.kancloud.cn/kancloud/command-line-with-node/48657)，瞬间就有了方向。

#### 具体做什么

- 像vue-cli那样输入命令 vue create hello-world就能开始执行交互程序。
- 交互式命令行
- 远程拉取项目到本地
- 修改package.json文件中的name和版本号

#### 怎么做

- 像vue-cli那样输入命令 vue create hello-world就能开始执行交互程序

从[Node.js 命令行程序开发教程-可执行脚本](https://www.kancloud.cn/kancloud/command-line-with-node/48648)中学到，
可以在package.json文件中的bin下面设置命令，然后通过npm link就可将命令加入到环境变量中,像下面这样配置 运行zwcli就会执行"bin/create-zw-template"脚本。相当于执行node bin/create-zw-template(记得npm link, npm unlink是用来设置刚才的删除环境变量的)

<pre>
"bin": {
    "zwcli": "bin/create-zw-template"
},
</pre>


- 交互式命令行

使用[NodeJs交互式命令行工具inquirer](https://www.npmjs.com/package/inquirer)来实现交互式命令。
    - 首先编写问题
    - 获取问答结果
<pre>
const questions = [{
    type: 'input', // type为答题的类型 
    name: 'projectName', // 本题的key，待会获取answers时通过这个key获取value
    message: 'projectName：', // 提示语
    validate (val) {
        if(!val) { // 验证一下输入是否正确
            return '请输入文件名';
        }
        if(fs.existsSync(val)) { // 判断文件是否存在
            return '文件已存在';
        }else {
            return true;
        };
    }
},{
    type: 'input',
    name: 'version',
    message: 'verson(1.0.0)：',
    default: '1.0.0',
    validate (val) {
        return true;
    }
},{
    type: 'input',
    name: 'repository',
    message: 'repository(zwfun/zw-vue-cli)：',
    default: 'zwfun/zw-vue-cli'
}];

inquirer
    .prompt(questions)
    .then(answers => {
        // 获取答案
        const version = answers.version;
        const projectName = answers.projectName;
        const repository = answers.repository;
       
        spinner.start();
        spinner.color = 'green';
        downloadTemplate({ repository, version, projectName });
    });
</pre>

- 获取到答案后，就从远程拉取代码下来
远程拉取代码我们选用[download-git-repo包从远程(GitHub, GitLab, Bitbucket)拉取文件到本地](https://www.npmjs.com/package/download-git-repo)

为了更好的用户体验我们需要引入[Ora 在控制台显示当前加载状态的](https://github.com/sindresorhus/ora)，来让加载过程更直观好像点

<pre>
/**
 * @description: 下载模板
 * @param {type} 
 * @return: 
 */
const downloadTemplate = function({ repository, version, projectName }) {
    // repository模板地址  projectName项目名称 // clone 是否是克隆
    download(repository, projectName, function (err) {
        console.log(err ? '模板加载错误' : '模板加载结束');
        if(err !== 'Error') {
            editFile({ version, projectName });
        }
    })
};

</pre>

- 拉取代码后，修改package.json文件
使用node自带的api就行

<pre>

const editFile = function({ version, projectName }) {
    // 读取文件
    fs.readFile(`${process.cwd()}/${projectName}/package.json`, (err, data) => {
        if (err) throw err;
        // 获取json数据并修改项目名称和版本号
        let _data = JSON.parse(data.toString())
        _data.name = projectName
        _data.version = version
        let str = JSON.stringify(_data, null, 4);
        // 写入文件
        fs.writeFile(`${process.cwd()}/${projectName}/package.json`, str, function (err) {
            if (err) throw err;
        })
        spinner.succeed();
    });
};
</pre>