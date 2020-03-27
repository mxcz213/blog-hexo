---
title: TypeScript-Node实现下载简书文章图片工具
---
####写在前面
经常的写作的人都有备份的好习惯，为了防止自己的文章丢失，简书提供了`下载所有文章`功能，可以让作者将文章下载到本地保存，或者上传到自己的站点。
![](https://upload-images.jianshu.io/upload_images/5541401-fa175038ada0a6cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但是简书的图片是放在专门的图片服务器上的，下载所有文章并不包含文章中的所有图片。所以我们现在写个小工具，通过命令行的方式将文章中的所有图片下载本地保存。

####需求实现步骤
* 下载简书文章，解压到 A 目录；
* 建一个 TypeScript + Node 项目，读取 A 目录中的所有 .md 文件；
* 提取文件内容中的图片链接，下载下来；
* 把下载的图片放到 B 目录/当前文章/ 中，用来分类；
* 重构优化代码；

下面按照这几个步骤一步步完成简书下载图片工具。

####下载简书文章
进入`我的简书 ->账号管理 打包下载全部的简书文章`即可，我是下载到了这个目录 `D:\jianshu_article\user-5541401-1565071963`，这个目录下的所有文件都是`文集/文章`的格式。接下来开始搭建项目结构。

####TypeScript Node 搭建项目
先在 github 上新建一个仓库，然后 clone 下来。开发工作一直在 master 分支上，然后每完成一步需求，新建一个分支用来保留记录，以后看的时候更清晰。

新建一个仓库然后 clone 下来：
`git clone git@github.com:mxcz213/download-jianshu-images.git`

#####开始项目搭建：
* 生成 package.json 文件；
```
npm init -f
```
* 下载项目依赖 ：typescript node 的ts 版本，download下载文件包，runscript 用来执行 shell 命令，ts-node 用来开发调试；
```
npm install @types/node download runscript ts-node typescript --save-dev
```
* 配置 tsconfig 文件，用来按照这个规则编译 ts 文件为 js 文件。执行命令 `tsc --init`，自动生成 `tsconfig.json` 文件；
```
//tsconfig.json
{
  "compilerOptions": {
    "target": "es5",  
    "module": "commonjs",
    "outDir": "./dist/", 
    "strict": true,
    "esModuleInterop": true                  
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```
* 配置 package.json 文件的 scripts 字段，启动项目和编译命令
```
{
  "name": "download-jianshu-images",
  "version": "1.0.0",
  "description": "Node + typescript 实现下载简书文章中所有的图片链接",
  "main": "dist/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "tsc",
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/mxcz213/download-jianshu-images.git"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/mxcz213/download-jianshu-images/issues"
  },
  "homepage": "https://github.com/mxcz213/download-jianshu-images#readme",
  "devDependencies": {
    "@types/node": "^12.6.9",
    "download": "^7.1.0",
    "runscript": "^1.4.0",
    "ts-node": "^8.3.0",
    "typescript": "^3.5.3"
  }
}
```
* 配置 vscode 的调试脚本 launch.json
```
//.vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [

        {
            "name": "Current TS File",
            "type": "node",
            "request": "launch",
            "program": "${workspaceRoot}/node_modules/ts-node/dist/bin.js",
            // "program": "${workspaceRoot}/test.js",
            "args": [
                "${relativeFile}"
            ],
            "cwd": "${workspaceRoot}",
            "protocol": "inspector"
        }
    ]
}
```
* 添加 .gitignore 文件配置忽略提交的目录
```
//.gitignore
/node_modules
```
* 新建 dist 目录用来放编译之后的 js 文件
* 新建 src 源代码文件目录

####具体代码实现，新建 src/index.ts 文件
```
//src index.ts
const fs = require('fs')
const path = require('path')
const runScript = require('runscript')
const download = require('download')

//windows中用户复制的目录
let originDir: string = 'D:\\jianshu_article\\user-5541401-1565071963\\'
let targetDir: string = 'E:\\workCode\\download-jianshu-images\\jianshu_article\\'

const readmeUrlReg: RegExp = /\s!\[\]\(https:\/\/\upload-images.jianshu.io\/upload_images\/[a-zA-Z0-9-_?%./]+\)\s/g
const imageUrlReg: RegExp = /https:\/\/\upload-images.jianshu.io\/upload_images\/[a-zA-Z0-9-_?%./]+/g

//用户通过命令行工具输入命令比如：node dist/index.js 简书解压目录 目标存储图片目录
process.argv.forEach((val, index) => {
    console.log(`${index}: ${val}`)
});

try {
    originDir = process.argv[2] ? process.argv[2] : originDir
    targetDir = process.argv[3] ? process.argv[3] : targetDir
} catch(e) {
    console.log('获取命令参数错误', e)
}

const downloadImages = (imgurl: string[], path: string) => {   
    let newUrlArr: any[] = []
    imgurl.forEach((item: any) => {
        if(item.match(imageUrlReg)){
            newUrlArr.push(item.match(imageUrlReg)[0])
        }
    })
    console.log(newUrlArr)
    Promise.all(newUrlArr.map((url: string) => {
        download(url, path)
    })).then(() => {
       console.log('all files downloaded')
    })
}

const runFunction = async () => {
    //shell ls拿到所有的.md文章
    const { stdout } = await runScript('ls **/*.md', {
        cwd: originDir,
        stdio: 'pipe'
    })
    let files: string[] = stdout.toString().split('\n')
    let num: number = 0
    try {
        files.forEach((fileitem: any, index: number) => {
            if(fileitem){
                let filepath: string = fileitem.split('.md')[0].split('/').join('\\')
                let dirStr: string = `${targetDir}\\${filepath}`                
                runScript(`mkdir ${dirStr}`, { stdio: 'pipe' })
                .then((stdio: any) => {
                    let fileContent = fs.readFileSync(path.join(originDir, fileitem.split('/').join('\\')), { encoding: 'utf8'})
                    let urlList: any = fileContent.match(readmeUrlReg)
                    if(urlList && urlList.length > 0){
                        downloadImages(urlList, dirStr)
                    }
                })
            }
        })
    } catch(e) {
        console.log(e)
    }
}
runFunction()
```
* 执行命令 `npm run build` 编译 ts 文件
* 执行命令`node . D:\jianshu_article\user-5541401-1565071963 D:\jianshu_article\article_img `，下载图片
`node .` 命令会到 package.json 文件中找到 main 字段执行入口文件。
`process.argv` 会获取到命令行参数。

接下来提交文件到 master 分支：
```
git add .
git commit -m "download jianshu images"
git push
```
然后根据 master 新建一个分支，用来保存这次的提交历史：
```
git checkout -b node_tool
git pull origin master
git push
```
#### 实现工具命令，如 jianshu  ...
配置命令行，通过 package.json 文件的 bin 字段，然后新建 bin 目录，在 bin 目录下新建 jianshu 文件；
```
//package.json
{
  ...
  "bin": {
    "jianshu": "bin/jianshu"
  }
  ...
}
```
```
//bin/jianshu
#!/usr/bin/env node

require('../dist/index');
```
配置完就可以通过命令 `jianshu D:\jianshu_article\user-5541401-1565071963 D:\jianshu_article\article_img` 实现下载图片。

通过`const [, , sourceDir, targetDir] = process.argv;`来获取命令行参数。

提交代码之后，这一步同样新建 node_cli 分支用来保存历史：
```
git checkout -b node_cli
git pull origin master
git pull
```
####代码重构优化
上面的代码只是实现的简单的功能，流程并不清晰，现在来重构代码，使主流程变的清晰。

**代码重构的原则：主流程要清晰**

每个函数只做一件事，有两个以上的函数，有内部函数式，就要考虑把这每个函数放到单独的文件里，然后用模块导入的方式。

#####以上代码展现的问题：
* 1. handleDir getArticleContent 重复判断平台和路径，没有把判断平台提出来
* 2. getMarkdownImageUrls getRealImageUrl 重复使用相似的正则，没使用exec和正则的捕获组
* 3. 小函数嵌套太严重，一个函数能搞定的
* 4. 没有异常判断，没有log
* 5. 逻辑层次不清晰，分了好多层
* 6. 关键注释缺失，例如files这个是相对路径的列表，不注明的话，以后肯定不知道

所以接下来就要重构这些代码，主要根据以下分类原则来实现模块的拆分：

#####分类原则

哪些是项目独有的逻辑（业务逻辑），
哪些是通用逻辑（可复用的），
哪些是模板代码（没啥用但是要写的）

根据以上原则，拆分出来工具函数 log，文件操作；核心函数 libs。
```
//src/utils/log.ts
const log = (str: string) => {
    console.log(str);
}
 const error = (str: string) => {
    console.error(str);
}
const warn = (str: string) => {
    console.warn(str);
}
export {
    log,
    error,
    warn
}
```
```
//src/utils/fs.ts
const fs = require('fs');
const runScript = require('runscript');
const download = require('download');

const read = (path: string, options?: {}) => {
    let fileContent = fs.readFileSync(path, options);
    return fileContent;
}
const createDir = async (targetDir: string) => {
    await runScript(`mkdir ${targetDir}`);
}
const deleteDir = async (targetDir: string) => {
    await runScript(`rd /s/q ${targetDir}`);
}
const isExistDir = (targetDir: string): boolean => {
    return fs.existsSync(targetDir);
}
const downloadFile = async(url: string, targetDir: string) => {
    await download(url, targetDir);
}

export {
    read,
    createDir,
    deleteDir,
    isExistDir,
    downloadFile
}
```
```
//src/libs/lib.ts
const runScript = require('runscript');
import { log } from '../utils/log';

//sourceDir：简书文章目录
export const getAllMarkdownFiles = async (sourceDir: string) => {
    //ls **/*.md 查询二级目录下的所有.md后缀的文件
    //stdio: pipe 在父进程和子进程之间建立管道
    const { stdout } = await runScript('ls **/*.md', {
        cwd: sourceDir,
        stdio: 'pipe'
    });
    const files: string[] = stdout.toString().split('\n');

    //去掉ls命令产生的尾部空行
    files.pop();
    log('获取所有的简书文章列表；');
    return files;
}

//获取图片url的markdown写法![](https://....)
export const getMarkdownImageUrls = (fileContent: string) => {
    const urlRegExp = /\!\[.*\]\((https?:\/\/.+?)\)/g;

    const imageUrls: string[] = [];
    while(true) {
        const match = urlRegExp.exec(fileContent);
        if(match === null) {
            break;
        }
        
        const [, url] = match;
        imageUrls.push(url);
    }
    return imageUrls;
}
```
主入口函数：
```
//src/index.ts
import path from 'path';
import { log } from './utils/log';
import { read, createDir, deleteDir, isExistDir, downloadFile } from './utils/fs';
import { getAllMarkdownFiles, getMarkdownImageUrls } from './libs/lib';

//入口函数
const main = async () => {
    //平台判断
    const { platform } = process;
    const isWindows: boolean = platform === 'win32';

    //获取命令行参数
    const [, , sourceDir, targetDir] = process.argv;

    //获取markdown文件列表
    const files: string[] = await getAllMarkdownFiles(sourceDir);

    //下载文件列表中每个文章的图片
    for(const file of files){
        // file 是相对路径 例如："2017-2018/前端模块化总结.md"

        // 兼容 windows 系统路径规则
        let platFile: string = isWindows ? `${file.split('.md')[0].split('/').join('\\')}.md` : file;
        const filepath: string = platFile.split('.md')[0];

        //读取文件内容
        const filecontent = read(path.join(sourceDir, platFile), { encoding: 'utf8'});
        
        //根据 md 文件名，创建目标文件夹，如果目标文件夹存在，则删除重建
        const newTargetDir: string = path.join(targetDir, filepath);
        if(isExistDir(newTargetDir)){
            await deleteDir(newTargetDir);
        }
        await createDir(newTargetDir);

        //找出图片，下载图片到目标目录
        const urlList: string[] = getMarkdownImageUrls(filecontent);
        for(const url of urlList){
            await downloadFile(url, newTargetDir);
        }
    }

    log('所有文章中的图片已下载成功！');
}
main();
```
提交代码到 master 分支，然后新建 node-cli-refactory 分支用来保存重构历史。
```
git checkout -b node-cli-refactory
git pull origin master
git push
```
最后这个下载图片的小工具就做好。

总结：在写代码的过程中，一定要分析什么是通用工具类，什么是独有的业务逻辑类，该模块化的模块化，目的只有一个就是：**主流程要清晰**。

**项目地址：https://github.com/mxcz213/download-jianshu-images**

参考：
https://www.npmjs.com/package/runscript
https://www.npmjs.com/package/download
https://www.npmjs.cn/files/package.json/











