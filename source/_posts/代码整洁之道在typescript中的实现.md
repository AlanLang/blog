---
title: 代码整洁之道在typescript中的实现
tags: 前端
categories: 前端
abbrlink: 52300
date: 2019-07-19 08:48:23
---

代码整洁之道在typescript中的实现
灵感来源于 [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)。

<!-- more -->

## 目录

  1. [简介](#简介)
  2. [变量](#变量)
  3. [方法](#方法)
  4. [对象和数据结构](#对象和数据结构)
  5. [类](#类)
  6. [SOLID原则](#SOLID原则)
  7. [测试](#测试)
  8. [并发](#并发)
  9. [异常处理](#异常处理)
  10. [格式化](#格式化)
  11. [注释](#注释)

## 简介

![软件质量取决于阅读代码时骂人的次数](https://www.osnews.com/images/comics/wtfm.jpg)

本文是 Robert C 所书写的关于软件工程原理的书籍《代码整洁之道》的 TypeScript版，这不是一篇风格指南，而是为了使用 TypeScript 生成可读、可复用和可重构的软件指南。
本文中并非所有的原则都必须严格遵守，其中仅有一部分原则会被普遍认同，这些仅仅是一些原则，只是致力于整洁代码的作者们多年的经验。
我们的软件工程技术只有50多年的历史，我们仍旧在学很多东西，当软件架构和架构本身一样古老时，我们更难去遵守规则，现在，就让这些指南最为评估您和您团队的 TypeScript 代码质量的炼金石吧。
此外，了解这些知识并不会让您马上成为一名优秀的软件开发人员，与代码合作多年并不意味着您不会犯错误，每一段代码都是从初稿开始，向黏土一样慢慢被塑造成最终的形状。然后，在我们与同行一起审查时，我们会慢慢去修复不完美的地方，所以不要因为改进初稿而感到挫败，让我们去打败代码吧。

**[⬆ 回到顶部](#目录)**

## 变量
### 使用有意义的变量名称
给变量取一个有意义的名称以便于帮助读者去区分变量之前的区别

**不推荐:**

```ts
function between<T>(a1: T, a2: T, a3: T): boolean {
  return a2 <= a1 && a1 <= a3;
}

```

**推荐:**

```ts
function between<T>(value: T, left: T, right: T): boolean {
  return left <= value && value <= right;
}
```

**[⬆ 回到顶部](#目录)**

### 使用容易拼读的变量名称
如果你无法拼读它，你会像一个白痴一样无法解释它

**不推荐:**

```ts
type DtaRcrd102 = {
  genymdhms: Date;
  modymdhms: Date;
  pszqint: number;
}
```

**推荐:**

```ts
type Customer = {
  generationTimestamp: Date;
  modificationTimestamp: Date;
  recordId: number;
}
```
**[⬆ 回到顶部](#目录)**

### 为同样类型的变量使用同样的词汇

**不推荐:**

```ts
function getUserInfo(): User;
function getUserDetails(): User;
function getUserData(): User;
```

**推荐:**

```ts
function getUser(): User;
```

**[⬆ 回到顶部](#目录)**

### 使用有意义的名称
相比于写代码而言，我们更多的是去阅读代码，所以你写的代码是否可读和可检索至关重要，未命名的变量会影响对我们代码里理解，并伤害读者，所以让你的变量名可检索。可以使用[TSLint](https://palantir.github.io/tslint/rules/no-magic-numbers/) 来帮助识别未命名的常量。

**不推荐:**

```ts
// 86400000 是什么鬼?
setTimeout(restart, 86400000);
```

**推荐:**

```ts
// 将它们声明为大写的命名常量.
const MILLISECONDS_IN_A_DAY = 24 * 60 * 60 * 1000;

setTimeout(restart, MILLISECONDS_IN_A_DAY);
```

**[⬆ 回到顶部](#目录)**

### 使用可解释的变量

**不推荐:**

```ts
declare const users: Map<string, User>;

for (const keyValue of users) {
}
```

**推荐:**

```ts
declare const users: Map<string, User>;

for (const [id, user] of users) {
}
```

**[⬆ 回到顶部](#目录)**

### 使用有意义的变量来避免心里映射

显示由于隐式

**不推荐:**

```ts
const u = getUser();
const s = getSubscription();
const t = charge(u, s);
```

**推荐:**

```ts
const user = getUser();
const subscription = getSubscription();
const transaction = charge(user, subscription);
```

**[⬆ 回到顶部](#目录)**

### 不要添加不必要的上下文

如果你的 class/type/object 的名称已经告诉了你，就不要要在属性里在重复写一遍。

**不推荐:**

```ts
type Car = {
  carMake: string;
  carModel: string;
  carColor: string;
}

function print(car: Car): void {
  console.log(`${car.carMake} ${car.carModel} (${car.carColor})`);
}
```

**推荐:**

```ts
type Car = {
  make: string;
  model: string;
  color: string;
}

function print(car: Car): void {
  console.log(`${car.make} ${car.model} (${car.color})`);
}
```

**[⬆ 回到顶部](#目录)**

### 尽量使用默认参数，而不是再进行判断

默认参数通常比再行判断更清晰。

**不推荐:**

```ts
function loadPages(count?: number) {
  const loadCount = count !== undefined ? count : 10;
  // ...
}
```

**推荐:**

```ts
function loadPages(count: number = 10) {
  // ...
}
```

**[⬆ 回到顶部](#目录)**

## 方法
### 参数 (最多两个)
限制方法参数的数量至关重要，因为它会使得您方法的测试变得更容易，参数数量超过三个就会造成过多的组合，你必须用每个单独的参数去测试大量的情况。
一个或两个参数是最理想的情况，如果可能的话应该尽量避免三个，应该去整合除此之外的任何东西。

通常如果你有两个以上的参数，说明你的方法做了太多的事情，如果不是，则用更高级的对象来作为参数。

如果你发现自己需要大量参数，则考虑使用对象。

可以使用[结构赋值](https://basarat.gitbooks.io/typescript/docs/destructuring.html)来明确方法所期望的参数属性。
这样有一些优点：
1. 当有人查看功能签名时，会立即清楚正在使用的属性。
2. 解构还克隆传递给函数的参数对象的指定原始值。 这有助于预防副作用。 注意：不会克隆从参数对象中解构的对象和数组。
3. TypeScript会警告您未使用的属性，如果没有解构，这将是不可能的。

**不推荐:**

```ts
function createMenu(title: string, body: string, buttonText: string, cancellable: boolean) {
  // ...
}

createMenu('Foo', 'Bar', 'Baz', true);
```

**推荐:**

```ts
function createMenu(options: { title: string, body: string, buttonText: string, cancellable: boolean }) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
});
```

你可以使用[类型别名](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases)来进一步提高可读性:

```ts

type MenuOptions = { title: string, body: string, buttonText: string, cancellable: boolean };

function createMenu(options: MenuOptions) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
});
```

**[⬆ 回到顶部](#目录)**

### 每个方法只做一件事情
这是迄今为止软件工程中最重要的规则。 当一个方法执行多个操作时，它们会更加难以编写，测试和推理。 当您可以将一个函数隔离到一个动作时，它们可以轻松地重构，并且您的代码将更加清晰。 如果你除了这个之外没有别的东西，那么你将领先于许多开发者。

**不推荐:**

```ts
function emailClients(clients: Client[]) {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```

**推荐:**

```ts
function emailClients(clients: Client[]) {
  clients.filter(isActiveClient).forEach(email);
}

function isActiveClient(client: Client) {
  const clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```

**[⬆ 回到顶部](#目录)**

### 在方法名上体现要做的事情
**不推荐:**

```ts
function addToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();

// 很难从方法名中看出增加的是什么
addToDate(date, 1);
```

**推荐:**

```ts
function addMonthToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();
addMonthToDate(date, 1);
```

**[⬆ 回到顶部](#目录)**

### 方法应该只是一个级别的抽象
如果你的一个方法有多个级别的抽象，这说明这个方法做的事情太多了。必要的拆分可以使得方法更容易被重用和更容易测试。

**不推荐:**

```ts
function parseCode(code: string) {
  const REGEXES = [ /* ... */ ];
  const statements = code.split(' ');
  const tokens = [];

  REGEXES.forEach((regex) => {
    statements.forEach((statement) => {
      // ...
    });
  });

  const ast = [];
  tokens.forEach((token) => {
    // lex...
  });

  ast.forEach((node) => {
    // parse...
  });
}
```

**推荐:**

```ts
const REGEXES = [ /* ... */ ];

function parseCode(code: string) {
  const tokens = tokenize(code);
  const syntaxTree = parse(tokens);

  syntaxTree.forEach((node) => {
    // parse...
  });
}

function tokenize(code: string): Token[] {
  const statements = code.split(' ');
  const tokens: Token[] = [];

  REGEXES.forEach((regex) => {
    statements.forEach((statement) => {
      tokens.push( /* ... */ );
    });
  });

  return tokens;
}

function parse(tokens: Token[]): SyntaxTree {
  const syntaxTree: SyntaxTree[] = [];
  tokens.forEach((token) => {
    syntaxTree.push( /* ... */ );
  });

  return syntaxTree;
}
```

**[⬆ 回到顶部](#目录)**

### 移除重复代码

尽量避免重复代码
重复代码是很糟糕的，因为这意味着如果你需要改某些逻辑，则必须去同时修改改多个地方。

想象一下，如果你经营者一家餐馆并需要追踪你的库存，比如你的西红柿、洋葱、大蒜、香料等，如果你有多个地方记录他们，每当他们变化了你就需要更新多个列表，如果你只有一个列表，那么你只用更新一个地方！

通常重复代码的出现是因为你有两个或者两个以上略有不同的东西，他们有很多共同之处，但是他们之间的差异迫使你需要有两个或者多个独立的方法来执行大部分相同的事情，删除重复代码意味着创建一个抽象，只需要一个方法/模块/类就可以处理这组不同的东西。

获得正确的抽象是至关重要的，这就是为什么你需要遵循[单一责任原则](#单一责任原则)，但是糟糕的抽象可能比重复代码更加糟糕，所以一定要小心，如果你能实现好的抽象，尽量去实现它，不要去做重复的事情。

**不推荐:**

```ts
function showDeveloperList(developers: Developer[]) {
  developers.forEach((developer) => {
    const expectedSalary = developer.calculateExpectedSalary();
    const experience = developer.getExperience();
    const githubLink = developer.getGithubLink();

    const data = {
      expectedSalary,
      experience,
      githubLink
    };

    render(data);
  });
}

function showManagerList(managers: Manager[]) {
  managers.forEach((manager) => {
    const expectedSalary = manager.calculateExpectedSalary();
    const experience = manager.getExperience();
    const portfolio = manager.getMBAProjects();

    const data = {
      expectedSalary,
      experience,
      portfolio
    };

    render(data);
  });
}
```

**推荐:**

```ts
class Developer {
  // ...
  getExtraDetails() {
    return {
      githubLink: this.githubLink,
    }
  }
}

class Manager {
  // ...
  getExtraDetails() {
    return {
      portfolio: this.portfolio,
    }
  }
}

function showEmployeeList(employee: Developer | Manager) {
  employee.forEach((employee) => {
    const expectedSalary = employee.calculateExpectedSalary();
    const experience = employee.getExperience();
    const extra = employee.getExtraDetails();

    const data = {
      expectedSalary,
      experience,
      extra,
    };

    render(data);
  });
}
```

**[⬆ 回到顶部](#目录)**

### 使用`Object.assign`或者解析构值来定义默认的对象

**不推荐:**

```ts
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu(config: MenuConfig) {
  config.title = config.title || 'Foo';
  config.body = config.body || 'Bar';
  config.buttonText = config.buttonText || 'Baz';
  config.cancellable = config.cancellable !== undefined ? config.cancellable : true;

  // ...
}

createMenu({ body: 'Bar' });
```

**推荐:**

```ts
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu(config: MenuConfig) {
  const menuConfig = Object.assign({
    title: 'Foo',
    body: 'Bar',
    buttonText: 'Baz',
    cancellable: true
  }, config);

  // ...
}

createMenu({ body: 'Bar' });
```

或者，你也可以使用解析构值来实现。

```ts
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu({ title = 'Foo', body = 'Bar', buttonText = 'Baz', cancellable = true }: MenuConfig) {
  // ...
}

createMenu({ body: 'Bar' });
```

通过显示传入`undefined`或者`null`值来避免任何副作用，也可以告诉`TypeScript`编译器不允许他们。请参阅[`--strictNullChecks`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#--strictnullchecks)

**[⬆ 回到顶部](#目录)**

### 不要将布尔值作为方法的参数

布尔值表示了你的方法不止做了一件事情，请根据布尔值拆分你的方法。

**不推荐:**

```ts
function createFile(name: string, temp: boolean) {
  if (temp) {
    fs.create(`./temp/${name}`);
  } else {
    fs.create(name);
  }
}
```

**推荐:**

```ts
function createTempFile(name: string) {
  createFile(`./temp/${name}`);
}

function createFile(name: string) {
  fs.create(name);
}
```

**[⬆ 回到顶部](#目录)**

### 避免副作用（第一部分）

如果一个方法除了返回值以外还进行了其他操作并影响了一个或者多个值，这个函数就会产生副作用。副作用可能是写入文件，修改某个全局变量，或意外地将所有资源连接给陌生人。
如果你确实需要一个方法产生副作用，例如刚才所说的写入文件之类的，那么不要使用多个方法或者类去写入特定的文件，而是只使用一个服务来实现它。
重点是避免常见的陷阱，比如在没有任何结构的对象之间共享状态，使用可以由任何东西写入的可变数据类型，而不是集中在副作用发生的位置。 如果你能做到这一点，你会比绝大多数其他程序员更快乐。

**不推荐:**

```ts
let name = 'Robert C. Martin';

function toBase64() {
  name = btoa(name);
}

toBase64();

console.log(name); // 打印的已经不是 'Robert C. Martin' 而是 'Um9iZXJ0IEMuIE1hcnRpbg=='
```

**推荐:**

```ts
const name = 'Robert C. Martin';

function toBase64(text: string): string {
  return btoa(text);
}

const encodedName = toBase64(name);
console.log(name);
```

**[⬆ 回到顶部](#目录)**

### 避免副作用（第二部分）

在JavaScript中，值类型通过值来传递，对象/数组通过引用传递，对于对象和数组，如果你在方法中进行了修改，那么该对象或数组在其他方法中都将受到影响，这有时候会很棒，但有时候也会很糟糕，让我们来思考一个糟糕的场景：
用户点击`购买`按钮，然后调用网络请求`购买`功能的接口，并将购物车的内容以数组的形式发送给后台，由于网络连接不畅，用户可能会继续重试请求。那如果在此期间用户在网络请求之前不小心点击了他们实际上不想购买的物品上的`添加到购物车`按钮，该怎么办？如果发生了这种情况并网络请求开始，那么该物品也意外得被购买了，因为它具有多购物车的引用。
一个很好的解决方案是`addItemToCart`始终克隆`cart`，编辑它，然后返回克隆。 这样可以确保任何保留在购物车参考上的其他功能都不会受到任何更改的影响。
提到这种方法的两点需要注意：
1.在某些情况下，您可能确实想要修改输入对象，但是当您采用这种编程习惯时，您会发现这些情况非常罕见。 大多数东西都可以重构，没有副作用！ （见[纯函数]（https://en.wikipedia.org/wiki/Pure_function））
克隆大对象在性能方面可能非常昂贵。 幸运的是，这在实践中不是一个大问题，因为有很好的库快速实现而不是像手动克隆对象和数组那样占用大量内存。

**不推荐:**

```ts
function addItemToCart(cart: CartItem[], item: Item): void {
  cart.push({ item, date: Date.now() });
};
```

**推荐:**

```ts
function addItemToCart(cart: CartItem[], item: Item): CartItem[] {
  return [...cart, { item, date: Date.now() }];
};
```

**[⬆ 回到顶部](#目录)**

### 不要书写全局方法
污点全局变量在JavaScript中是一种不好的做法，因为你可能会与另一个库发生冲突。 让我们考虑一个例子：如果你想扩展JavaScript的原生Array方法以获得一个可以显示两个数组之间差异的`diff`方法，该怎么办？ 您可以将新函数写入`Array.prototype`，但它可能会与另一个尝试执行相同操作的库发生冲突。 如果那个其他库只是使用`diff`找到数组的第一个和最后一个元素之间的区别怎么办？ 这就是为什么只使用类并简单地扩展`Array`全局变得更好的原因。

**不推荐:**

```ts
declare global {
  interface Array<T> {
    diff(other: T[]): Array<T>;
  }
}

if (!Array.prototype.diff) {
  Array.prototype.diff = function <T>(other: T[]): T[] {
    const hash = new Set(other);
    return this.filter(elem => !hash.has(elem));
  };
}
```

**推荐:**

```ts
class MyArray<T> extends Array<T> {
  diff(other: T[]): T[] {
    const hash = new Set(other);
    return this.filter(elem => !hash.has(elem));
  };
}
```

**[⬆ 回到顶部](#目录)**

### 使用函数式编程而不是命令式编程

**不推荐:**

```ts
const contributions = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];

let totalOutput = 0;

for (let i = 0; i < contributions.length; i++) {
  totalOutput += contributions[i].linesOfCode;
}
```

**推荐:**

```ts
const contributions = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];

const totalOutput = contributions
  .reduce((totalLines, output) => totalLines + output.linesOfCode, 0);
```

**[⬆ 回到顶部](#目录)**

### 封装条件

**不推荐:**

```ts
if (subscription.isTrial || account.balance > 0) {
  // ...
}
```

**推荐:**

```ts
function canActivateService(subscription: Subscription, account: Account) {
  return subscription.isTrial || account.balance > 0
}

if (canActivateService(subscription, account)) {
  // ...
}
```

**[⬆ 回到顶部](#目录)**

### 避免负向判断

**不推荐:**

```ts
function isEmailNotUsed(email: string): boolean {
  // ...
}

if (isEmailNotUsed(email)) {
  // ...
}
```

**推荐:**

```ts
function isEmailUsed(email): boolean {
  // ...
}

if (!isEmailUsed(node)) {
  // ...
}
```

**[⬆ 回到顶部](#目录)**

### 减少判断

这似乎是一项不可能的任务。 在第一次听到这个观点时，大多数人都会说，“如果没有`if`陈述，我该怎么做？” 答案是，在许多情况下，都可以使用多态来实现相同的任务。 第二个问题通常是，“那很好，但我为什么要那样做呢？” 答案是我们之前学到的一个简洁的代码概念：一个函数应该只做一件事。 当你有具有`if`语句的类和函数时，你告诉你的用户你的函数不止一件事。 记住，只做一件事。

**不推荐:**

```ts
class Airplane {
  private type: string;
  // ...

  getCruisingAltitude() {
    switch (this.type) {
      case '777':
        return this.getMaxAltitude() - this.getPassengerCount();
      case 'Air Force One':
        return this.getMaxAltitude();
      case 'Cessna':
        return this.getMaxAltitude() - this.getFuelExpenditure();
      default:
        throw new Error('Unknown airplane type.');
    }
  }

  private getMaxAltitude(): number {
    // ...
  }
}
```

**推荐:**

```ts
abstract class Airplane {
  protected getMaxAltitude(): number {
    // shared logic with subclasses ...
  }

  // ...
}

class Boeing777 extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getPassengerCount();
  }
}

class AirForceOne extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude();
  }
}

class Cessna extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getFuelExpenditure();
  }
}
```

**[⬆ 回到顶部](#目录)**

### 不要过度优化

现代浏览器在运行时进行了大量的优化。 很多时候，如果你正在优化那么你只是在浪费你的时间。 你可以在[这里]（https://github.com/petkaantonov/bluebird/wiki/Optimization-killers）看到缺少优化的地方。 同时针对那些，直到它们被修复为止。

**不推荐:**

```ts
// 在旧浏览器上，每次使用未缓存的`list.length`迭代都会很昂贵
// 因为`list.length`会重新计算。 但是在现代浏览器中，这已经被优化了。
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```

**推荐:**

```ts
for (let i = 0; i < list.length; i++) {
  // ...
}
```

**[⬆ 回到顶部](#目录)**

### 移除无用的代码

无用的代码和重复代码一样糟糕。 没有理由将它保留在您的代码库中。
如果它没有被调用，请删掉它！ 如果后面需要，可以在版本管理的历史中找到它。

**不推荐:**

```ts
function oldRequestModule(url: string) {
  // ...
}

function requestModule(url: string) {
  // ...
}

const req = requestModule;
inventoryTracker('apples', req, 'www.inventory-awesome.io');
```

**推荐:**

```ts
function requestModule(url: string) {
  // ...
}

const req = requestModule;
inventoryTracker('apples', req, 'www.inventory-awesome.io');
```

**[⬆ 回到顶部](#目录)**

### 使用迭代器和生成器

**不推荐:**

```ts
function fibonacci(n: number): number[] {
  if (n === 1) return [0];
  if (n === 2) return [0, 1];

  const items: number[] = [0, 1];
  while (items.length < n) {
    items.push(items[items.length - 2] + items[items.length - 1]);
  }

  return items;
}

function print(n: number) {
  fibonacci(n).forEach(fib => console.log(fib));
}

print(10);
```

**推荐:**

```ts
function* fibonacci(): IterableIterator<number> {
  let [a, b] = [0, 1];

  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

function print(n: number) {
  let i = 0;
  for (const fib of fibonacci()) {
    if (i++ === n) break;  
    console.log(fib);
  }  
}

print(10);
```
有些库允许我们用之前熟悉的方法来处理例如 `map`, `slice`, `forEach` 等等. 参考这里 [itiriri](https://www.npmjs.com/package/itiriri) (或者 [itiriri-async](https://www.npmjs.com/package/itiriri-async) 用于异步处理).

```ts
import itiriri from 'itiriri';

function* fibonacci(): IterableIterator<number> {
  let [a, b] = [0, 1];
 
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

itiriri(fibonacci())
  .take(10)
  .forEach(fib => console.log(fib));
```

**[⬆ 回到顶部](#目录)**

## 对象和数据结构

### 使用 getters 和 setters

TypeScript支持getter/setter语法。
使用getter和setter访问来自封装行为的对象的数据可能比简单地查找对象上的属性更好。
你可能会问为什么，下面是一些原因：

 - 当您想要获取对象属性之外的其他操作时，您不必查找并更改代码库中的每个访问者。
 - 在执行`set`时使添加验证变得简单。
 - 封装内部表示。
 - 获取和设置时，可以轻松添加日志记录和错误处理。
 - 您可以延迟加载对象的属性，假设从服务器获取它。

**不推荐:**

```ts
type BankAccount = {
  balance: number;
  // ...
}

const value = 100;
const account: BankAccount = {
  balance: 0,
  // ...
};

if (value < 0) {
  throw new Error('Cannot set negative balance.');
}

account.balance = value;
```

**推荐:**

```ts
class BankAccount {
  private accountBalance: number = 0;

  get balance(): number {
    return this.accountBalance;
  }

  set balance(value: number) {
    if (value < 0) {
      throw new Error('Cannot set negative balance.');
    }

    this.accountBalance = value;
  }

  // ...
}

// 现在 `BankAccount` 封装了验证逻辑
// 如果有一个我们想增加额外的验证规则,
// 我们只需要去改变 `setter` 的实现,
// 而不影响其他代码.
const account = new BankAccount();
account.balance = 100;
```

**[⬆ 回到顶部](#目录)**

### 设置 private/protected

TypeScript 支持 `public` *(默认)*, `protected` 还有 `private`.  

**不推荐:**

```ts
class Circle {
  radius: number;
  
  constructor(radius: number) {
    this.radius = radius;
  }

  perimeter() {
    return 2 * Math.PI * this.radius;
  }

  surface() {
    return Math.PI * this.radius * this.radius;
  }
}
```

**推荐:**

```ts
class Circle {
  constructor(private readonly radius: number) {
  }

  perimeter() {
    return 2 * Math.PI * this.radius;
  }

  surface() {
    return Math.PI * this.radius * this.radius;
  }
}
```

**[⬆ 回到顶部](#目录)**

### 更倾向于不可变
TypeScript 的类型系统允许在接口/类的各个属性上标记`readonly`，对于更高级的场景，有一个内置里类型`Readonly`，它接受一个泛型`<T>`并使用映射类型将其所有属性标记为只读，请见[这里](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types)。

**不推荐:**

```ts
interface Config {
  host: string;
  port: string;
  db: string;
}
```

**推荐:**

```ts
interface Config {
  readonly host: string;
  readonly port: string;
  readonly db: string;
}
```

**[⬆ 回到顶部](#目录)**

### `type` 对比 `interface`

当您使用联合或者交叉时使用`type`，当想要继承或者实现时使用`interface`，但是没有严格的规则，请使用适合自己的规则。
有关更详细的说明，请参阅[这里](https://stackoverflow.com/questions/37233735/typescript-interfaces-vs-types/54101543#54101543)了解ts中`type`和`interface`之间的区别。

**不推荐:**

```ts
interface EmailConfig {
  // ...
}

interface DbConfig {
  // ...
}

interface Config {
  // ...
}

//...

type Shape = {
  // ...
}
```

**推荐:**

```ts

type EmailConfig = {
  // ...
}

type DbConfig = {
  // ...
}

type Config  = EmailConfig | DbConfig;

// ...

interface Shape {
  // ...
}

class Circle implements Shape {
  // ...
}

class Square implements Shape {
  // ...
}
```

**[⬆ 回到顶部](#目录)**

## 类

### 类应该是精简的
类的规模是由其责任来衡量的。 遵循*单一责任原则*一个类应该很小。

**不推荐:**

```ts
class Dashboard {
  getLanguage(): string { /* ... */ }
  setLanguage(language: string): void { /* ... */ }
  showProgress(): void { /* ... */ }
  hideProgress(): void { /* ... */ }
  isDirty(): boolean { /* ... */ }
  disable(): void { /* ... */ }
  enable(): void { /* ... */ }
  addSubscription(subscription: Subscription): void { /* ... */ }
  removeSubscription(subscription: Subscription): void { /* ... */ }
  addUser(user: User): void { /* ... */ }
  removeUser(user: User): void { /* ... */ }
  goToHomePage(): void { /* ... */ }
  updateProfile(details: UserDetails): void { /* ... */ }
  getVersion(): string { /* ... */ }
  // ...
}

```

**推荐:**

```ts
class Dashboard {
  disable(): void { /* ... */ }
  enable(): void { /* ... */ }
  getVersion(): string { /* ... */ }
}

// 通过将其他方法移动到其他类中来分担责任。
// ...
```

**[⬆ 回到顶部](#目录)**

### 高内聚和低耦合
聚合程度决定了类成员的彼此相关程度。在理想情况下，每个方法都应该使用类中的所有字段，然后我们称之为*最大内聚类*，但这是不容易实现的，甚至是不可取的，但是我们都更喜欢凝聚力。
耦合是指两个类相互依赖的程度，如果其中一个类的修改不会影响另一个类，则称这两个类是低耦合的。
良好的软件设计应该是**高内聚**和**低耦合**。

**不推荐:**

```ts
class UserManager {
  // 不推荐理由: 每一个私有变量都有特定的一组方法使用.
  // 这明确的表名这个类不止做了一件事情.
  constructor(
    private readonly db: Database,
    private readonly emailSender: EmailSender) {
  }

  async getUser(id: number): Promise<User> {
    return await db.users.findOne({ id });
  }

  async getTransactions(userId: number): Promise<Transaction[]> {
    return await db.transactions.find({ userId });
  }

  async sendGreeting(): Promise<void> {
    await emailSender.send('Welcome!');
  }

  async sendNotification(text: string): Promise<void> {
    await emailSender.send(text);
  }

  async sendNewsletter(): Promise<void> {
    // ...
  }
}
```

**推荐:**

```ts
class UserService {
  constructor(private readonly db: Database) {
  }

  async getUser(id: number): Promise<User> {
    return await this.db.users.findOne({ id });
  }

  async getTransactions(userId: number): Promise<Transaction[]> {
    return await this.db.transactions.find({ userId });
  }
}

class UserNotifier {
  constructor(private readonly emailSender: EmailSender) {
  }

  async sendGreeting(): Promise<void> {
    await this.emailSender.send('Welcome!');
  }

  async sendNotification(text: string): Promise<void> {
    await this.emailSender.send(text);
  }

  async sendNewsletter(): Promise<void> {
    // ...
  }
}
```

**[⬆ 回到顶部](#目录)**

### 比起继承来更喜欢使用组合
正如[设计模式](https://en.wikipedia.org/wiki/Design_Patterns)中所说的那样，你应该**尽可能得优先组合而不是继承**，有很多很好的理由去使用继承，同时也有很多很好的理由去使用组合，这就意味着如果你的思想本能的是使用继承，那就试着想一想，使用组合是不是能更好的解决你的问题，一般在某些情况下它是可以的。
你可能想知道我什么时候该使用组合，这取决于你手头的问题，不过下面的列表列出了使用继承更好的一些情况：
1. 你的继承表示的是"is-a"关系而不是"has-a"关系（Human-> Animal vs. User-> UserDetails）。

2. 您可以重用基类中的代码（人类可以像所有动物一样移动）。

3. 您希望通过更改基类对派生类进行全局更改。 （改变所有动物移动时的热量消耗）。

**不推荐:**

```ts
class Employee {
  constructor(
    private readonly name: string,
    private readonly email: string) {
  }

  // ...
}

class EmployeeTaxData extends Employee {
  constructor(
    name: string,
    email: string,
    private readonly ssn: string,
    private readonly salary: number) {
    super(name, email);
  }

  // ...
}
```

**推荐:**

```ts
class Employee {
  private taxData: EmployeeTaxData;

  constructor(
    private readonly name: string,
    private readonly email: string) {
  }

  setTaxData(ssn: string, salary: number): Employee {
    this.taxData = new EmployeeTaxData(ssn, salary);
    return this;
  }

  // ...
}

class EmployeeTaxData {
  constructor(
    public readonly ssn: string,
    public readonly salary: number) {
  }

  // ...
}
```

**[⬆ 回到顶部](#目录)**

### 使用链式编程

这种模式非常有用，并且常用于许多库中。 它允许您的代码更直观，而不是冗长。 因此，请使用链式编程并检查代码的清洁程度。

**不推荐:**

```ts
class QueryBuilder {
  private collection: string;
  private pageNumber: number = 1;
  private itemsPerPage: number = 100;
  private orderByFields: string[] = [];

  from(collection: string): void {
    this.collection = collection;
  }

  page(number: number, itemsPerPage: number = 100): void {
    this.pageNumber = number;
    this.itemsPerPage = itemsPerPage;
  }

  orderBy(...fields: string[]): void {
    this.orderByFields = fields;
  }

  build(): Query {
    // ...
  }
}

// ...

const queryBuilder = new QueryBuilder();
queryBuilder.from('users');
queryBuilder.page(1, 100);
queryBuilder.orderBy('firstName', 'lastName');

const query = queryBuilder.build();
```

**推荐:**

```ts
class QueryBuilder {
  private collection: string;
  private pageNumber: number = 1;
  private itemsPerPage: number = 100;
  private orderByFields: string[] = [];

  from(collection: string): this {
    this.collection = collection;
    return this;
  }

  page(number: number, itemsPerPage: number = 100): this {
    this.pageNumber = number;
    this.itemsPerPage = itemsPerPage;
    return this;
  }

  orderBy(...fields: string[]): this {
    this.orderByFields = fields;
    return this;
  }

  build(): Query {
    // ...
  }
}

// ...

const query = new QueryBuilder()
  .from('users')
  .page(1, 100)
  .orderBy('firstName', 'lastName')
  .build();
```

**[⬆ 回到顶部](#目录)**

## SOLID原则

### 单一责任原则 (SRP)
正如代码整洁之道上所说，“应该有且仅有一个原因引起类的变更”，要最大幅度的减少类的变更，如果一个类的功能太负责而你修改的它的一部分，那么很难保证是否会影响到其他地方。

**不推荐:**

```ts
class UserSettings {
  constructor(private readonly user: User) {
  }

  changeSettings(settings: UserSettings) {
    if (this.verifyCredentials()) {
      // ...
    }
  }

  verifyCredentials() {
    // ...
  }
}
```

**推荐:**

```ts
class UserAuth {
  constructor(private readonly user: User) {
  }

  verifyCredentials() {
    // ...
  }
}


class UserSettings {
  private readonly auth: UserAuth;

  constructor(private readonly user: User) {
    this.auth = new UserAuth(user);
  }

  changeSettings(settings: UserSettings) {
    if (this.auth.verifyCredentials()) {
      // ...
    }
  }
}
```

**[⬆ 回到顶部](#目录)**

### 开放封闭原则 (OCP)

一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。但并不意味着不做任何修改，低层模块的变更，必然要有高层模块进行耦合，否则就是一个孤立无意义的代码片段。

**不推荐:**

```ts
class AjaxAdapter extends Adapter {
  constructor() {
    super();
  }

  // ...
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
  }

  // ...
}

class HttpRequester {
  constructor(private readonly adapter: Adapter) {
  }

  async fetch<T>(url: string): Promise<T> {
    if (this.adapter instanceof AjaxAdapter) {
      const response = await makeAjaxCall<T>(url);
      // 转换response并返回
    } else if (this.adapter instanceof NodeAdapter) {
      const response = await makeHttpCall<T>(url);
      // 转换response并返回
    }
  }
}

function makeAjaxCall<T>(url: string): Promise<T> {
  // 转换response并返回
}

function makeHttpCall<T>(url: string): Promise<T> {
  // 转换response并返回
}
```

**推荐:**

```ts
abstract class Adapter {
  abstract async request<T>(url: string): Promise<T>;

}

class AjaxAdapter extends Adapter {
  constructor() {
    super();
  }

  async request<T>(url: string): Promise<T>{
    // 转换response并返回
  }

  // ...
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
  }

  async request<T>(url: string): Promise<T>{
    // 转换response并返回
  }

  // ...
}

class HttpRequester {
  constructor(private readonly adapter: Adapter) {
  }

  async fetch<T>(url: string): Promise<T> {
    const response = await this.adapter.request<T>(url);
    // 转换response并返回
  }
}
```

**[⬆ 回到顶部](#目录)**

### 里氏替换原则 (LSP)

只要父类能出现的地方子类都可以出现，而且替换为子类也不会产生任何错误或异常，使用者可有根本就不需要知道是父类还是子类。但是，反过来就不行了，有子类出现的地方，父类未必能适应。

**不推荐:**

```ts
class Rectangle {
  constructor(
    protected width: number = 0,
    protected height: number = 0) {

  }

  setColor(color: string): this {
    // ...
  }

  render(area: number) {
    // ...
  }

  setWidth(width: number): this {
    this.width = width;
    return this;
  }

  setHeight(height: number): this {
    this.height = height;
    return this;
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width: number): this {
    this.width = width;
    this.height = width;
    return this;
  }

  setHeight(height: number): this {
    this.width = height;
    this.height = height;
    return this;
  }
}

function renderLargeRectangles(rectangles: Rectangle[]) {
  rectangles.forEach((rectangle) => {
    const area = rectangle
      .setWidth(4)
      .setHeight(5)
      .getArea();
    rectangle.render(area);
  });
}

const rectangles = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles(rectangles);
```

**推荐:**

```ts
abstract class Shape {
  setColor(color: string): this {
    // ...
  }

  render(area: number) {
    // ...
  }

  abstract getArea(): number;
}

class Rectangle extends Shape {
  constructor(
    private readonly width = 0,
    private readonly height = 0) {
    super();
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(private readonly length: number) {
    super();
  }

  getArea(): number {
    return this.length * this.length;
  }
}

function renderLargeShapes(shapes: Shape[]) {
  shapes.forEach((shape) => {
    const area = shape.getArea();
    shape.render(area);
  });
}

const shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)];
renderLargeShapes(shapes);
```

**[⬆ 回到顶部](#目录)**

### 接口隔离原则 (ISP)

建立单一接口，不要建立臃肿庞大的接口。接口隔离原则与单一职责的审视角度是不相同的，单一职责要求的是类和接口职责单一，注重的是职责，这是业务逻辑上的划分，而接口隔离原则要求接口的方法尽量少。

**不推荐:**

```ts
interface SmartPrinter {
  print();
  fax();
  scan();
}

class AllInOnePrinter implements SmartPrinter {
  print() {
    // ...
  }  
  
  fax() {
    // ...
  }

  scan() {
    // ...
  }
}

class EconomicPrinter implements SmartPrinter {
  print() {
    // ...
  }  
  
  fax() {
    throw new Error('Fax not supported.');
  }

  scan() {
    throw new Error('Scan not supported.');
  }
}
```

**推荐:**

```ts
interface Printer {
  print();
}

interface Fax {
  fax();
}

interface Scanner {
  scan();
}

class AllInOnePrinter implements Printer, Fax, Scanner {
  print() {
    // ...
  }  
  
  fax() {
    // ...
  }

  scan() {
    // ...
  }
}

class EconomicPrinter implements Printer {
  print() {
    // ...
  }
}
```

**[⬆ 回到顶部](#目录)**

### 依赖倒置原则 (DIP)

含义：
1.高层模块不应该依赖低层模块，两者都应该依赖其抽象。
2.抽象不应该依赖细节。
3.细节应该依赖抽象。
  
DIP 通常通过使用控制反转 (IoC) 来实现. 这里有一个例子 [InversifyJs](https://www.npmjs.com/package/inversify)

**不推荐:**

```ts
import { readFile as readFileCb } from 'fs';
import { promisify } from 'util';

const readFile = promisify(readFileCb);

type ReportData = {
  // ..
}

class XmlFormatter {
  parse<T>(content: string): T {
    // 将XML字符串序列化为对象.
  }
}

class ReportReader {

  // 不推荐: 我们已经创建了对特定请求实现的依赖。
  // 我们应该让ReportReader依赖于一个解析方法：`parse`
  private readonly formatter = new XmlFormatter();

  async read(path: string): Promise<ReportData> {
    const text = await readFile(path, 'UTF8');
    return this.formatter.parse<ReportData>(text);
  }
}

// ...
const reader = new ReportReader();
await report = await reader.read('report.xml');
```

**推荐:**

```ts
import { readFile as readFileCb } from 'fs';
import { promisify } from 'util';

const readFile = promisify(readFileCb);

type ReportData = {
  // ..
}

interface Formatter {
  parse<T>(content: string): T;
}

class XmlFormatter implements Formatter {
  parse<T>(content: string): T {
    // 将XML字符串序列化为对象.
  }
}


class JsonFormatter implements Formatter {
  parse<T>(content: string): T {
    // 将JSON字符串序列化为对象.
  }
}

class ReportReader {
  constructor(private readonly formatter: Formatter) {
  }

  async read(path: string): Promise<ReportData> {
    const text = await readFile(path, 'UTF8');
    return this.formatter.parse<ReportData>(text);
  }
}

// ...
const reader = new ReportReader(new XmlFormatter());
await report = await reader.read('report.xml');

// 如果我们需要解析json
const reader = new ReportReader(new JsonFormatter());
await report = await reader.read('report.json');
```

**[⬆ 回到顶部](#目录)**

## 测试
测试比编码更重要，如果你的测试数量很少或者根本没有，那么每次修改代码时，你都不会确定有没有破坏其他地方，决定要到什么样的测试程度取决于你的团队，但是100%的覆盖率会让你开发的时候更有信心。这意味你需要一个好的框架之外还需要一个好的[覆盖统计工具](https://github.com/gotwarlost/istanbul)
没有理由不写测试，有好多很好的js测试框架对typescript都有很好的支持，当您找到适合您团队的时，请始终为您引入的每个新功能/模块编写测试。 如果您首选的方法是测试驱动开发（TDD），那很好，但重点是确保在启动任何功能或重构现有功能之前达到覆盖目标。

### 测试驱动开发的三条原则

1. 除非这能让失败的单元测试通过，否则不允许去编写任何的产品代码。
2. 只允许编写刚好能够导致失败的单元测试。 （编译失败也属于一种失败）
3. 只允许编写刚好能够导致一个单元测试失败的产品代码。

**[⬆ 回到顶部](#目录)**

### F.I.R.S.T. 原则

整洁的测试需要遵循以下原则:

- **快速** 测试运行起来应该是快速的因为我们会经常运行他们。

- **独立** 每天测试不允许互相依赖，无论是独立运行还是顺序运行他们的返回应该是相同的。

- **可复现** 测试应该是在任意环境下都是可复现的。

- **自我确认** 测试需要自己体现**通过**或是**不通过**，而不是靠开发者自己比对日志来判断是否通过。

- **及时** 单元测试应该在生产代码之前编写。 如果您在生产代码之后编写测试，您可能会发现编写测试太辛苦了。

**[⬆ 回到顶部](#目录)**

### 测试单一责任

测试也需要遵循**单一责任原则**。

**不推荐:**

```ts
import { assert } from 'chai';

describe('AwesomeDate', () => {
  it('handles date boundaries', () => {
    let date: AwesomeDate;

    date = new AwesomeDate('1/1/2015');
    assert.equal('1/31/2015', date.addDays(30));

    date = new AwesomeDate('2/1/2016');
    assert.equal('2/29/2016', date.addDays(28));

    date = new AwesomeDate('2/1/2015');
    assert.equal('3/1/2015', date.addDays(28));
  });
});
```

**推荐:**

```ts
import { assert } from 'chai';

describe('AwesomeDate', () => {
  it('handles 30-day months', () => {
    const date = new AwesomeDate('1/1/2015');
    assert.equal('1/31/2015', date.addDays(30));
  });

  it('handles leap year', () => {
    const date = new AwesomeDate('2/1/2016');
    assert.equal('2/29/2016', date.addDays(28));
  });

  it('handles non-leap year', () => {
    const date = new AwesomeDate('2/1/2015');
    assert.equal('3/1/2015', date.addDays(28));
  });
});
```

**[⬆ 回到顶部](#目录)**

### 测试的名字应该表示了测试的意图

**不推荐:**

```ts
describe('Calendar', () => {
  it('2/29/2020', () => {
    // ...
  });

  it('throws', () => {
    // ...
  });
});
```

**推荐:**

```ts
describe('Calendar', () => {
  it('should handle leap year', () => {
    // ...
  });

  it('should throw when format is invalid', () => {
    // ...
  });
});
```

**[⬆ 回到顶部](#目录)**

## 并发

### promises 对比 callbacks
回调嵌套过多会产生**回调地狱**，使用promises可以有效解决这个问题。

**不推荐:**

```ts
import { get } from 'request';
import { writeFile } from 'fs';

function downloadPage(url: string, saveTo: string, callback: (error: Error, content?: string) => void) {
  get(url, (error, response) => {
    if (error) {
      callback(error);
    } else {
      writeFile(saveTo, response.body, (error) => {
        if (error) {
          callback(error);
        } else {
          callback(null, response.body);
        }
      });
    }
  });
}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html', (error, content) => {
  if (error) {
    console.error(error);
  } else {
    console.log(content);
  }
});
```

**推荐:**

```ts
import { get } from 'request';
import { writeFile } from 'fs';
import { promisify } from 'util';

const write = promisify(writeFile);

function downloadPage(url: string, saveTo: string): Promise<string> {
  return get(url)
    .then(response => write(saveTo, response));
}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html')
  .then(content => console.log(content))
  .catch(error => console.error(error));  
```

**[⬆ 回到顶部](#目录)**

### Async/Await 比 Promises 更加简洁

使用`async` /`await`语法，您可以编写比链式调用更清晰，更易理解的代码。 在一个以`async`关键字为前缀的方法中，您可以告诉JavaScript运行时暂停执行`await`关键字上的代码（当在promise上使用时）。

**不推荐:**

```ts
import { get } from 'request';
import { writeFile } from 'fs';
import { promisify } from 'util';

const write = util.promisify(writeFile);

function downloadPage(url: string, saveTo: string): Promise<string> {
  return get(url).then(response => write(saveTo, response));
}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html')
  .then(content => console.log(content))
  .catch(error => console.error(error));  
```

**推荐:**

```ts
import { get } from 'request';
import { writeFile } from 'fs';
import { promisify } from 'util';

const write = promisify(writeFile);

async function downloadPage(url: string, saveTo: string): Promise<string> {
  const response = await get(url);
  await write(saveTo, response);
  return response;
}

try {
  const content = await downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html');
  console.log(content);
} catch (error) {
  console.error(error);
}
```

**[⬆ 回到顶部](#目录)**

## 异常处理

抛出异常是件好事！ 它们意味着运行时已经成功识别出程序中的某些内容出错并及时通知您。

### 始终使用`Error`对象来抛出错误。

**不推荐:**

```ts
function calculateTotal(items: Item[]): number {
  throw 'Not implemented.';
}

function get(): Promise<Item[]> {
  return Promise.reject('Not implemented.');
}
```

**推荐:**

```ts
function calculateTotal(items: Item[]): number {
  throw new Error('Not implemented.');
}

function get(): Promise<Item[]> {
  return Promise.reject(new Error('Not implemented.'));
}

async function get(): Promise<Item[]> {
  throw new Error('Not implemented.');
}
```

**[⬆ 回到顶部](#目录)**

### 不要忽略掉捕捉后的错误

对捕获的错误不执行任何操作并不能使您能够修复或对所述错误做出反应。 将错误记录到控制台（`console.log`）也不是很好，因为它常常会被其他东西所淹没。 所以你应该有一个统一的地方集中处理这些错误。

**不推荐:**

```ts
try {
  functionThatMightThrow();
} catch (error) {
  console.log(error);
}

// 或者更严重

try {
  functionThatMightThrow();
} catch (error) {
  // 直接忽略掉
}
```

**推荐:**

```ts
import { logger } from './logging'

try {
  functionThatMightThrow();
} catch (error) {
  logger.log(error);
}
```

**[⬆ 回到顶部](#目录)**

### 不要忽略promises 的 rejected

就像不能忽略`try/catch`到的异常一样

**不推荐:**

```ts
getUser()
  .then((user: User) => {
    return sendEmail(user.email, 'Welcome!');
  })
  .catch((error) => {
    console.log(error);
  });
```

**推荐:**

```ts
import { logger } from './logging'

getUser()
  .then((user: User) => {
    return sendEmail(user.email, 'Welcome!');
  })
  .catch((error) => {
    logger.log(error);
  });

// 或者

try {
  const user = await getUser();
  await sendEmail(user.email, 'Welcome!');
} catch (error) {
  logger.log(error);
}
```

**[⬆ 回到顶部](#目录)**

## 格式化

格式化是主观的。 像这里的许多规则一样，没有必须遵循特定硬性规则。 重点是*不要过于格式化*格式化。 有很多工具可以实现自动化。 用一个就好！ 工程师争论格式化是浪费时间和金钱的。 遵循的一般规则是*保持一致的格式规则*就可以了。

TypeScript 有一个很强大的工具叫做 [TSLint](https://palantir.github.io/tslint/). 它是一个静态分析工具，可以帮助您显着提高代码的可读性和可维护性。 下面是您可以在项目中引用的TSLint配置：

- [TSLint Config Standard](https://www.npmjs.com/package/tslint-config-standard) - 标准风格规则

- [TSLint Config Airbnb](https://www.npmjs.com/package/tslint-config-airbnb) - Airbnb风格

- [TSLint Clean Code](https://www.npmjs.com/package/tslint-clean-code) - 代码整洁之道的规则

- [TSLint react](https://www.npmjs.com/package/tslint-react) - React 和 JSX 的规则

- [TSLint + Prettier](https://www.npmjs.com/package/tslint-config-prettier) -  [Prettier](https://github.com/prettier/prettier) 代码风格

- [ESLint rules for TSLint](https://www.npmjs.com/package/tslint-eslint-rules) - ESLint 规则

### 使用一致的写法

这些规则是主观的，所以你的团队可以选择按自己的风格制定。 关键是，无论你选择什么，只要*保持一致*。

**不推荐:**

```ts
const DAYS_IN_WEEK = 7;
const daysInMonth = 30;

const songs = ['Back In Black', 'Stairway to Heaven', 'Hey Jude'];
const Artists = ['ACDC', 'Led Zeppelin', 'The Beatles'];

function eraseDatabase() {}
function restore_database() {}

type animal = { /* ... */ }
type Container = { /* ... */ }
```

**推荐:**

```ts
const DAYS_IN_WEEK = 7;
const DAYS_IN_MONTH = 30;

const SONGS = ['Back In Black', 'Stairway to Heaven', 'Hey Jude'];
const ARTISTS = ['ACDC', 'Led Zeppelin', 'The Beatles'];

function eraseDatabase() {}
function restoreDatabase() {}

type Animal = { /* ... */ }
type Container = { /* ... */ }
```


**[⬆ 回到顶部](#目录)**

### 调用者和被调用者的方法应该是相邻的

如果函数调用另一个函数，请在源文件中设置这两个函数相邻。 理想情况下，将呼叫者保持在被叫者正上方。
我们倾向于从头到尾阅读代码，就像报纸一样。 因此，请以这种方式阅读您的代码。

**不推荐:**

```ts
class PerformanceReview {
  constructor(private readonly employee: Employee) {
  }

  private lookupPeers() {
    return db.lookup(this.employee.id, 'peers');
  }

  private lookupManager() {
    return db.lookup(this.employee, 'manager');
  }

  private getPeerReviews() {
    const peers = this.lookupPeers();
    // ...
  }

  review() {
    this.getPeerReviews();
    this.getManagerReview();
    this.getSelfReview();

    // ...
  }

  private getManagerReview() {
    const manager = this.lookupManager();
  }

  private getSelfReview() {
    // ...
  }
}

const review = new PerformanceReview(employee);
review.review();
```

**推荐:**

```ts
class PerformanceReview {
  constructor(private readonly employee: Employee) {
  }

  review() {
    this.getPeerReviews();
    this.getManagerReview();
    this.getSelfReview();

    // ...
  }

  private getPeerReviews() {
    const peers = this.lookupPeers();
    // ...
  }

  private lookupPeers() {
    return db.lookup(this.employee.id, 'peers');
  }

  private getManagerReview() {
    const manager = this.lookupManager();
  }

  private lookupManager() {
    return db.lookup(this.employee, 'manager');
  }

  private getSelfReview() {
    // ...
  }
}

const review = new PerformanceReview(employee);
review.review();
```

**[⬆ 回到顶部](#目录)**

### 组织 imports

使用简洁易读的import语句，您可以快速查看当前代码的依赖关系。 确保对`import`语句应用以下良好实践：

- 导入语句应按字母顺序排列并分组。
- 不需要的引用应该被移除。
- 命名导入必须按字母顺序排列 例如: `import {A, B, C} from 'foo';`
- 导入源必须在组内按字母顺序排列, 例如: `import * as foo from 'a'; import * as bar from 'b';`
- 每一组引用之间用空行分隔。
- 分组之间需要遵循以下顺序:
  - Polyfills (例如. `import 'reflect-metadata';`)
  - Node 内置模块 (例如. `import fs from 'fs';`)
  - 外部模块 (例如. `import { query } from 'itiriri';`)
  - 内部模块 (例如. `import { UserService } from 'src/services/userService';`)
  - 来自父目录的模块 (例如. `import foo from '../foo'; import qux from '../../foo/qux';`)
  - 来自兄弟目录或同目录的模块 (例如. `import bar from './bar'; import baz from './bar/baz';`)

**不推荐:**

```ts
import { TypeDefinition } from '../types/typeDefinition';
import { AttributeTypes } from '../model/attribute';
import { ApiCredentials, Adapters } from './common/api/authorization';
import fs from 'fs';
import { ConfigPlugin } from './plugins/config/configPlugin';
import { BindingScopeEnum, Container } from 'inversify';
import 'reflect-metadata';
```

**推荐:**

```ts
import 'reflect-metadata';

import fs from 'fs';
import { BindingScopeEnum, Container } from 'inversify';

import { AttributeTypes } from '../model/attribute';
import { TypeDefinition } from '../types/typeDefinition';

import { ApiCredentials, Adapters } from './common/api/authorization';
import { ConfigPlugin } from './plugins/config/configPlugin';
```

**[⬆ 回到顶部](#目录)**

### 使用别名

通过在`tsconfig.json`的compilerOptions部分中定义路径和baseUrl属性来创建更漂亮的导入。
这样可以在进行导入时避免长的相对路径。

**不推荐:**

```ts
import { UserService } from '../../../services/UserService';
```

**推荐:**

```ts
import { UserService } from '@services/UserService';
```

```js
// tsconfig.json
...
  "compilerOptions": {
    ...
    "baseUrl": "src",
    "paths": {
      "@services": ["services/*"]
    }
    ...
  }
...
```

**[⬆ 回到顶部](#目录)**

## 注释

当没有注释无法表达的时候，才使用注释

### 更倾向于自解释代码而不是注释

**不推荐:**

```ts
// 检查订阅是否到期
if (subscription.endDate > Date.now) {  }
```

**推荐:**

```ts
const isSubscriptionActive = subscription.endDate > Date.now;
if (isSubscriptionActive) { /* ... */ }
```

**[⬆ 回到顶部](#目录)**

### 不要在代码库中留下注释的代码

版本控制的存在是有原因的。 在您的历史记录中保留旧代码。

**不推荐:**

```ts
type User = {
  name: string;
  email: string;
  // age: number;
  // jobPosition: string;
}
```

**推荐:**

```ts
type User = {
  name: string;
  email: string;
}
```

**[⬆ 回到顶部](#目录)**

### 不要使用日记评论

请记住，使用版本控制！ 不需要无用代码，注释代码，尤其是日记评论。 使用`git log`获取历史记录！

**不推荐:**

```ts
/**
 * 2016-12-20: Removed monads, didn't understand them (RM)
 * 2016-10-01: Improved using special monads (JP)
 * 2016-02-03: Added type-checking (LI)
 * 2015-03-14: Implemented combine (JR)
 */
function combine(a: number, b: number): number {
  return a + b;
}
```

**推荐:**

```ts
function combine(a: number, b: number): number {
  return a + b;
}
```

**[⬆ 回到顶部](#目录)**

### 禁止位置分隔
这会增加代码噪声，大多数IDE是支持代码折叠的，请使用。

**不推荐:**

```ts
////////////////////////////////////////////////////////////////////////////////
// Client class
////////////////////////////////////////////////////////////////////////////////
class Client {
  id: number;
  name: string;
  address: Address;
  contact: Contact;

  ////////////////////////////////////////////////////////////////////////////////
  // public methods
  ////////////////////////////////////////////////////////////////////////////////
  public describe(): string {
    // ...
  }

  ////////////////////////////////////////////////////////////////////////////////
  // private methods
  ////////////////////////////////////////////////////////////////////////////////
  private describeAddress(): string {
    // ...
  }

  private describeContact(): string {
    // ...
  }
};
```

**推荐:**

```ts
class Client {
  id: number;
  name: string;
  address: Address;
  contact: Contact;

  public describe(): string {
    // ...
  }

  private describeAddress(): string {
    // ...
  }

  private describeContact(): string {
    // ...
  }
};
```

**[⬆ 回到顶部](#目录)**

### TODO 标记

当您发现自己需要在代码中留下注释以进行后续改进时，使用`// TODO`注释来实现。 大多数IDE都支持这种注释，方便你可以快速浏览整个待办事项列表。

**不推荐:**

```ts
function getActiveSubscriptions(): Promise<Subscription[]> {
  // ensure `dueDate` is indexed.
  return db.subscriptions.find({ dueDate: { $lte: new Date() } });
}
```

**推荐:**

```ts
function getActiveSubscriptions(): Promise<Subscription[]> {
  // TODO: ensure `dueDate` is indexed.
  return db.subscriptions.find({ dueDate: { $lte: new Date() } });
}
```

**[⬆ 回到顶部](#目录)**