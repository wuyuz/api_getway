## Konga 的使用

​	Kong的功能已经足够强大，但很可惜，它的免费版是没有提供GUI界面的，通过CLI使用始终相对不便。于是，网上一些有志之士开发了相关的GUI界面。其中，做得比较好的有：

- Kong dashboard，[https://github.com/PGBI/kong-dashboard](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FPGBI%2Fkong-dashboard)，已经长期没有更新，只支持到Kong的0.9版本，目前Kong的最新版本是1.2.x
- Konga，[https://github.com/pantsel/konga](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fpantsel%2Fkonga)，功能完善，持续更新中



#### centos中安装

Konga是使用NodeJS开发的项目，因此需要下载源码并安装。受网速影响，整个安装配置过程大，所以比较耗时。由于Konga依赖比较多，安装起来比较麻烦，尤其安装在CentOS等其他发行版的Linux更加复杂，可以把已经安装好的环境克隆过去，然后试试以下命令。

```
git clone https://github.com/pantsel/konga.git
cd konga
npm install konga
```

- 安装完后，我们需要修改写配置文件，因为konga默认是使用mysql作为数据库

  ```shell
  # 示例配置位置，进入konga的安装目录，找到config进入，修改
  /config/local_example.js
   
  # 拷贝一份
  cd ./config/
  cp local_example.js ./local.js
   
  # 配置默认数据库
  vi ./local.js
  models: {
      connection: process.env.DB_ADAPTER || 'localDiskDb',
  },
  # 改成
  models: {
      connection: process.env.DB_ADAPTER || 'mysql', # 这里可以用‘mysql’，‘mongo’，‘sqlserver’，‘postgres’
  },
  # 保存
   
  # 修改数据库默认配置
  vi connections.js
  
  # 改成
  postgres: {
      adapter: 'sails-postgresql',
      url: process.env.DB_URI,
      host: process.env.DB_HOST || 'localhost',
      user:  process.env.DB_USER || 'kong',
      password: process.env.DB_PASSWORD || '123',
      port: process.env.DB_PORT || 5432,
      database: process.env.DB_DATABASE ||'kong',
      // schema: process.env.DB_PG_SCHEMA ||'public',
      poolSize: process.env.DB_POOLSIZE || 10,
      ssl: process.env.DB_SSL ? true : false // If set, assume it's true
    },
  # 保存
   
  # 启动
  cd ../
  npm start local.js
  ```

- 成功启动后，我们可以 使用浏览器访问：localhost:1338，端口可以在local.js改。这里需要注册一个管理员账号：如，wanglx -- xxxx123  ；然后绑定Admin API的url绑定我们的kong

  ```
  1、如果我们注册好账号后，出现空白页，在kanga的项目目录下执行：npm run bower-deps
  	In some cases when running npm install, the bower dependencies are not installed properly. You will need to cd into your project's root directory and install them manually by typing
  
  2. Can't add/edit some plugin properties.
  	When a plugin property is an array, the input is handled by a chip component. You will need to press enter after every value you type in so that the component assigns it to an array index. See issue #48 for reference.
  
  3. EACCES permission denied, mkdir '/kongadata/'.
      If you see this error while trying to run Konga, it means that konga has no write permissions to it's default data dir /kongadata. You will just have to define the storage path yourself to a directory Konga will have access permissions via the env var STORAGE_PATH.
  
  4. The hook grunt is taking too long to load
  	The default timeout for the sails hooks to load is 60000. In some cases, depending on the memory the host machine has available, startup tasks like code minification and uglyfication may take longer to complete. You can fix that by setting then env var KONGA_HOOK_TIMEOUT to something greater than 60000, like 120000.
  ```

  ![1583751767361](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1583751767361.png)

