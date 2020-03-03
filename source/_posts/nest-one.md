---
title: Nest.js项目入门实战(一)
tags: 
	- Nest.js
	- 入门实战
categories: nestjs
date: 2019/12/10
---

### 内容概要
- 该篇主要介绍Nest.js项目的基础知识以及项目初始搭建,nest cli的使用,以及连接数据库实现一个简单的增删改查接口的项目示例

### Nest.js简介
Nest是一套现代化的基于Node.js的强大的Web应用框架

Nest的核心概念是提供一种体系结构，它帮助开发人员实现层的最大分离，并在应用程序中增加抽象

>文档：https://nestjs.com/
>中文文档：https://docs.nestjs.cn/
>git地址：https://github.com/nestjs/nest

---

### 使用Nest CLI创建项目

安装CLI &nbsp;&nbsp;&nbsp; `npm i -g @nestjs/cli`
使用CLI新建项目 &nbsp;&nbsp;&nbsp; `nest new project-name`

运行后会默认创建以下内容

	CREATE /nest-test-peoject/.prettierrc (51 bytes)
	CREATE /nest-test-peoject/nest-cli.json (64 bytes)
	CREATE /nest-test-peoject/package.json (1700 bytes)
	CREATE /nest-test-peoject/README.md (3370 bytes)
	CREATE /nest-test-peoject/tsconfig.build.json (97 bytes)
	CREATE /nest-test-peoject/tsconfig.json (336 bytes)
	CREATE /nest-test-peoject/tslint.json (426 bytes)
	CREATE /nest-test-peoject/src/app.controller.spec.ts (617 bytes)
	CREATE /nest-test-peoject/src/app.controller.ts (274 bytes)
	CREATE /nest-test-peoject/src/app.module.ts (249 bytes)
	CREATE /nest-test-peoject/src/app.service.ts (142 bytes)
	CREATE /nest-test-peoject/src/main.ts (208 bytes)
	CREATE /nest-test-peoject/test/app.e2e-spec.ts (561 bytes)
	CREATE /nest-test-peoject/test/jest-e2e.json (183 bytes)


这里为我们创建了一个根模块AppModule &nbsp;&nbsp;&nbsp; `main.ts`为项目启动入口文件

###  Nest项目-基础知识

- 模块Module 
    用于将代码拆分为独立的和可重用的模块，例如有一个用户信息模块，然后将该用户模块的控制器和服务集合进来，最后需要将用户模块导入到根Module使用。
每个Nest应用都有一个根模块，通常是 AppModule。根模块提供了用来启动应用的引导机制，可以包含很多功能模块。
<br />

	**Module模块中@Module()装饰器接受一个对象**
	属性|描述
	---|:--:|
	providers|由nest注入器实例化的服务，可以在这个模块之间共享
	controller|存放创建的一组控制器
	imports|导入此模块中所需的提供程序的模块列表
	exports|导出这个模块可以其他模块享用providers里的服务

- 控制器Controller
	通俗来说就是路由Router，负责处理客户端传入的请求参数并向客户端返回响应数据，也可以理解是HTTP请求。nest.js提供许多http请求的装饰器，例如:方法参数装饰器@Param,@Body(),@Query;方法装饰器@Post(),@Get,@Put()等。

- 服务与依赖注入 Provider & Dependency injection
    在这里处理所有请求执行逻辑，在控制器中通过constructor函数以依赖注入的方式实现。Controller负责处理Request及产生Response，而数据库存取或是业务逻辑通常写在Service class里。
	提供程序是 Nest的一个基本概念。许多基本的 Nest类可能被视为提供者 - service,repository, factory 等等。 他们都可以通过 constructor 注入依赖关系。 这意味着对象可以彼此创建各种关系，并且“连接”对象实例的功能在很大程度上可以委托给 Nest运行时系统。 提供者只是一个用 @Injectable()装饰器注释的类。

- Middleware 中间件
	中间件是在路由处理程序之前调用的函数。中间件功能可以访问请求和响应对象，以及应用程序请求-响应周期中的下一个中间件功能。
	中间件函数可以执行以下任务:
	- 执行任何代码。
	- 对请求和响应对象进行更改。
	- 结束请求-响应周期。
	- 调用堆栈中的下一个中间件函数。
	- 如果当前的中间件函数没有结束请求-响应周期, 它必须调用 next() 将控制传递给下一个中间件函数。否则, 请求将被挂起。

