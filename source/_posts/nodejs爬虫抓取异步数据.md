---
title: nodejs爬虫抓取异步数据
copyright: true
date: 2017-07-21 21:56:20
tags:
- node
- 爬虫
- promise
categories:
- 爬虫 
---

我们在抓取网页的时候，如果目标站点是服务端渲染好的页面，那么我们在抓取网页内容就很方便，只需要分析对应的dom节点内容就可以获取我们需要的数据。
但是，如果数据是前端异步请求获取，再由js构造的节点，那么我们直接分析抓取到的网页是没有用的，即使我们在浏览器的开发者工具中能够看到对应的节点，
我们也无法获取到这部分异步刷新的节点，因为这是js构造的，而我们通过request请求到的是还没有js进行处理过的页面，分辨是否异步刷新的方法很简单，
右键网页查看源代码，如果在源代码里面有的，就是可以直接分析得到，如果没有，则需要我们去分析后台接口。

<!--more-->

# 案例介绍
作为ACM会长，由于暑期集训，需要记录集训队员的日常做题情况，而手动查看rank榜极其不便，因此通过node+request编写了一个爬取rank榜过滤出本校成员的rank，
并导出到excel记录，而我们训练的 [virtual judge](https://vjudge.net/) 上的比赛列表和rank榜的数据都是通过ajax请求后台接口获取的，所以这里就记录
一下编写爬虫的过程


# 获取比赛列表

## 分析网页元素和url的关系
<https://vjudge.net/contest> 这里提供了比赛列表，但是我们需要hrbustacm哈理工发布的比赛，所以在标题搜索栏中输入2017,作者搜索栏中输入hrbustacm，发现url地址变成：
<https://vjudge.net/contest#category=mine&running=0&title=&owner=hrbustacm>，所以我们抓取比赛列表的url就是这个。
{% asset_img 1_1.png %}

## 注意User-Agent
这里如果过直接发送get请求,vj网站会发现我们是爬虫，从而拒绝返回数据
我们通过复制浏览器中的请求这张网页的http请求头，一并发送到服务器，就能够获取到这张页面了

## 辨别是哪个请求
注意这里是第一个请求，因为这个请求才是请求的这个页面本身，其他的请求是获取外联css，js，图片，获取我们等等要说的ajax异步请求
{% asset_img 1_2.png %}

## 提取需要发送的headers
点开来第一个请求,关注我们的request headers,这是我们要一同发送的请求头
{% asset_img 1_3.png %}

## 获取网页的代码
```javascript
request.get({
            url:'https://vjudge.net/contest#category=mine&running=0&title=&owner=hrbustacm',
            rejectUnauthorized: false,
            headers:{
                "Accept":"application/json, text/javascript, */*; q=0.01",
                "Accept-Encoding":"gzip, deflate, br",
                "Accept-Language":"zh-CN,zh;q=0.8",
                "Connection":"keep-alive",
                "Content-Type":"application/x-www-form-urlencoded; charset=UTF-8",
                "Host":"vjudge.net",
                "Referer":"https://vjudge.net/contest/",
                "User-Agent":"Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36",
            }
        },function(err,res,body){
            console.log(body);
        });
```
## 网页无数据，寻找xhr请求接口
获取到页面之后我们查看输出，并未发现比赛列表，因为他是通过ajax请求的数据，所以上面的代码实际上是获取了一张没有数据的网页

## xhr请求分析
我们现在的目标是获取比赛列表，查看Network下的所有xhr请求，也就是ajax异步请求
{% asset_img 1_4.png %}

通过观察两个请求返回的数据，我们发现第二个data请求返回的数据里面有我们需要的比赛信息
{% asset_img 1_5.png %}

很兴奋有没有，再次观察请求header里面的url，这个就是我们要请求的地址
{% asset_img 1_6.png %}

## 获取比赛列表代码：
```javascript
function getContestList(){
    return new Promise(function(resolve,reject){
        request.post({
            url:'https://vjudge.net/contest/data',
            rejectUnauthorized: false,
            gzip: true,
            headers:{
                "Accept":"application/json, text/javascript, */*; q=0.01",
                "Accept-Encoding":"gzip, deflate, br",
                "Accept-Language":"zh-CN,zh;q=0.8",
                "Connection":"keep-alive",
                "Content-Type":"application/x-www-form-urlencoded; charset=UTF-8",
                "Host":"vjudge.net",
                "Referer":"https://vjudge.net/contest/",
                "User-Agent":"Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36",
            },
            form: queryString.parse("draw=1&columns%5B0%5D%5Bdata%5D=function&columns%5B0%5D%5Bname%5D=&columns%5B0%5D%5Bsearchable%5D=true&columns%5B0%5D%5Borderable%5D=true&columns%5B0%5D%5Bsearch%5D%5Bvalue%5D=&columns%5B0%5D%5Bsearch%5D%5Bregex%5D=false&columns%5B1%5D%5Bdata%5D=function&columns%5B1%5D%5Bname%5D=&columns%5B1%5D%5Bsearchable%5D=true&columns%5B1%5D%5Borderable%5D=false&columns%5B1%5D%5Bsearch%5D%5Bvalue%5D=&columns%5B1%5D%5Bsearch%5D%5Bregex%5D=false&columns%5B2%5D%5Bdata%5D=function&columns%5B2%5D%5Bname%5D=&columns%5B2%5D%5Bsearchable%5D=true&columns%5B2%5D%5Borderable%5D=true&columns%5B2%5D%5Bsearch%5D%5Bvalue%5D=&columns%5B2%5D%5Bsearch%5D%5Bregex%5D=false&columns%5B3%5D%5Bdata%5D=function&columns%5B3%5D%5Bname%5D=&columns%5B3%5D%5Bsearchable%5D=true&columns%5B3%5D%5Borderable%5D=true&columns%5B3%5D%5Bsearch%5D%5Bvalue%5D=&columns%5B3%5D%5Bsearch%5D%5Bregex%5D=false&columns%5B4%5D%5Bdata%5D=function&columns%5B4%5D%5Bname%5D=&columns%5B4%5D%5Bsearchable%5D=true&columns%5B4%5D%5Borderable%5D=true&columns%5B4%5D%5Bsearch%5D%5Bvalue%5D=&columns%5B4%5D%5Bsearch%5D%5Bregex%5D=false&columns%5B5%5D%5Bdata%5D=function&columns%5B5%5D%5Bname%5D=&columns%5B5%5D%5Bsearchable%5D=true&columns%5B5%5D%5Borderable%5D=false&columns%5B5%5D%5Bsearch%5D%5Bvalue%5D=&columns%5B5%5D%5Bsearch%5D%5Bregex%5D=false&columns%5B6%5D%5Bdata%5D=function&columns%5B6%5D%5Bname%5D=&columns%5B6%5D%5Bsearchable%5D=true&columns%5B6%5D%5Borderable%5D=false&columns%5B6%5D%5Bsearch%5D%5Bvalue%5D=&columns%5B6%5D%5Bsearch%5D%5Bregex%5D=false&order%5B0%5D%5Bcolumn%5D=0&order%5B0%5D%5Bdir%5D=desc&start=0&length=20&search%5Bvalue%5D=&search%5Bregex%5D=false&category=mine&running=0&title=2017&owner=hrbustacm")
        }, 
        function(err,res,body){
            // console.log(err);
            // console.log(res.statusCode);
            // console.log(JSON.parse(body).data);
            if(err){
                reject(err);
            }
            var contestList =  JSON.parse(body).data;
            resolve(contestList);
        });
    });
}
```

## 提取headers，注意gzip
这里headers就是request headers 里的内容，其中cookie等，因为不需要登录什么的就不用加
注意response header里面返回的gzip之后的数据，我们需要解压才能正确看到，只需要在请求的时候加上gzip:true就行了
至于这个form里面一大串奇怪的东西，就是
{% asset_img 1_7.png %}

## 请求参数反序列化
需要反序列化的参数在header下的formdata里，这里也不知道他的请求参数真的有那么复杂还是什么，需要他这一大堆参数才能请求成功，但是这是序列化后的结果，需要用querystring反序列化过

## 获得比赛id列表，开始获取单场比赛rank榜
至此我们已经获取了一页比赛的比赛id，比赛名，这里比赛id有什么用呢，通过观察，我们发现点击一场比赛，他的url就变成https://vjudge.net/contest/168971，最后是比赛id
所以我们能够访问每一场比赛的具体信息啦，而他的rank榜的url就是https://vjudge.net/contest/168971#rank

# 获取比赛榜单

## 榜单数据也是异步刷新
费了这么大劲，我们终于获取到了比赛id列表，接下来就是获取每个比赛的rank榜单了，通过查看源码和分析榜单所在dom节点，我们发现，榜单数据也是异步刷新出来的，
可见，一个网站要么是后台渲染好数据返回前台，要么是前台异步请求数据再构造dom节点到相应位置。

## 分析xhr请求
{% asset_img 2_1.png %}
通过分析返回数据，我们发现第二个和第三个请求返回了差不多格式的数据，但是第三个接口url地址是我们已经有的比赛id，而第二个请求的地址带了hash码，所以我们优先考虑第三个，
如果分析不出来再看第二个

##  participants分析
通过观察participants和他的英文原意，我们不难发现，这是所有的参赛者信息，通过用户id排序，不是根据rank排名来的
{% asset_img 2_2.png %}

##  submissions分析
通过观察submissions和他的英文原意，发现他是一个提交数组，他的第一个参数是用户id，第二个参数和第三个参数比较难分析，需要找一个具体用户分析，最后我分析出来第二个参数是
题目编号0代表A，1代表B...第三个参数永远是0或者1,所以应该代表是否争取，第四个参数代表提交距离开始的毫秒数
{% asset_img 2_3.png %}

## 没有现成榜单数据，自己计算得出
但是分析了这个请求之后，发现并没有我们的榜单排名数组，于是回过去分析刚刚那个剩下的请求，发现这两个请求的返回数据是一样的！！！！！
于是就陷入了瓶颈，经过一番思考，我们知道这肯定是异步刷出来的数据，但是返回的数据又没有我们需要的rank，只有比赛者信息和提交信息，
但是，我们发现，通过这两个信息，加上已知的排名规则，我们完全可以自己计算出比赛排名，首先是根据通过的题目数量，其次是做题速度，
事实上，这个网站也是通过前端的js计算，计算出这个排名，因为他最后展现的不仅仅是比赛有效排名，还有比赛后的提交排名和各种筛选条件，所以如果他这个接口直接返回比赛排名还真有点不妥，
无奈的是这个计算的js并不好找，因为他可能被压缩过，代码极其难看懂，也有可能是外联的js，而在这么多外联的js中，他们也是压缩过的，根本一个都看不懂，
所以最后还是自己写一个模拟算法计算出比赛的rank排名，毕竟，怎么说也是一个ACMer去爬ACM比赛排行，给定数据下算出排名还是小菜一碟的(臭不要脸)

## 请求比赛者信息和比赛提交信息的代码：
```javascript
//获取单个比赛的rank数据，参与者和提交记录
function getRankDate(id){
    return new Promise(function(resolve,reject){
        request.post({
            url:'https://vjudge.net/contest/rank/single/' + id,
            rejectUnauthorized: false,
            gzip:true,
            headers:{
                "Accept":"application/json, text/javascript, */*; q=0.01",
                "Accept-Encoding":"gzip, deflate, br",
                "Accept-Language":"zh-CN,zh;q=0.8",
                "Connection":"keep-alive",
                "Host":"vjudge.net",
                "Referer":"https://vjudge.net/contest/" + id,
                "User-Agent":"Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36",
            }
        }, 
        function(err,res,body){
            if(err){
                reject(err);
            }
            // console.log(JSON.parse(body));
            resolve(JSON.parse(body));

        });
    });
}
```

## 根据比赛者信息和提交记录计算比赛排名的代码，具体规则就不说了，就是一个计算排名的逻辑，重要我们可以根据已有的信息获取我们想要的信息这个思路
```javascript
//根据参与者和提交记录计算排名
function calculateRank(parts,submit,length){
    // var parts = {
    //     327:['a','aaa'],
    //     515:['b','bbb'],
    //     155:['c','ccc']
    // }
    // var submit = [
    //     [327,0,0,12],
    //     [515,0,1,12],
    //     [327,0,1,20],
    //     [327,0,1,33],
    //     [327,1,1,33],
    //     [155,0,1,66],
    //     [155,1,1,81981],
    //     [155,2,1,198190]
    // ];
    var map = {};

    for(let i = 0;i < submit.length;i++){
        var id = submit[i][0];
        // console.log(id);
        var que = submit[i][1];
        var is_AC = submit[i][2];
        var time = submit[i][3];
        if(time > length){
            continue;
        }
        try{
        if(map[id] == null){
            map[id] = {};
            map[id].isSolve = {};
            map[id].totalTime = {};
            map[id].time = 0;
            if(is_AC == 1){
                map[id].solveCnt = 1;
                map[id].totalTime[que] = time;
                map[id].isSolve[que] = 1;
                map[id].time += map[id].totalTime[que];
                // console.log(map[id].time);
            }else{
                map[id].solveCnt = 0;
                map[id].isSolve[que] = 0;
                map[id].totalTime[que] = 1200;
            }
        }else{
            if(is_AC == 1){
                if(map[id].isSolve[que] == null){
                    map[id].totalTime[que] = time;
                }else if(map[id].isSolve[que] == 0){
                    map[id].totalTime[que] += time;
                }
                map[id].solveCnt++;
                map[id].isSolve[que] = 1;
                map[id].time += map[id].totalTime[que];
            }else{
                if(map[id].isSolve[que] == null){
                    map[id].totalTime[que] = 1200;
                }else if(map[id].isSolve[que] == 0){
                    map[id].totalTime[que] += 1200;
                }
                map[id].isSolve[que] = 0;
            }
        }}catch(e){
            console.log(e);
            throw(e);
        }
    }

    // console.log(map);
    var arr = [];
    for(let i in map){
        map[i].id = i;
        // console.log(map[i]);
        arr.push(map[i]);
    }
    // console.log(arr);

    arr.sort(function(a,b){
        if(a.solveCnt > b.solveCnt){
            return -1;
        }else if(a.solveCnt == b.solveCnt){
            if(a.time < b.time){
                return -1
            }else{
                return 1;
            }
        }else{
            return 1;
        }
    });

    // console.log(arr);
    for(let i = 0;i < arr.length;i++){
        // console.log(parts[arr[i].id]);
        arr[i].nickName = parts[arr[i].id][0];
        arr[i].name = parts[arr[i].id][1]
        // console.log(nickName+'('+name+')');
        // console.log(arr[i].solveCnt);
    }
    return arr;
    // [
    //     {
    //         id:115651,
    //         solveCnt:6,
    //         time:15118,
    //         nickname:'a',
    //         name:'aaa'
    //     }
    // ]
}
```

# 导出rank记录到excel

## 既然已经有了每场比赛的rank榜的数据，我们就可以导出到excel便于观察，这里我通过exceljs这个模块导出，具体用法参见[exceljs](https://github.com/guyonroche/exceljs)的github
```javascript
function exportToExcel(contestRanks){
    var Excel = require('exceljs');
    // construct a streaming XLSX workbook writer with styles and shared strings
    var options = {
        filename: './vj训练记录.xlsx',
        useStyles: true,
        useSharedStrings: true
    };
    var workbook = new Excel.stream.xlsx.WorkbookWriter(options);

    workbook.creator = 'tosim';
    workbook.lastModifiedBy = 'tosim';
    workbook.created = new Date(2017, 7, 20);
    workbook.modified = new Date();

    for(let i in contestRanks){
        var worksheet = workbook.addWorksheet(i);
        worksheet.columns = [
            { header: 'Rank', key: 'rank', width: 10 },
            { header: 'Team', key: 'team', width: 50 },
            { header: 'Score', key: 'score', width: 10, outlineLevel: 1 },
            { header: 'Penalty', key: 'penalty', width: 10, outlineLevel: 1 }
        ];
        for(let j = 0;j < contestRanks[i].length;j++){
            var team = contestRanks[i][j].nickName+'('+contestRanks[i][j].name+')';
            if(/(zust)|(浙科院)|(科院)/.test(team)){//这里过滤出我们学校的用户，因为我们的队员都是带这三个其中一个的
                // console.log(team);
                // console.log(contestRanks[i][j].time/60);
                worksheet.addRow({rank: j+1, team: team, score:contestRanks[i][j].solveCnt,penalty:parseInt(contestRanks[i][j].time/60)});
            }
        }
        // worksheet.addRow({rank: 1, team: '营业员', score:5,penalty:123});
        worksheet.commit();
    }

    workbook.commit();
}
```

## 正则表达式推荐大家好好学习一下，威力无穷啊，这里有点大才小用的感觉- -，过滤出了所有用户里面我们集训队的成员
```javascript
for(let j = 0;j < contestRanks[i].length;j++){
    var team = contestRanks[i][j].nickName+'('+contestRanks[i][j].name+')';
    if(/(zust)|(浙科院)|(科院)/.test(team)){//这里过滤出我们学校的用户，因为我们的队员都是带这三个其中一个的
        // console.log(team);
        // console.log(contestRanks[i][j].time/60);
        worksheet.addRow({rank: j+1, team: team, score:contestRanks[i][j].solveCnt,penalty:parseInt(contestRanks[i][j].time/60)});
    }
}
```


# 组合Promise的运行逻辑
 
## 上面三个是经过抽象的函数都返回的Promise，Promise威力无穷，对于node的异步编程好处多多，建议大家学习一下Promise，这里推荐大家[阮一峰的教程](http://es6.ruanyifeng.com/#docs/promise)，启蒙老师

## 组合代码
```javascript
getContestList()//获取比赛id
    .then(function(contestList){
        // console.log(contestList);
        var validateList = [];
        var now = new Date().getTime();
        contestList.forEach(function(item){
            // console.log(item);
            // console.log("end");
            if(/^训练赛20170[78]\d\d$/.test(item[1])){
                if(now < item[3]){//比赛还没结束或还没开始
                    return;
                }
                validateList.push({
                    id:item[0],
                    name:item[1]
                });    
            }
        });
        // console.log(validateList);
        return new Promise(function(resolve,reject){
            var promiseList = [];
            for(let i = 0;i < validateList.length;i++){
                promiseList.push(getRankDate(validateList[i].id));
            }
            Promise.all(promiseList)
                .then(function(results){
                    var contestRanks = {};//比赛名称:比赛rank
                    for(let i = 0;i < results.length;i++){
                        // console.log(parseInt(results[i].length/1000));
                        contestRanks[validateList[i].name] = calculateRank(results[i].participants,results[i].submissions,parseInt(results[i].length/1000));
                    }
                    resolve(contestRanks);
                })
                .catch(function(err){
                    reject(err);
                });
                
        });
    })
    .then(function(contestRanks){
        for(let i in contestRanks){
            console.log(i);
        }
        // console.log(contestRanks['训练赛20170720']);
        exportToExcel(contestRanks);
        console.log("done");
    })
```

# 个人建议

1. 编写爬虫其实并没有你想象的那么难，无非就是发送请求，处理返回的结果，发送请求我使用request模块，建议学习一下<https://github.com/request/request>

2. 写爬虫最主要的是分析，相信很多人看到我这么多字就不想读下去了，急于看代码，看了代码又不懂就丧失了信心，其实只要分析出来需要请求的地址，配合上面发送请求的api，得到了我们想要的数据，就非常简单

3. 这里我没有讲到如何分析网页中的数据，即使用[cheerio模块](https://github.com/cheeriojs/cheerio)分析服务端渲染的网页，这部分比较简单，主要就是分析需要的数据在哪个dom节点里再通过类似jquery的方法提取出来，如果大家有需要，可以在下方评论留言。

4. 希望没有看完我文字分析的同学好好看看分析过程，写爬虫主要是分析，写爬虫主要是分析，写爬虫主要是分析！

5. 希望大家多多练习ACM的题目，嘻嘻嘻

6. 本项目的github地址：<https://github.com/tosim/craw_vj>