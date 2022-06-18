# 微前端

本质跟jq差不多, 插过来

# 原件化

```ts
ng new --create-application=false  mfdemo1 && cd mfdemo1
// 新建两个项目
ng g application mfdemo1 --routing --style=scss
ng g application mf-element1 --routing --style=scss
// 自定义元素
ng add @angular/elements --project=mfdemo1
// 扩展 Angular CLI 的默认构建行为
ng add ngx-build-plus --project=mfddemo1
或者把对应的mf-element1的angular改下
          "builder": "ngx-build-plus:build",
//          "builder": "@angular-devkit/build-angular:browser",
```

我们在组件里面`mf-element1`项目中`app.module.ts`去掉

```ts
import {createCustomElement} from "@angular/elements";
import {Injector, NgModule} from '@angular/core';
@Component({
 // selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})

@NgModule({
  providers: [],
//  bootstrap: [AppComponent]
})
export class AppModule {
  constructor(private injector: Injector) {
  }

  ngDoBootstrap(): void {
    const customElement = createCustomElement(AppComponent, {injector: this.injector})
    // 原生自定义元素的方法
    customElements.define('mf-element1', customElement)
  }
}
```



# 关于Polyfills设定

> 如果在非`angular`的项目使用, 引入`zone.js`
>
> * `node_modules/zone.js/dist/zone.min.js`
>
> 如果你的项目不支持自定义元素`Polyfills(IE11)`
>
> * 执行`ng add @angular/elements` 自定义会引入`document-register-element`
> * `node_modules/document-register-element/build/document-register-element.js`

# 打包

`选项--prod是--configuration="production"的别名。但是--prod命令行参数在 Angular 12 中已被弃用。`

```json
// 打包的时候取消哈希的设置
   "build:mfdemo1": "ng build --project=mfdemo1 --configuration production --output-hashing=none",
   "build:mf-element1": "ng build --project=mf-element1 --configuration production --output-hashing=none --single-bundle",
// 把主要的两个文件拷到主要文件里面
    "build:mf-element1:deploy": "cp dist/mf-element1/main.js dist/mfdemo1/mf-element1.js & cp dist/mf-element1/polyfills.js dist/mfdemo1/mf-polyfills.js",
// yarn add lite-server  -D
 "serve:mfdemo1": "cd dist/mfdemo1 && lite-server",
```

> 介绍`https://github.com/manfredsteyer/ngx-build-plus`
>
> ```ts
> ng build --single-bundle
> ：将所有可从主入口点访问的内容放入一个包中。Polyfill、脚本和样式保留在自己的包中，因为消费应用程序可能有自己的版本。
> ```

# 打包后的差异

```js
main.js  主要程式
mfmemo2 会把 runtime.js合并到 main.js
```

我们在主文件的`index.html`里面

# 执行

```ts
执行 start serve:mfdemo1
```

看看index.html

```ts

<!DOCTYPE html><html lang="en"><head>
  <meta charset="utf-8">
  <title>Mfdemo1</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
<!--<link rel="stylesheet" href="styles.css"></head>-->
<body>
<!--  <app-root></app-root>-->
<!--<script src="runtime.js" defer></script><script src="polyfills.js" defer></script><script src="main.js" defer></script>-->
<h1>ok</h1>
<mf-element1></mf-element1>
<script src="mf-polyfills.js"></script>
<script src="mf-element1.js"></script>
</body></html>
```

我们知道把`mf-polyfills.js`改成`zone.min.js` 也是可以的, 也就是说我们一个主文件不需要一个就行了

# Angular 载入Angular Element

```ts
yarn add @angular-extensions/elements
```

导入到`AppModule`

```ts
 import {LazyElementsModule} from "@angular-extensions/elements";
import {CUSTOM_ELEMENTS_SCHEMA, NgModule} from '@angular/core';

imports: [
    BrowserModule,
    AppRoutingModule,
 +    LazyElementsModule
  ],
 schemas:[CUSTOM_ELEMENTS_SCHEMA],
```

在`app.component.ts`, 自定义载入

```ts
<elementUrl *axLazyElement="elementUrl"></elementUrl>

  elementUrl = "/mf-element1.js"
```

然后执行

```ts
 yarn build:mfdemo1  打包主程序
 yarn build:mf-element1:deploy  把我们的包引过来
 yarn serve:mfdemo1  执行下打包后的index.html
 
```

