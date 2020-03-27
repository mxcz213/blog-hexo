---
title: （Node-mongo系列）Puppeteer爬取网页数据并写入文件（三）
---
#####官方介绍
在浏览器中手动执行的绝大多数操作都可以使用` Puppeteer `来完成！ 下面是一些示例：
*   生成页面 PDF。
*   抓取 SPA（单页应用）并生成预渲染内容（即“SSR”（服务器端渲染））。
*   自动提交表单，进行 UI 测试，键盘输入等。
*   创建一个时时更新的自动化测试环境。 使用最新的 JavaScript 和浏览器功能直接在最新版本的Chrome中执行测试。
*   捕获网站的 [timeline trace](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference)，用来帮助分析性能问题。
*   测试浏览器扩展。

#####实践
通过`Puppeteer`的`api`来实例化一个`browser`，然后新建一个`page`，打开一个`url`地址，渲染完成之后，模拟在`console`面板操作`dom`，来获取想要的数据。
新建一个`index.js`文件，拿到数据之后讲数据写入文件`curse-list.json`文件中，代码如下：
```
//index.js
//用puppeteer来模拟浏览器操作拿到前端免费课程的列表，第一页
const puppeteer = require('puppeteer');
const fs = require('fs');
const imoocUrl = 'https://www.imooc.com/course/list?c=fe';
;(async () => {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    console.log('start open url:',imoocUrl);
    await page.goto(imoocUrl);
    
    //操作数据
    console.log('operate dom by console');
    const result = await page.evaluate(() => {
        let $ = window.$;
        let data = [];
        let courseList = $('.moco-course-list').find('.course-card-container');
        if(courseList.length > 1){
            courseList.each((index,item) => {
                let item_a = $(item).find('a');
                let tags = [];
                let labels = item_a.find('.course-label label');
                if(labels.length > 0){
                    labels.each((index,item) => {
                        tags.push($(item).text());
                    })
                }
                let content = item_a.find('.course-card-content');
                let title = content.find('.course-card-name').text();
                let card = content.find('.course-card-info span');
                let level = $(card[0]).text();
                let desc = content.find('.course-card-desc').text();
                data.push({
                    title,
                    tags: tags.join(','),
                    level,
                    desc
                });
            })
        }
        return data;
    });

    await browser.close();
    console.log('打印数据','\n',result);

    //将数据写入到文件中，通过fs模块
    let apiData = {
        data: result,
        code: 0,
        message: 'success'
    }
    fs.writeFile('course-list.json',JSON.stringify(apiData,null,'\t'));
})();
```
运行命令：
`node index.js`
然后打开json文件，数据已经被写入到文件中。
![](https://upload-images.jianshu.io/upload_images/5541401-19b40e33ba687c03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####注意：fs.writeFile()写入的数据类型必须是这几种
*   `data` [<string>](http://nodejs.cn/s/9Tw2bK) | [<Buffer>](http://nodejs.cn/s/6x1hD3) | [<TypedArray>](http://nodejs.cn/s/oh3CkV) | [<DataView>](http://nodejs.cn/s/yCdVkD)
所以数据用`JSON.stringify`包一层，然后`\t`格式化数据`JSON.stringify(apiData,null,'\t')`。

本来不加`\t`直接写入的是没有格式化的数据，又看了`JSON.stringify`的参数`JSON.stringify(value[, replacer [, space]])`；
`space`用于美化输出。

#####结语
最后一个数据的爬取就完成了，也可以封装成一个函数，通过传入一个`url`，一个`dom操作的回调函数`，来获取想要的各种各样的数据，写入到文件中，在需要用到数据的时候，通过读取文件来mock数据，这样前端就可以专注于前端了。


参考：
https://github.com/GoogleChrome/puppeteer
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify

