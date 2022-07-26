
定义：快速的，节省磁盘空间的包管理工具。它有以下特点：
## 节约磁盘空间并提升安装速度 

1. 如果你用到了某依赖项的不同版本，只会将不同版本间有差异的文件添加到仓库。 例如，如果某个包有100个文件，而它的新版本只改变了其中1个文件。那么 pnpm update 时只会向存储中心额外添加1个新文件，而不会因为仅仅一个文件的改变复制整新版本包的内容。（**基于内容版本的增量添加**）
1. 所有文件都会存储在硬盘上的某一位置。 当软件包被被安装时，包里的文件会硬链接到这一位置，而不会占用额外的磁盘空间。 这允许你跨项目地共享同一版本的依赖。(**内容寻址储存CAS**)
## 非扁平化的node_modules文件夹

1. 使用 npm 或 Yarn Classic 安装依赖项时，所有包都被提升到模块目录的根目录。 因此，项目可以访问到未被添加进当前项目的依赖。（npm 或 Yarn Classic  的node_modules文件是平铺的，可能会存在**幽灵引用**和**依赖分身**的问题）
## 包管理工具现状：
npm的早期阶段，采用的是**嵌套的node_modules结构，**直接依赖会平铺在根目录下，子依赖会嵌套在直接依赖的node_modules下 ，例如：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29734034/1658819250035-a5ac7247-2655-4af9-81e0-269d0c5b6b3b.png#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=166&id=u9c7c6b79&margin=%5Bobject%20Object%5D&name=image.png&originHeight=169&originWidth=663&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16164&status=done&style=none&taskId=ufc6efd38-4819-40e5-a3ac-fa74ab8e891&title=&width=650.5)
如果 D 也依赖 B@1.0，会生成如下的嵌套结构：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29734034/1658819332980-8ac35b01-75a3-4f93-9349-ba8e14f5fec7.png#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=216&id=u8cdb4c31&margin=%5Bobject%20Object%5D&name=image.png&originHeight=225&originWidth=678&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22939&status=done&style=none&taskId=u60986907-9dc6-4217-9082-135edeac95e&title=&width=651)
### 依赖地狱
在业务场景中,依赖的增多，冗余的包也会变多。依赖嵌套的深度会变得非常的可怕。这个叫做**依赖地狱**。
npm在后续的v3版本中修改了依赖的层级，将所有的子依赖提升，尽量的扁平化node_modules结构。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29734034/1658819681029-008e1288-8f81-482e-9e77-8d14b558336c.png#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=150&id=u5ab45529&margin=%5Bobject%20Object%5D&name=image.png&originHeight=150&originWidth=656&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14021&status=done&style=none&taskId=ubd01db8d-893e-49fa-943b-0b62bf076ad&title=&width=655)

A、C的依赖被平铺，B因为版本号的问题，没能被提升，任然被嵌套在C的目录中。
看上去这个方法解决了包的冗余问题，依赖的层级也没有变的更深，解决了依赖地狱的问题，但是也出现了新的问题：**幽灵依赖。**
### 幽灵依赖
幽灵依赖是指在package.json中未能定义的依赖，在项目中可以被引用。
在上面的例子中，由于子依赖的提升，子依赖 **B@1.0.0 **可以在项目中被引用，如果某天某个版本的 A 依赖不再依赖 B 或者 B 的版本发生了变化，那么就会造成依赖缺失或兼容性问题。
### 不确定性
依赖的安装顺序会决定node_modules的结构，导致**不确定性**,例如：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29734034/1658820702788-5d0a69f1-0791-4bab-ba30-1666d6184a09.png#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=147&id=u4cb57a05&margin=%5Bobject%20Object%5D&name=image.png&originHeight=150&originWidth=686&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14220&status=done&style=none&taskId=ue53f09bb-b446-4ecb-ad18-8938cb1c97b&title=&width=670)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29734034/1658820710299-b4eb9622-cda3-4ec1-9f18-a739928d3563.png#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=139&id=uaffd4d55&margin=%5Bobject%20Object%5D&name=image.png&originHeight=144&originWidth=693&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14213&status=done&style=none&taskId=u92d9fa93-38a3-494d-820d-998b431836d&title=&width=669.5)
上图2种情况取决于node_modules的安装顺序。
所以在开发场景中，一旦有package.json的变更，本地需要删除node_modules ，重新install。否测可能会出现开发环境和生产环境的node_modules结构不同，代码无法运行。
### 依赖分身
假设基于上面例子的场景中，继续安装依赖D和E,这2个模块同时依赖** B@2.0.0**，此时：

- A 和 D 依赖 B@1.0
- C 和 E 依赖 B@2.0

