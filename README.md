# 油记·示例插件

[油记](https://github.com/fuelnote/fuelnote)的数据源插件示例，也是写插件的模板。它不连任何服务：交易、车辆、行程、车况、养修、油价和信息卡片都是按当前时间编的，演示[插件契约](https://github.com/fuelnote/fuelnote/blob/main/docs/plugins.md)里的每一个动作。

- 想先看看插件在油记里是什么样子：按「[试用](#试用)」装上它。
- 想接入自己的数据：在本仓库点「Use this template」建一个自己的仓库，按「[写自己的插件](#写自己的插件)」改。

字段的定义只以油记仓库的 [`docs/plugins.md`](https://github.com/fuelnote/fuelnote/blob/main/docs/plugins.md)（下文叫「契约」）为准。本文只讲怎么做，用到哪些字段时写契约的节号。

## 插件是什么

数据源插件是一个独立的 [Scripting](https://scriptingapp.github.io/guide/) 脚本。油记需要数据时，在后台用 `Script.run` 运行它；它只回答「有哪些数据」，不写油记的账本，也不知道油记的表单里填了什么。几笔交易合成一次加油、填表、核对，都在油记里做；把数据源的原始数据整理成契约里的格式，在插件里做。

插件能提供的数据：

| 能力 | 油记用它做什么 | 契约 |
|---|---|---|
| `fuel` | 记加油时「从 <插件名> 选」一笔或几笔交易 | 第 5 节 |
| `vehicles` | 关联车辆；提供 `trips`、`vehicle_status`、`service` 时必须有 | 第 6 节 |
| `trips` | 记加油时按停车的位置认油站、按熄火时的里程表填里程 | 第 6 节 |
| `vehicle_status` | 记录页「当前」的剩余油量、续航和里程 | 第 6 节 |
| `service` | 记费用时选一次进店，要记的几笔一起填好 | 第 7 节 |
| `fuel_price` | 按所在地区的挂牌价算加满要多少钱 | 第 8 节 |
| `info` | 记录页和小组件里的信息卡片，例如附近油站今天的优惠 | 第 9 节 |

只实现你的数据源有的那几样就行。

## 试用

1. 在 Scripting 里用这个地址导入脚本（总是最新版本）：

   ```
   https://github.com/fuelnote/fuelnote-plugin-example/releases/latest/download/fuelnote-plugin-example.zip
   ```

2. 打开油记的「设置 › 数据源 › 添加插件」，选「油记·示例插件」。
3. 记加油时点「从示例选」，能看到编的交易。最近一次分两笔付，可以合成一次加油；旁边单价不同、油站不同的两笔不能合成。
4. 记录页「当前」下面有一组「示例卡片」。轻点打开，长按一行可以导航到那个示例油站。
5. 在 Scripting 里直接运行这个插件，可以切换「已连接」「已过期」「未连接」，看油记在出错时的样子。

## 写自己的插件

### 1. 建仓库、起名字

1. 在本仓库点「Use this template」，建一个自己的仓库。仓库名建议 `fuelnote-plugin-<数据源>`。用到账号、Cookie 或没有公开的接口时，仓库设成私有。
2. 改 `script.json`：
   - `name`：脚本名，也是 Scripting 里的文件夹名。建议写 `FuelNote<数据源>`，例如 `FuelNoteAcme`。定了不要改：油记按脚本名调用插件，改了要在油记里移除再添加。
   - `localizedNames.zh`：显示名，写「油记·<数据源>」，在 Scripting 的脚本列表里认得出是油记的插件。
   - `description`、`localizedDescriptions`、`author`；`version` 从 `1.0.0` 开始。
3. `fuelnote.json` 保持 `{ "contract": 1 }`：「添加插件」只列出带这个文件的脚本。
4. 改 `index.tsx` 里 `describe` 的返回：
   - `id`：插件的标识，定了不要改，也不能和别的插件重复。油记按它关联车辆，记录的来源也存它。
   - `name`：油记里显示的名称，例如「Acme」。记加油时写「从 Acme 选」。
   - `capabilities`：只写你提供的。
5. 删掉不提供的动作，把 `CHANGELOG.md` 换成你自己的记录。

### 2. 在手机上调试

1. 用 Scripting 的开发服务器：电脑上在 Scripting 的工作区里运行 `npx scripting-cli start`（见 [Scripting 的文档](https://scriptingapp.github.io/guide/)），你的仓库放在工作区的 `scripts` 里，文件夹名和 `script.json` 的 `name` 一样。这样放时，`tsconfig.json` 用工作区里 Scripting 的类型定义：编辑器有提示，也可以用 `tsc -p tsconfig.json` 做类型检查。
2. 手机上的 Scripting 连上开发服务器。第一次先在手机上新建一个同名的空脚本，再在开发服务器页面里选它，代码就同步到手机上；之后改了文件会自动同步。
3. 在油记里添加你的插件，记一次加油、看一次车况，确认数据对。
4. 插件里 `console.log` 的内容，在运行开发服务器的终端里能看到。

### 3. 油记怎么调用插件

油记这样运行插件（契约第 2 节）：

```ts
Script.run({ name: 脚本名, queryParameters: { fuelnote: 1, action, ...参数 }, singleMode: true })
```

`index.tsx` 最后的 `run()` 就是入口：

- `Script.queryParameters` 里有 `fuelnote` 时，是油记在调用：按 `action` 处理，处理完一定 `Script.exit(结果)`。出错时也要 exit，所以最外层包一层 try/catch。不 exit，油记会一直等到超时。
- 没有 `fuelnote` 时，是用户自己打开的：显示插件自己的页面，例如连接、填凭据。页面用大标题，下面一行写版本（`navigationSubtitle` 写「版本 ${Script.metadata.version}」），用户同步了新版以后一打开就知道装的是哪一版。
- 成功返回 `{ ok: true, ... }`，失败返回 `{ ok: false, error, message }`；`message` 直接显示给用户（契约第 3 节）。

### 4. 每个动作

| 动作 | 示例里的写法 | 要点 |
|---|---|---|
| `describe` | `handle()` 的第一段 | 5 秒内返回；`version` 用 `Script.metadata.version`；提供 `info` 时用 `cards` 声明会给哪些卡片 |
| `status` | `status()` | 是否连接、凭据是否过期、打码后的账号；5 秒内返回，最好只读本地存的凭据 |
| `connect` | `ConnectPage` | 弹出插件自己的页面让用户登录或填凭据，关掉后返回状态 |
| `fuel` | `transactions()` | `from`～`to` 之间已完成的加油，退款、充值等不返回；金额、升数两位小数，坐标 WGS-84 |
| `vehicles`、`trips`、`vehicle_status` | `CAR`、`trips()`、`vehicleStatus()` | 行程按开始时间升序，里程用熄火时的里程表 |
| `service` | `serviceVisits()` | 只返回已结算的进店，新的在前；结算明细的各行加起来等于自付 |
| `fuel_price` | `EXAMPLE_PRICES` | 按传来的省、市给挂牌价，最好不用登录 |
| `info` | `info()` | 慢的分批给：没查完时带 `next` 和 `progress`；一行带 `place`（名称和坐标）时，油记长按这一行可以导航；只给参数 `cards` 里要的卡片（`wantedCards()`），卡片都要在 `describe` 的 `cards` 里声明 |

每个动作的参数、字段、单位和油记等多久，见契约第 4～9 节。

用户可以在油记的插件详情里关掉其中几项（油记 2.7 起），关掉的油记就不再调用。插件不用管这件事：`describe` 照常写提供的全部，每个动作照常回答（契约第 13 节）。

### 5. 出错和超时

- `error` 只用这几个值：`not_connected`（还没连接）、`expired`（凭据过期）、`network`（连不上）、`unsupported`（不提供这个动作）、`failed`（其他）。油记按它提示下一步，例如凭据过期时让用户到插件里重新连接。
- 不要只返回一部分数据：有一部分没取到时（例如两种付款方式里一种的凭据过期），整次返回错误。只列出一部分交易，用户会以为那次加油只付了一笔（契约第 3 节）。
- 油记等每个动作的时间见契约第 4 节的表。超时后油记不再等，晚回来的结果丢掉。慢的查询分批给，例如信息卡片。

### 6. 凭据和隐私

- 凭据（账号、Cookie、token）存在插件自己的钥匙串（`Keychain`）里。油记读不到，油记的备份里也没有。
- `status`、`connect` 返回的账号先打码，例如「示例账号 ****0001」。
- 公开的仓库里不要写密钥、Cookie、手机号、车牌和真实的交易。
- 要用手机的位置时，插件自己取（和油记共用 Scripting 的定位权限）。油记交给插件的位置最多到省、市。

### 7. 发布和更新

1. 改 `script.json` 的 `version`。加了能力、改了名称时一定要升：油记打开时发现版本变了，会重新 `describe`，用户不用移除再添加（契约第 12 节）。
2. 在 `CHANGELOG.md` 最上面加一节 `## x.y.z — 日期`。
3. 提交，打标签 `vx.y.z` 并推送。`.github/workflows/release.yml` 会核对版本号，把运行需要的文件打成 `<仓库名>.zip`（文件在根目录；`README.md`、`CHANGELOG.md`、`LICENSE` 和 `.github` 不放进去），用 `CHANGELOG.md` 里这一节发布 GitHub Release。
4. 别人用 `https://github.com/<你>/<仓库名>/releases/latest/download/<仓库名>.zip` 导入。要自动更新，在 `script.json` 里加上：

   ```json
   "remoteResource": {
     "url": "https://github.com/<你>/<仓库名>/releases/latest/download/<仓库名>.zip",
     "autoUpdateInterval": 86400
   }
   ```

   Scripting 每天检查一次更新。地址必须是你自己仓库的：对不上时发布流程会停下，不发布。本仓库没有加这一项，免得用模板建的插件被自动换回示例。
5. 私有的插件可以把 zip 放在 Secret Gist：拿到链接的人都能下载，链接不要外传。

### 8. 写 Scripting 脚本的几个坑

- 所有文件都按 TSX 转译：泛型箭头函数写成 `<T,>(v: T) => …` 或 function 声明，也不要写 `<T>value` 这样的类型断言。这类错误类型检查发现不了，到手机上才报 `Unexpected token`。
- 组件的可选属性不需要时整个不传（用条件展开）：传 `undefined` 会留出空位。
- `Script.queryParameters` 保留 JSON 的类型：`from`、`to` 是数字。
- `fetch` 的请求体用 `Data`。网址里的参数全部自己编码，包括 `|` 这样的符号；留着没编码的字符时，手机上会把整个网址再编码一次。

## 检查清单

- [ ] `fuelnote.json` 是 `{ "contract": 1 }`。
- [ ] `script.json` 的 `name` 定了不再改；`localizedNames.zh` 是「油记·<数据源>」。
- [ ] `describe` 的 `id` 唯一、不变；`capabilities` 只写提供的；提供 `trips`、`vehicle_status`、`service` 时也提供 `vehicles`。
- [ ] 每条路径都 `Script.exit`，出错时也是；最外层有 try/catch。
- [ ] 失败时 `error` 用契约里的值，`message` 写给用户看。
- [ ] 时间是毫秒时间戳，坐标是 WGS-84，金额、升数两位小数。
- [ ] 取不全时整次返回错误，不返回一部分。
- [ ] 账号打码；凭据只在钥匙串；仓库里没有密钥和真实数据。
- [ ] 加了能力、改了名称时升了 `version`。
- [ ] 自己的页面用大标题，下面一行写版本。

## 许可证

[MIT](LICENSE)
