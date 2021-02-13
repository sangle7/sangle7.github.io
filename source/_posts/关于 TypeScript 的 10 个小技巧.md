---
title: 关于 TypeScript 的 10 个小技巧
date: 2020-07-30 10:50:44
tags: 
  - 翻译
---

## 技巧 1. TypeScript 和 DOM

当您开始使用 TypeScript 时，您会发现在浏览器环境中使用它非常了解。 假设我想在页面上找到一个<input>元素：

```javascript
const textEl = document.querySelector('inpot');
console.log(textEl.value);
// 🛑 Property 'value' does not exist on type 'Element'.
```

Oops…… 抛出了一个错误，因为我把 'input' 打成了 'inpot'

它怎么知道的？ 答案在于 lib.dom.d.ts 文件，该文件是 TypeScript 库的一部分，并且基本上描述了浏览器中发生的所有事情（对象，函数，事件）。 该定义的一部分是在 querySelector 方法的输入中使用的接口，并将特定的字符串文字（例如'div', 'table'或'input'）映射到相应的 HTML 元素类型：

```typescript
interface HTMLElementTagNameMap {
  a: HTMLAnchorElement;
  abbr: HTMLElement;
  address: HTMLElement;
  applet: HTMLAppletElement;
  area: HTMLAreaElement;
  article: HTMLElement;
  /* ... */
  input: HTMLInputElement;
  /* ... */
}
```

<!-- more -->

这不是一个完美的解决方案，因为它仅适用于基本元素选择器，但总比没有好，对吧？

这种'智能'TypeScript 行为的另一个示例是在处理浏览器事件时：

```javascript
textEl.addEventListener('click', (e) => {
  console.log(e.clientX);
});
```

上面的示例中的.clientX 在任何给定事件上都不可用-仅在 MouseEvent 上可用。 然后 TypeScript 根据作为 addEventListener 方法中第一个参数的“click”文字确定事件的类型。

## 技巧 2. 期望泛型

因此，如果您使用其他任何东西而不是元素选择器：

```javascript
document.querySelector（'input.action'）
```

那么 `HTMLELementTagNameMap` 将不再有用，TypeScript 只会返回一个相当基本的 `Element` 类型。

与 `querySelector` 一样，函数通常可以返回各种不同的结构，而 TypeScript 不可能确定将是哪种结构。 在那种情况下，您可以非常期待，该函数也是通用的，并且可以使用方便的通用语法提供该返回类型：

```javascript
textEl = document.querySelector < HTMLInputElement > 'input.action';
console.log(textEl.value);
// 👍 'value' is available because we've instructed TS
// about the type the 'querySelector' function works with.
```

## 技巧 3. “我们真的找到了吗？”

该 document.querySelector（...）方法实际上并不总是返回一个对象，是吗？ 与选择器匹配的元素可能不在页面上-函数将返回 null 而不是对象。 因此，默认情况下，访问.value 属性可能不会保存所有内容。

默认情况下，类型检查器认为 null 和 undefined 可分配给任何类型。 您可以通过在 tsconfig.json 中添加严格的 null 检查来使其更加安全并限制这种行为：

```json
{
  "compilerOptions": {
    "strictNullChecks": true
  }
}
```

使用该设置后，如果您尝试访问可能为 `null` 的对象上的属性，TypeScript 将会报错，并且你将不得不确保该对象的存在，例如 通过用 `if（textEl）{...}` 条件包装该部分。

除了 `querySelector` 之外，另一个流行的例子是 `Array.find` 方法，其结果可能是不确定的。

您并非总能找到想要的东西:-)

## 技巧 4. “TS，我告诉你，在这里！”

正如我们已经确定的那样，通过严格的 null 检查，TypeScript 将更加怀疑我们的价值观。 另一方面，有时您仅从外部就知道将设置该值。 在这种特殊情况下，您可以使用“后缀表达式运算符”：

```javascript
const textEl = document.querySelector('input');
console.log(textEl!.value);
// 👍 with "!" we assure TypeScript
// that 'textEl' is not null/undefined
```

## Tip 5. 当迁移到 TS…

通常，当您具有要迁移到 TypeScript 的旧版代码库时，更大的麻烦之一就是使 id 遵守您的 TSLint 规则。 您可以做的是通过添加以下内容来编辑所有这些文件

```
// tslint:disable
```

在每个文件的第一行中，这样 TSLint 不会真正检查它们。 然后，仅当开发人员处理旧文件时，他才会删除此注释并仅修复该文件中的所有掉毛错误。 这样一来，我们就不会进行革命，而只会进行进化-代码库会逐渐但安全地得到改善。

至于将实际类型添加到旧的 JavaScript 代码中，实际上通常可以不这样做。 只有在您有一些令人讨厌的代码（例如， 为同一变量分配不同类型的值，您可能会遇到问题。 如果重构不是一个小问题，您可以使用这个方法解决问题：