- Exception filter 过滤器
	Nest 内置了全局异常处理层，一旦应用程序抛出异常，Nest 便会自动捕获这些异常并给出 JSON 形式的响应。
	未知类型的错误将会抛出以下错误
	```
	{
		"statusCode": 500,
		"message": "Internal server error"
	}```
- Pipe 管道
	管道是具有 @Injectable() 装饰器的类。管道应实现 PipeTransform 接口。
	管道有两个类型:
	转换：管道将输入数据转换为所需的数据输出
	验证：对输入数据进行验证，如果验证成功继续传递; 验证失败则抛出异常;
- Guard 守卫
	守卫可以做权限认证，如果你没有权限可以拒绝你访问这个路由，默认返回403错误
	守卫是用@Injectable()装饰器注释的类。应该实现CanActivate接口，具体代码在canActivate方法实现，返回一个布尔值，true就表示有权限，false抛出异常403错误。
	使用：
	直接@UseGuards()装饰器里面使用，作用当前控制器路由所有的请求参数 从 @nestjs/common 包导入。
	在全局注册使用内置实例方法useGlobalGuards，作用整个项目。
	```
	const app = await NestFactory.create(AppModule);
	app.useGlobalGuards(new Guard());
	```
- Interceptor 拦截器
	拦截器可以做很多功能，比如缓存处理，响应数据转换，异常捕获转换，响应超时跑错，打印请求响应日志。
	设置拦截器：为了设置拦截器, 我们使用从 @nestjs/common 包导入的 @UseInterceptors() 装饰器。与守卫一样, 拦截器可以是控制器范围内的, 方法范围内的或者全局范围内的。

