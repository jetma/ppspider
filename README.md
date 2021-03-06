

<!-- toc -->

- [Translations](#translations)
- [Puppeteer Doc](#puppeteer-doc)
  * [Questions about puppeteer](#questions-about-puppeteer)
- [Quick Start](#quick-start)
  * [Install NodeJs](#install-nodejs)
  * [Install TypeScript](#install-typescript)
  * [Prepare the development environment](#prepare-the-development-environment)
  * [Download And Run ppspider_example](#download-and-run-ppspider_example)
    + [Clone ppspider_example with IDEA](#clone-ppspider_example-with-idea)
    + [Install npm dependencies](#install-npm-dependencies)
    + [Run tsc](#run-tsc)
    + [Startup ppspider App](#startup-ppspider-app)
- [ppspider System Introduction](#ppspider-system-introduction)
  * [Decorator](#decorator)
    + [@Launcher](#launcher)
    + [@OnStart](#onstart)
    + [@OnTime](#ontime)
    + [@AddToQueue @FromQueue](#addtoqueue-fromqueue)
    + [@JobOverride](#joboverride)
    + [@Serialize Serializable @Transient](#serialize-serializable-transient)
  * [PuppeteerUtil](#puppeteerutil)
    + [PuppeteerUtil.defaultViewPort](#puppeteerutildefaultviewport)
    + [PuppeteerUtil.addJquery](#puppeteerutiladdjquery)
    + [PuppeteerUtil.jsonp](#puppeteerutiljsonp)
    + [PuppeteerUtil.setImgLoad](#puppeteerutilsetimgload)
    + [PuppeteerUtil.onResponse](#puppeteerutilonresponse)
    + [PuppeteerUtil.onceResponse](#puppeteerutilonceresponse)
    + [PuppeteerUtil.downloadImg](#puppeteerutildownloadimg)
    + [PuppeteerUtil.links](#puppeteerutillinks)
    + [PuppeteerUtil.count](#puppeteerutilcount)
    + [PuppeteerUtil.specifyIdByJquery](#puppeteerutilspecifyidbyjquery)
    + [PuppeteerUtil.scrollToBottom](#puppeteerutilscrolltobottom)
    + [PuppeteerUtil example](#puppeteerutil-example)
  * [logger](#logger)
- [Debug](#debug)
- [WebUI](#webui)

<!-- tocstop -->

# Translations
[中文文档](https://github.com/xiyuan-fengyu/ppspider/blob/master/README.cn.md)  

# Puppeteer Doc
https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md   
## Questions about puppeteer
1. When installing puppeteer, it will download chromium by default. If download is fail, you 
    can set temp environment variable by terminal before executing 'npm install'  
    ```
    # win
    set PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=1
    # unix
    export PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=1
    ```
    Then you can download chromium at this page (https://download-chromium.appspot.com/) or
    use the local chrome installed.  
    Finally, init the PuppeteerWorkerFactory with parameter named 'executablePath'.  
    ```
    @Launcher({
        workplace: __dirname + "/workplace",
        tasks: [
            TestTask
        ],
        workerFactorys: [
            new PuppeteerWorkerFactory({
                headless: false,
                executablePath: "YOU_CHROMIUM_OR_CHROME_EXECUTE_PATH"
            })
        ]
    })
    class App {}
    ```


# Quick Start
## Install NodeJs
[nodejs official downloading address](http://nodejs.cn/download/)

## Install TypeScript
```
npm install -g typescript
```

## Prepare the development environment
Recommended IDEA(Ultimate version) + NodeJs Plugin  
![Ultimate 版本截图](https://s1.ax1x.com/2018/06/13/CO6qRf.png)  
The NodeJs plugin's version should match with the IDEA's version    
![IDEA 版本](https://s1.ax1x.com/2018/06/13/COcdYt.png)  
![nodejs 插件版本](https://s1.ax1x.com/2018/06/13/COcBSf.png)

[IDEA download](https://www.jetbrains.com/idea/download/)  
[NodeJs Plugin download](http://plugins.jetbrains.com/plugin/6098-nodejs)

## Download And Run ppspider_example
ppspider_example github address  
https://github.com/xiyuan-fengyu/ppspider_example  
### Clone ppspider_example with IDEA   
Warning: git is required and the executable file path of git should be set in IDEA  
![IDEA git 配置](https://s1.ax1x.com/2018/06/13/COcr6S.png)

![IDEA clone from git](https://s1.ax1x.com/2018/06/13/COc6mQ.png)  
![IDEA clone from git](https://s1.ax1x.com/2018/06/13/COccwj.png)

### Install npm dependencies  
Run 'npm install' in terminal  
Or  
ContextMenu on package.json -> Run 'npm install'  

All npm dependencies will be saved to node_modules folder in the project   

### Run tsc
Run 'tsc' in terminal  
Or  
ContextMenu on package.json -> Show npm Scripts -> Double click 'auto build'   
tsc is a TypeScript compiler which can auto compile the ts file to js file after any ts file change  


### Startup ppspider App
Run lib/quickstart/App.js    
Open http://localhost:9000 in the browser to check the ppspider's status  

# ppspider System Introduction
## Decorator
Declare like    
```
export function TheDecoratorName(args) { ... }
```
Usage  
```
@TheDecoratorName(args)
```
Decorator is similar of look with java Annotation, but Decorator is stronger.  Decorator can provide meta data by parameters and modify the target or descriptor to change behavior of class or method  
In ppspider, many abilities are provided by Decorator  

### @Launcher
```
export function Launcher(theAppInfo: AppInfo) { ... }
```
Launcher of a ppspider app  
Params type  
```
export type AppInfo = {
    // all cache files and the db file will be saved to the workplace folder. You can save the data files to this folder too. 
    workplace: string;
    
    // all task class    
    tasks: any[];
    
    // all workerFactory instances, only PuppeteerWorkerFactory is provided at present  
    workerFactorys: WorkerFactory<any>[];
    
    // the port for web ui，default 9000  
    webUiPort?: number | 9000;
}
```
You can get theAppInfo by a global variable: appInfo
```
import {Launcher, PuppeteerWorkerFactory} from "ppspider";
import {TestTask} from "./tasks/TestTask";

@Launcher({
    workplace: __dirname + "/workplace",
    tasks: [
        TestTask
    ],
    workerFactorys: [
        new PuppeteerWorkerFactory({
            headless: false
        })
    ]
})
class App {

}
```

### @OnStart
```
export function OnStart(config: OnStartConfig) { ... }
```
A sub task type executed once after app start, but you can execute it again by pressing the button in webUI. The button will be found after the sub task name in webUI's Queue panel.  

Params type  
```
export type OnStartConfig = {
    // urls to crawl
    urls: string | string[];  
    
    // a workerFactory class whose instance shold be declared in @Launcher parameter's property: workerFactorys. Only PuppeteerWorkerFactory is supported at present.
    workerFactory: WorkerFactoryClass;
    
    // config of max paralle num, can be a number or a object with cron key and number value    
    parallel?: ParallelConfig;
    
    // the execute interval between jobs, all paralle jobs share the same exeInterval  
    exeInterval?: number;
    
    // description of this sub task type
    description?: string;
}
```
[@OnStart example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/quickstart)
```
import {Job, OnStart, PuppeteerUtil, PuppeteerWorkerFactory} from "ppspider";
import {Page} from "puppeteer";

export class TestTask {

    @OnStart({
        urls: "http://www.baidu.com",
        workerFactory: PuppeteerWorkerFactory
    })
    async index(page: Page, job: Job) {
        await page.goto(job.url());
        const urls = await PuppeteerUtil.links(page, {
            "all": "http.*"
        });
        console.log(urls);
    }

}
```

### @OnTime
```
export function OnTime(config: OnTimeConfig) { ... }
```
A sub task type executed at special times resolved by cron expression  
Params type  
```
export type OnTimeConfig = {
    urls: string | string[];  
    
    // cron expression
    cron: string;
    
    workerFactory: WorkerFactoryClass;
    
    parallel?: ParallelConfig;
    
    exeInterval?: number;
    
    description?: string;
}
```
[@OnTime example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/ontime)
```
import {Job, PuppeteerWorkerFactory, OnTime, DateUtil} from "ppspider";
import {Page} from "puppeteer";

export class TestTask {

    @OnTime({
        urls: "http://www.baidu.com",
        cron: "*/5 * * * * *",
        workerFactory: PuppeteerWorkerFactory
    })
    async index(page: Page, job: Job) {
        console.log(DateUtil.toStr());
    }

}
```

### @AddToQueue @FromQueue
Those two should be used together, @AddToQueue will add the function's result to job queue, @FromQueue will fetch jobs from queue to execute     

@AddToQueue
```
export function AddToQueue(queueConfigs: AddToQueueConfig | AddToQueueConfig[]) { ... }
```
@AddToQueue accepts one or multi configs  
Config type：
```
export type AddToQueueConfig = {
    // queue name
    name: string;
    
    // queue provided: DefaultQueue(FIFO), DefaultPriorityQueue
    queueType?: QueueClass;
    
    // filter provided: NoFilter(no check), BloonFilter(check by job's key)
    filterType?: FilterClass;
}
```
You can use @AddToQueue to add jobs to a same queue at multi places, the queue type is fixed at the first place, but you can use different filterType at each place.  

The method Decorated by @AddToQueue shuold return a AddToQueueData like.
```
export type CanCastToJob = string | string[] | Job | Job[];

export type AddToQueueData = Promise<CanCastToJob | {
    [queueName: string]: CanCastToJob
}>
```
If @AddToQueue has multi configs, the return data must like 
```
Promise<{
    [queueName: string]: CanCastToJob
}>
```
PuppeteerUtil.links is a convenient method to get all expected urls, and the return data is just AddToQueueData like.  


@FromQueue
```
export function FromQueue(config: FromQueueConfig) { ... }

export type FromQueueConfig = {
    name: string;
    
    workerFactory: WorkerFactoryClass;

    parallel?: ParallelConfig;
    
    exeInterval?: number;
    
    description?: string;
}
```
[@AddToQueue @FromQueue example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/queue)
```
import {AddToQueue, AddToQueueData, FromQueue, Job, OnStart, PuppeteerUtil, PuppeteerWorkerFactory} from "ppspider";
import {Page} from "puppeteer";

export class TestTask {

    @OnStart({
        urls: "http://www.baidu.com",
        workerFactory: PuppeteerWorkerFactory,
        parallel: {
            "0/20 * * * * ?": 1,
            "10/20 * * * * ?": 2
        }
    })
    @AddToQueue({
        name: "test"
    })
    async index(page: Page, job: Job): AddToQueueData {
        await page.goto(job.url());
        return PuppeteerUtil.links(page, {
            "test": "http.*"
        });
    }

    @FromQueue({
        name: "test",
        workerFactory: PuppeteerWorkerFactory,
        parallel: 1,
        exeInterval: 1000
    })
    async printUrl(page: Page, job: Job) {
        console.log(job.url());
    }

}
```

### @JobOverride
```
export function JobOverride(queueName: string) { ... }
```
Modify job info before inserted into the queue.  
You can set a JobOverride just once for a queue.  

A usage scenario is: when some urls with special suffix or parameters navigate to the same page,
you can modify the job key to some special and unique id taken from the url with a JobOverride. After 
that, jobs with duplicate keys will be filtered out.  

Actually, sub task type OnStart/OnTime is also managed by queue whose name just likes OnStart_ClassName_MethodName 
or OnTime_ClassName_MethodName, so you can set a JobOverride to it.   
[JobOverride example](https://github.com/xiyuan-fengyu/ppspider_example/blob/master/src/bilibili/tasks/BilibiliTask.ts)

### @Serialize Serializable @Transient
```
export function Serialize(config?: SerializeConfig) { ... }
export class Serializable { ... }
export function Transient() { ... }
```
@Serialize is used to mark a class, then the class info will keep during serializing and deserializing.
Otherwise, the class info will lose when serializing.   
a class which extends from Serializable supports customized serialize / deserialize， for example: [BitSet](https://github.com/xiyuan-fengyu/ppspider/blob/master/src/common/util/BitSet.ts)    
@Transient is used to mark a field which will be ignored when serializing and deserializing.
Warn: static field will not be serialized.   
These three are mainly used to save running status. You can use @Transient to ignore fields which are not 
related with running status, then the output file will be smaller in size 
and the possible error caused by deep nested object during deserializing will be avoid.  
[example](https://github.com/xiyuan-fengyu/ppspider/blob/master/src/test/SerializeTest.ts)



## PuppeteerUtil
### PuppeteerUtil.defaultViewPort
set page's view port to 1920 * 1080  

### PuppeteerUtil.addJquery
inject jquery to page, jquery will be invalid after page refresh or navigate, so you should call it after page load.  

### PuppeteerUtil.jsonp
parse json in jsonp string  

### PuppeteerUtil.setImgLoad
enable/disable image load 

### PuppeteerUtil.onResponse
listen response with special url, max listen num is supported

### PuppeteerUtil.onceResponse
listen response with special url just once

### PuppeteerUtil.downloadImg
download image with special css selector

### PuppeteerUtil.links
get all expected urls 

### PuppeteerUtil.count
count doms with special css selector

### PuppeteerUtil.specifyIdByJquery
Find dom nodes by jQuery(selector) and specify random id if not existed，
finally return the id array.    
A usage scenario is:   
In puppeteer, all methods of Page to find doms by css selector finally call 
document.querySelector / document.querySelectorAll which not support some 
special css selectors, howerver jQuery supports. Such as: "#someId a:eq(0)", "#someId a:contains('next')".   
So we can call specifyIdByJquery to specify id to the dom node and keep the special id returned, 
then call Page's method with the special id.  

### PuppeteerUtil.scrollToBottom
scroll to bottom  

### PuppeteerUtil example
[PuppeteerUtil example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/puppeteerUtil)
```
import {appInfo, Job, OnStart, PuppeteerUtil, PuppeteerWorkerFactory} from "ppspider";
import {Page} from "puppeteer";

export class TestTask {

    @OnStart({
        urls: "http://www.baidu.com",
        workerFactory: PuppeteerWorkerFactory
    })
    async index(page: Page, job: Job) {
        await PuppeteerUtil.defaultViewPort(page);

        await PuppeteerUtil.setImgLoad(page, false);

        const hisResWait = PuppeteerUtil.onceResponse(page, "https://www.baidu.com/his\\?.*", async response => {
            const resStr = await response.text();
            console.log(resStr);
            const resJson = PuppeteerUtil.jsonp(resStr);
            console.log(resJson);
        });

        await page.goto("http://www.baidu.com");
        await PuppeteerUtil.scrollToBottom(page, 5000, 100, 1000);

        await PuppeteerUtil.addJquery(page);
        console.log(await hisResWait);

        const downloadImgRes = await PuppeteerUtil.downloadImg(page, ".index-logo-src", appInfo.workplace + "/download/img");
        console.log(downloadImgRes);

        const href = await PuppeteerUtil.links(page, {
            "index": ["#result_logo", ".*"],
            "baidu": "^https?://[^/]*\\.baidu\\.",
            "other": (a: Element) => {
                const href = (a as any).href as string;
                if (href.startsWith("http")) return href;
            }
        });
        console.log(href);

        const count = await PuppeteerUtil.count(page, "#result_logo");
        console.log(count);

        const ids = await PuppeteerUtil.specifyIdByJquery(page, "a:visible:contains('登录')");
        if (ids) {
            console.log(ids);
            await page.tap("#" + ids[0]);
        }
    }

}
```

## logger
Use logger.debug, logger.info, logger.warn or logger.error to print log.  
Those functions are defined in src/common/util/logger.ts.  
The output logs contain extra info: timestamp, log level, source file position.  
```
// example
// set format, the default format is as follows
// logger.format = "yyyy-MM-dd HH:mm:ss.SSS [level] position message"
// set log level, must be "debug", "info", "warn" or "error"
// logger.level = "info";
logger.debugValid && logger.debug("test debug");
logger.info("test info");
logger.warn("test warn");
logger.error("test error");
```

# Debug
simple typescript/js code can be debugged in IDEA  

The inject js code can be debugged in Chromium. When building the PuppeteerWorkerFactory instance, set headless = false, devtools = true to open Chromium devtools panel.  
[Inject js debug example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/debug)   
```
import {Launcher, PuppeteerWorkerFactory} from "ppspider";
import {TestTask} from "./tasks/TestTask";

@Launcher({
    workplace: __dirname + "/workplace",
    tasks: [
        TestTask
    ],
    workerFactorys: [
        new PuppeteerWorkerFactory({
            headless: false,
            devtools: true
        })
    ]
})
class App {

}
```

<html>
    <p>
    Add <span style="color: #ff490d; font-weight: bold">debugger;</span> where you want  to debug
    </p>
</html>

```
import {Job, OnStart, PuppeteerWorkerFactory} from "ppspider";
import {Page} from "puppeteer";

export class TestTask {

    @OnStart({
        urls: "http://www.baidu.com",
        workerFactory: PuppeteerWorkerFactory
    })
    async index(page: Page, job: Job) {
        await page.goto(job.url());
        const title = await page.evaluate(() => {
           debugger;
           const title = document.title;
           console.log(title);
           return title;
        });
        console.log(title);
    }

}
```

# WebUI
open http://localhost:9000 in browser  

Queue panel: view and control app status  
![Queue Help](https://s1.ax1x.com/2018/06/13/COcgTs.png)  

Job panel: search jobs and view details  
![Job Help](https://s1.ax1x.com/2018/06/13/COcRkn.png)