```typescript
let mything = 2;
mything = 'hi';
// 🛑 Type '"hi"' is not assignable to type 'number'
mything = 'hi' as any;
// 👍 if you say "any", TypeScript says ¯\_(ツ)_/¯
```
但是真的，真的，真的将其用作最后的手段。 我们不喜欢TypeScript中的 any。

## Tip 6. 更多限制

有时TypeScript无法推断类型。 最常见的情况是一个函数参数：

```javascript
function fn(param) {
    console.log(param);
}
```

在内部，它需要在此处为param分配某种类型，因此它可以分配任何类型。 由于我们希望将any限制为绝对最小值，因此通常建议使用另一个tsconfig.json设置来限制该行为：

```json
{
    "compilerOptions": {
        "noImplicitAny": true
    }
}
```

不幸的是，我们不能在函数返回类型上使用这种安全带（需要明确输入）。 因此，如果改为使用函数fn（param）：string {我会忘记该类型（函数fn（param）{），TypeScript将不会关注我返回的内容，即使我从该函数返回了任何内容。 更准确地说：它将根据您退回或未退回的商品推断出退货价值。

幸运的是，TSLint可以为您提供帮助。 使用typedef规则，您可以使返回类型成为必需：

```json
{
    "rules": {
        "typedef": [
            true,
            "call-signature"
        ]
    }
}
```

这看起来是个好主意！

## Tip 7. 类型保护

当值具有多种类型时，必须在算法中将其考虑在内，以区分一种类型与另一种类型。 关于TypeScript的事情是它了解这种逻辑。

```typescript
type BookId = number | string;
function returnFormatterId(id: BookId) {
    return id.toUpperCase();
    // 🛑 'toUpperCase' does not exist on type 'number'.
}
function returnFormatterId(id: BookId) {
    if (typeof id === 'string') {
        // we've made sure it's a string:
        return id.toUpperCase(); // so it's 👍
    }
    // 👍 TS also understands that it
    // has to be a number here:
    return id.toFixed(2)
}
```

## Tip 8. 再谈泛型

假设我们具有这种相当通用的结构：

```typescript
interface Bookmark {
    id: string;
}
class BookmarksService {
    items: Bookmark[] = [];
}
```

您想在不同的应用程序中使用它，例如 用于存储书籍或电影。

在这样的应用程序中，您可以执行以下操作：

```typescript
interface Movie {
    id: string;
    name: string;
}
class SearchPageComponent {
    movie: Movie;
    constructor(private bs: BookmarksService) {}
    getFirstMovie() {
        // 🛑 types are not assignable
        this.movie = this.bs.items[0];
        // 👍 so you have to manually assert type:
        this.movie = this.bs.items[0] as Movie;
    }
    getSecondMovie() {
        this.movie = this.bs.items[1] as Movie;
    }
}
```

在该类中可能需要多次这种类型声明。

我们可以做的是将 `BookmarksService` 类定义为通用类：

```typescript
class BookmarksService<T> {
    items: T[] = [];
}
```

好吧，不过现在它太通用了……我们要确保此类使用的类型能够满足Bookmark接口（即具有id：string属性）。 这是改进的声明：

```typescript
class BookmarksService<T extends Bookmark> {
    items: T[] = [];
}
```

现在，在我们的SearchPageComponent中，我们只需指定一次类型：

```typescript
class SearchPageComponent {
    movie: Movie;
    constructor(private bs: BookmarksService<Movie>) {}
    getFirstMovie() {
        this.movie = this.bs.items[0]; // 👍
    }
    getSecondMovie() {
        this.movie = this.bs.items[1]; // 👍
    }
}
```

对于该通用类，还有一项可能是有用的改进-如果您以这种通用身份在其他地方使用它，而又不想编写BookmarksService <Bookmark>的话。

您可以在泛型的定义中提供默认类型：

```typescript
class BookmarksService<T extends Bookmark = Bookmark> {
    items: T[] = [];
}
const bs = new BookmarksService();
// I don't have to provide the type for the generic now
// - in that case 'bs' will be of that default type 'Bookmark'
```

## Tip 9. 路由参数

```typescript
export interface DashboardRouteParams {
  countryId: string;
  deviceId: string;
}

const routes: Routes = [{
  path: 'country/:countryId/device/:deviceId/dashboard'
}]

```

## Tip10. 根据 API 响应创建 Interface

如果您收到来自API的大量嵌套响应，那么手动键入相应的接口确实很麻烦。 这是应该由机器处理的任务！ 🤖

有两种选择:

- http://www.json2ts.com
- http://www.jsontots.com

..是其中的一些，但是坦率地说，它们的服务器通常不可用。 由于URL的记忆力很强，我通常只是从它们开始：-)为了获得最佳结果和一些其他选项，请使用

- https://app.quicktype.io/

它还提供了一个方便的Visual Studio Code插件~