<font color='red'> 以上内容nestjs文档都有详细描述具体可以[前往阅读](https://docs.nestjs.cn/)</font>

###  Nest项目-连接数据库

- Nest 与数据库无关，可以轻松地与任何 SQL 或 NoSQL 数据库集成。我们可以直接使用任何通用的 Node.js 数据库集成库或 ORM ，例如 [Sequelize (recipe)](https://sequelize.org/),[TypeORM](https://typeorm.io/#/) ，以在更高的抽象级别上进行操作。Nest 使用TypeORM是因为它是 TypeScript 中最成熟的对象关系映射器( ORM )。因为它是用 TypeScript 编写的，所以可以很好地与 Nest 框架集成。这里我们连接操作mysql数据库。
安装typeorm 和 mysql  `npm install --save @nestjs/typeorm typeorm mysql`
将`TypeOrmModule`导入`AppModule`。

```ts
	import { Module } from '@nestjs/common';
	import { TypeOrmModule } from '@nestjs/typeorm';

	@Module({
	imports: [
		TypeOrmModule.forRoot({
		type: 'mysql', // 数据库类型。你必须指定要使用的数据库引擎。该值可以是"mysql"，"postgres"，"mariadb"，"sqlite"，"cordova"，"nativescript"，"oracle"，"mssql"，"mongodb"，"sqljs"，"react-native"。此选项是必需的。
		name:'', // 连接名。 在使用 getConnection(name: string) 或 ConnectionManager.get(name: string)时候需要用到。不同连接的连接名称不能相同，它们都必须是唯一的。如果没有给出连接名称，那么它将被设置为"default"。
		host: 'localhost', // 数据库 host
		port: 3306, // 数据库端口。mysql 默认的端口是3306.
		username: 'root', // 数据库用户名
		password: 'root', // 数据库密码
		database: 'test', // 数据库名
		entities: [],//  要加载并用于此连接的实体。接受要加载的实体类和目录路径。目录支持 glob 模式。示例：entities: [ "entity/*.js", "modules/**/entity/*.js"]。
		synchronize: true,// 指示是否在每次应用程序启动时自动创建数据库架构。 请注意此选项，不要在生产环境中使用它，否则将丢失所有生产数据。但是此选项在调试和开发期间非常有用。
		charset:'utf8mb4', // 连接的字符集
		timezone: '+0800', // MySQL 服务器上配置的时区。这用于将服务器日期/时间值强制转换为 JavaScript Date 对象
		logging: true, // 指示是否启用日志记录。如果设置为true，则将启用查询和错误日志记录。
		connectTimeout:'10000', // 在连接到 MySQL 服务器期间发生超时之前的毫秒数。 （默认值：10000）
		...
		}),
	],
	})
	export class AppModule {}
```
forRoot() 方法接受与来自 TypeORM包的 createConnection() 相同的配置对象。另外，我们可以创建 ormconfig.json ，而不是将配置对象传递给 forRoot()。
一旦完成，TypeORM 连接和 EntityManager 对象就可以在整个项目中注入(不需要导入任何模块)

- 存储库模式
	TypeORM 支持存储库设计模式，因此每个实体都有自己的存储库。可以从数据库连接获得这些存储库。
	需要使用官方TypeORM文档中的  entity 实体

###  Nest项目-创建一个PhotoModule
上面已经在AppModule注入了TypeOrmModule并配置数据库信息，现在我们配置TypeOrmModule.forRoot()中的entities为(glob 模式匹配目录文件)

	entities:[__dirname + '/**/*.entity{.ts,.js}']

方便我们在文件中创建entity文件，其中`__dirname` 表示的是当前文件所在目录  `**`匹配任意数量的字符包括空字符和路径分隔符 `*`匹配任意数量的字符包括空字符
接下来使用`nest g mo photo`创建PhotoModule，该命令将在src目录下创建了photo文件目录并新建了PhotoModule文件自动注入到了AppModule

	nest g mo photo
	CREATE /src/photo/photo.module.ts (82 bytes)
	UPDATE /src/app.module.ts (1359 bytes)

创建完成后AppModule如下
```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { Connection } from 'typeorm';	
import { TypeOrmModule } from '@nestjs/typeorm';
import { PhotoModule } from './photo/photo.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
        type: 'mysql',
        host: '192.168.1.6',
        port: 3306,
        username: 'root',
        password: 'xxxx',
        database: 'xxxx',
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: true,
        timezone: '+0800',
        logging: true,
    }),
    PhotoModule,
  ],
  controllers: [AppController],
  providers: [
      AppService
    ],
})
export class AppModule {
	constructor(private readonly connection: Connection) { }
}
```
接下来我们依次创建PhotoController和PhotoService

	nest g co photo
	CREATE /src/photo/photo.controller.spec.ts (486 bytes)
	CREATE /src/photo/photo.controller.ts (99 bytes)
	UPDATE /src/photo/photo.module.ts (170 bytes)

	nest g s photo
	CREATE /src/photo/photo.service.spec.ts (453 bytes)
	CREATE /src/photo/photo.service.ts (89 bytes)
	UPDATE /src/photo/photo.module.ts (247 bytes)

一般的我们将entity文件放在他们的域中, 放在相应的模块目录中。当然也可以在src目录下新建entities目录将entity文件放在该目录下管理。

- 创建photo实体(photo.entity.ts)
实体是一个映射到数据库表（或使用 MongoDB 时的集合）的类。 你可以通过定义一个新类来创建一个实体，并用@Entity()来标记：
```ts
	import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

	@Entity('photo') // 实体名称 对应数据库表名
	export class Photo {
	@PrimaryGeneratedColumn({comment: '照片表'})
	id: number;

	@Column({ nullable: true, length: 30, comment: '照片名称' })
	name: string;

	@Column({ nullable: false, default:0, comment: '该条记录状态 0正常 1删除' })
	status: number;

	@Column({ nullable: true, type: 'datetime', comment: '该条记录创建时间' })
	create_time: string;

	@Column({ nullable: true, length: 255, comment: '照片资源链接' })
	url: string;
	}

	// orm配置中若synchronize为true则会在数据库自动执行创建表sql 创建photo表
	CREATE TABLE `photo` (
		`id` int(11) NOT NULL AUTO_INCREMENT COMMENT '照片表',
		`name` varchar(30) COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '照片名称',
		`status` int(11) NOT NULL DEFAULT '0' COMMENT '该条记录状态 0正常 1删除',
		`create_time` datetime DEFAULT NULL COMMENT '该条记录创建时间',
		`url` varchar(255) COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '照片资源链接',
		PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

<font color='red'> synchronize: true,指示是否在每次应用程序启动时自动创建数据库架构。 
请注意此选项，不要在生产环境中使用它，否则将丢失所有生产数据。但是此选项在调试和开发期间非常有用。</font>

[实体列属性等相关配置具体可参考typeOrm中实体](https://typeorm.io/#/entities)

###  Nest项目-PhotoController和PhotoService
接下来我们在PhotoController中创建几个方法用来接收客户端传入的请求参数并向客户端返回响应数据
```ts
import { Controller, Get, Post, Patch, Body, Query } from '@nestjs/common';
import { PhotoService } from './photo.service';

@Controller('photo')
export class PhotoController {
    // 宣告变量，型別为PhotoService
    // private readonly photoService:PhotoService;
    // 建立PhotoController实例，一并建立PhotoService实例
    // constructor() { this.photoService= new PhotoService(); } 
    constructor(private readonly photoService: PhotoService) { } // 将PhotoService注入到PhotoController

    /**
     * 创建相片记录
     * @param body
     */
    @Post() // 请求方法装饰器
    async createPhoto(@Body() body): Promise<any> {
        return await this.photoService.createPhoto(body);
    }

    /**
     * 删除相片记录
     * @param body
     */
    @Patch()
    async deletePhoto(@Body() body): Promise<any> {
        return await this.photoService.deletePhoto(body);
    }

    /**
     * 编辑相片记录内容
     * @param body
     */
    @Patch()
    async updatePhoto(@Body() body): Promise<any> {
        return await this.photoService.updatePhoto(body);
    }

    /**
     * 获取相片列表
     * @param query
     */
    @Get()
    async getPhotos(@Query() query): Promise<any> {
        return await this.photoService.getPhotos(query);
    }
}
```
对应的 我们需要在PhotoService中写上对应的方法处理所有请求执行逻辑

```ts
import { Injectable } from '@nestjs/common';
import { Photo } from '../entities/photo.entity'; // 引入实体文件
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, getRepository } from 'typeorm';

@Injectable()
export class PhotoService {
    constructor(
        @InjectRepository(Photo)
        private readonly photoRepository: Repository<Photo>, // Photo存储库
    ) { }

    /**
     * 创建相片记录
     * @param body
     */
    async createPhoto(body): Promise<any> {
        const newPhoto = new Photo();
        newPhoto.create_time = body.create_time;
        newPhoto.name = body.name;
        // newPhoto.status = body.status;
        newPhoto.url = body.url;
        const newOrderRes = await this.photoRepository.save(newPhoto);
        return { message: '创建成功', code: '01', content: newOrderRes };
    }

    /**
     * 删除相片记录
     * @param body
     */
    async deletePhoto(body): Promise<any> {
        // await this.photoRepository.delete(body.id);
        // await this.photoRepository.delete( { name:body.name});
        const needDelete = await this.photoRepository.findOne(body.id);
        await getRepository(Photo)
            .createQueryBuilder('Photo')
            .update(needDelete)
            .where('id = :id', { id: body.id })
            .set({ status: 1 })
            .execute();
            return { message: '删除成功', code: '01' };
    }

    /**
     * 编辑相片记录内容
     * @param body
     */
    async updatePhoto(body): Promise<any> {
        const needUpdate = await this.photoRepository.findOne(body.id);
        const updated = Object.assign(needUpdate, body);
        await this.photoRepository.update(body.id,updated);
    }

    /**
     * 获取相片列表
     * @param query
     */
    async getPhotos(query): Promise<any> {
        const qb = await getRepository(Photo)
            .createQueryBuilder('p')
            // .select('p')
            .select(['p.name','p.url'])
            .skip((query.page - 1) * query.size)
            .take(query.size)
            .orderBy({
                'p.create_time': 'DESC',
            });
        qb.where('1 = 1');
        if ('name' in query) {
            qb.andWhere('p.name = :name', { name: query.name });
        }
        qb.andWhere('p.status != 1 ');
        const list = await qb.getMany();
        const total = await qb.getCount();
        return { list, total, message: '请求列表成功', code: '01' };
		
    }
}
```
在photoModule中
```ts
import { Module } from '@nestjs/common';
import { PhotoController } from './photo.controller';
import { PhotoService } from './photo.service';
import { Photo } from '../entities/photo.entity';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [TypeOrmModule.forFeature([Photo])], // 使用forFeature()方法来定义应在当前范围中注册的存储库。
  controllers: [PhotoController],
  providers: [PhotoService]
})
export class PhotoModule {}
```
到这里一个PhotoModule模块就完成啦

使用`npm run start` 启动项目通过postman传入参数请求到 http://localhost/photo 就可以访问接口完成增删改查

### PhotoService中处理数据
在PhotoService中处理数据时用到了typeorm的api
- Repository ，Query Builder
	通过`this.photoRepository` `getRepository().createQueryBuilder()` `getConnection().createQueryBuilder()` 都可以操作想要操作的存储库

### 未完待续..