结构如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29734034/1658821144689-56fe2afd-c895-48e5-8bc9-c3f87e27f1e0.png#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=218&id=u0adc8a76&margin=%5Bobject%20Object%5D&name=image.png&originHeight=223&originWidth=690&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23039&status=done&style=none&taskId=u6d373611-8395-4658-bf57-837ac176c39&title=&width=674)
B@2.0.0会被安装多次，重复安装的B就是**依赖分身**。
npm v3 看起来解决了嵌套地狱的问题，但是又产生了其他问题：安装慢（**依赖分身**）, 不确定性（**幽灵依赖，安装顺序问题**）。
于是，yarn登场。
### yarn
yarn在npm v3期间推出，它的出现主要解决v3的部份副作用问题：

- 提升安装速度：
npm: 串行下载，逐一安装。 （npm v5版本添加支持）
yarn: 并行下载，建立缓存，如果本地有缓存依赖，直接脱机安装。
- locakfile，解决不确定性：lockfile 里记录了依赖和子依赖的版本，获取版本，和验证一致性的hash。（npm v5版本添加支持）
### npm 和 yarn的弊端：
虽然都扁平化了node_modules结构，但是没有解决**幽灵依赖**和**依赖分身**的问题。

## 如何解决？
pnpm的推出解决了这几个问题。
### 内容寻址存储 CAS
与扁平化node_modules 不同，pnpm采用的另外一套依赖管理策略，内容寻址储存。
pnpm每次安装的依赖将会全局安装在本机的store中，依赖的每个版本只会被安装一次。
在node_modules中，将会通过**硬链接**和**软连接**建立依赖关系。

- **硬链接 Hard link**：硬链接可以理解为**源文件的副本**，项目里安装的其实是副本，它使得用户可以通过路径引用查找到全局 store 中的源文件，而且这个副本根本不占任何空间。同时，pnpm 会在全局 store 里存储硬链接，不同的项目可以从全局 store 寻找到同一个依赖，大大地节省了磁盘空间。
- **符号链接 Symbolic link**：也叫软连接，可以理解为**快捷方式**，pnpm 可以通过它找到对应磁盘目录下的依赖地址。

例子：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29734034/1658822508730-371c5e94-d8a5-432b-9f00-4803b2b35e12.png#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=448&id=ue230b2c9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=412&originWidth=709&originalType=binary&ratio=1&rotation=0&showTitle=false&size=57822&status=done&style=none&taskId=ub03dff27-2c9e-45a4-b7e7-01a4ba09504&title=&width=771.5)
<store>开头的是硬链接，指向store中安装的依赖。
官方的示意图：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29734034/1658822622607-7a09ef93-889b-4384-b432-17e674025b2d.png#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=696&id=ufa0aa7c9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1392&originWidth=2920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=701779&status=done&style=none&taskId=u867db280-a067-4250-97d2-91fdf1c2bab&title=&width=1460)

这套机制解决了幽灵依赖和依赖分身的问题。

1. 幽灵依赖问题：只有直接依赖会平铺在 node_modules 下，子依赖不会被提升，不会产生幽灵依赖。
1. 依赖分身问题：相同的依赖只会在全局 store 中安装一次。项目中的都是源文件的副本，几乎不占用任何空间，没有了依赖分身。

包管理器性能对比：
![alotta-files.svg](https://cdn.nlark.com/yuque/0/2022/svg/29734034/1658823028174-710b6c98-1b94-4074-bcb9-d1e634129f5a.svg#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=drop&height=793&id=uf27fd6a2&margin=%5Bobject%20Object%5D&name=alotta-files.svg&originHeight=150&originWidth=141&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7235&status=done&style=none&taskId=u1fbed590-71d3-4eca-9ec4-bf9d018e090&title=&width=745)
在功能支持上面：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29734034/1658823475742-d7a45556-a861-4215-a822-de6a2fc16a4d.png#clientId=ua3045b87-6df7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=593&id=u3112332c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=723&originWidth=882&originalType=binary&ratio=1&rotation=0&showTitle=false&size=92876&status=done&style=none&taskId=u883f0faf-f627-48d6-8213-890289bc2b3&title=&width=724)
弊端：

1. 由于 pnpm 创建的 node_modules 依赖软链接，因此在不支持软链接的环境中，无法使用 pnpm，比如 Electron 应用。
1. 因为依赖源文件是安装在 store 中，调试依赖或 patch-package 给依赖打补丁也不太方便，可能会影响其他项目。
## 总结：

- 不同的 node_modules 结构，有嵌套，扁平，不同的结构也伴随着兼容与安全问题。
- 不同的依赖存储方式来节约磁盘空间，提升安装速度。
- 每种管理器都伴随新的工具和命令，不同程度的可配置性和扩展性，影响开发者体验。
- 这些包管理器也对 monorepo 有不同程度的支持，会直接影响项目的可维护性和速度。



## 

