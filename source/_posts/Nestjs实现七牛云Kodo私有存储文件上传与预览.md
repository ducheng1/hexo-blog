---
title: Nestjs实现七牛云Kodo私有存储文件上传与预览
author: DuCheng
tags:
  - nestjs
categories:
  - 后端
date: 2024-03-28 16:04:00
---

# 安装七牛云SDK

```shell
pnpm i qiniu
```

# 新建qiniu模块

> 别在意angular的图标

![文件结构示意图](https://img.dcwedu.top/i/2024/04/01/660a610f9f8cb.png)

## qiniu.service.ts

由于是私有存储，在上传和下载时都需要生成对应的token，具体实现可以查看文档[七牛Node.jsSDK文件数据流上传](https://developer.qiniu.com/kodo/sdk/nodejs#form-upload-stream)。
**注意：这里的key尽量别直接使用文件名，七牛云的上传策略会按key拦截或覆盖文件上传**

```typescript
import { Injectable } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import * as qiniu from "qiniu";
import { Readable } from "stream";
import { UploadCallback } from "./types";
import { v4 as uuidV4 } from "uuid";

@Injectable()
export class QiniuService {
  constructor(private readonly configService: ConfigService) {}

  ak = this.configService.get<string>("qiniu.ak");
  sk = this.configService.get<string>("qiniu.sk");
  bucket = this.configService.get<string>("qiniu.bucket");

  // 生成凭证
  generateMac() {
    return new qiniu.auth.digest.Mac(this.ak, this.sk);
  }

  // 生成上传凭证
  generateUploadToken() {
    const putPolicy = new qiniu.rs.PutPolicy({
      scope: this.bucket,
      // 定义返回格式
      // 魔法变量：https://developer.qiniu.com/kodo/1235/vars#magicvar
      returnBody:
        '{"key":"$(key)","hash":"$(etag)","imageInfo":$(imageInfo),"fname":"$(fname)","fsize":$(fsize),"type":"$(mimeType)"}',
      // 如果需要将body返回给其他域名/服务则使用如下参数
      // returnUrl: ''
    });
    return putPolicy.uploadToken(this.generateMac());
  }

  // 上传
  async upload(file: Express.Multer.File): Promise<UploadCallback> {
    const uploader = new qiniu.form_up.FormUploader(
      new qiniu.conf.Config({
        zone: qiniu.zone.Zone_z0,
        useHttpsDomain: true,
      })
    );
    const putExtra = new qiniu.form_up.PutExtra();
    // buffer转换为可读流
    const stream = Readable.from(file.buffer);

    return new Promise((resolve, reject) => {
      uploader.putStream(
        this.generateUploadToken(),
        `${uuidV4()}.${file.originalname}`,
        stream,
        putExtra,
        (e, respBody, respInfo) => {
          if (e) {
            reject(e);
          }
          if (respInfo.statusCode === 200) {
            resolve(respBody);
          } else {
            reject(respBody);
          }
        }
      );
    });
  }

  // 生成预览链接 https://developer.qiniu.com/kodo/sdk/nodejs#private-get
  async preview(key: string) {
    const bucketManager = new qiniu.rs.BucketManager(
      this.generateMac(),
      new qiniu.conf.Config({
        zone: qiniu.zone.Zone_z0,
        useHttpsDomain: true,
      })
    );
    const domain = this.configService.get<string>("qiniu.domain");
    // 1小时后过期
    const deadline = Math.floor(Date.now() / 1000) + 3600;
    return bucketManager.privateDownloadUrl(domain, key, deadline);
  }
}
```

## qiniu.controller.ts

controller的实现就比较简单，上传使用`@nestjs/common`中的`UseInterceptors`拦截器注解和`UploadedFile`body参数注解即可，具体查看[Nestjs文件上传](https://docs.nestjs.com/techniques/file-upload)。

```typescript
import {
  Controller,
  Get,
  Post,
  Query,
  UploadedFile,
  UseInterceptors,
} from "@nestjs/common";
import { Express } from "express";
import { QiniuService } from "./qiniu.service";
import { FileInterceptor } from "@nestjs/platform-express";
import { response } from "../../utils/response";
import { UploadCallback } from "./types";

@Controller("/file")
export class QiniuController {
  constructor(private readonly qiniuService: QiniuService) {}

  @Post("/upload")
  @UseInterceptors(FileInterceptor("file"))
  async uploadFile(@UploadedFile() file: Express.Multer.File) {
    try {
      const res: UploadCallback = await this.qiniuService.upload(file);
      const url = await this.qiniuService.preview(res.key);
      return response.ok({
        ...res,
        url,
      });
    } catch (e) {
      return response.fail("上传失败");
    }
  }

  @Get("/preview")
  async getPreviewUrl(@Query("key") key: string) {
    try {
      const url = await this.qiniuService.preview(key);
      return response.ok(url);
    } catch (e) {
      return response.fail();
    }
  }
}
```

## qiniu.module.ts

最后将controller和service分别注册到module中即可。

```typescript
import { Module } from "@nestjs/common";
import { QiniuService } from "./qiniu.service";
import { ConfigModule } from "@nestjs/config";
import { QiniuController } from "./qiniu.controller";

@Module({
  imports: [ConfigModule],
  controllers: [QiniuController],
  providers: [QiniuService],
  exports: [QiniuService],
})
export class QiniuModule {}
```

## app.module.ts

别忘了在app中注册`QiniuModule`。

```typescript
import { MiddlewareConsumer, Module, NestModule } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { TenantMiddleware } from "./middleware/tenant.middleware";
import { ConfigModule, ConfigService } from "@nestjs/config";
import config from "./config";
import { RedisModule } from "./modules/redis/redis.module";
import { QiniuModule } from "./modules/qiniu/qiniu.module";

@Module({
  imports: [
    // env
    ConfigModule.forRoot({
      load: [config],
      envFilePath: [".env", ".env.dev", ".env.prod", ".env.local"],
    }),
    // database
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: "mysql",
        host: configService.get<string>("database.host"),
        port: configService.get<number>("database.port"),
        username: configService.get<string>("database.username"),
        password: configService.get<string>("database.password"),
        database: configService.get<string>("database.name"),
        autoLoadEntities: true,
        synchronize: true,
      }),
    }),
    // redis
    RedisModule,
    // qiniu
    QiniuModule,
  ],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer): any {
    // 多租户中间件
    consumer.apply(TenantMiddleware).forRoutes("");
  }
}
```

# ApiFox中的测试结果

![文件上传](https://img.dcwedu.top/i/2024/04/01/660a61913e600.png)

![预览/下载](https://img.dcwedu.top/i/2024/04/01/660a61b93a3db.png)
