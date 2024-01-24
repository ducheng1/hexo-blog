---
title: Gorm开启Logger
author: DuCheng
top: false
tags:
  - golang
categories:
  - golang
date: 2023-02-27 16:27:00
---

```go
package utils

import (
  "fmt"
  "github.com/spf13/viper"
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
  "gorm.io/gorm/logger"
  "log"
  "os"
  "time"
)

var DB *gorm.DB

var globalLogger = logger.New(
  log.New(os.Stdout, "\r\n", log.LstdFlags),
  logger.Config{
    SlowThreshold:             time.Second,
        // 自己切换logger的优先级，logger.Silent为不打印log
    LogLevel:                  logger.Info,
    IgnoreRecordNotFoundError: false,
    Colorful:                  true,
  },
)

func InitMySQL() {
  // 获取配置项
  username := viper.GetString("datasource.username")
  password := viper.GetString("datasource.password")
  host := viper.GetString("datasource.host")
  port := viper.GetString("datasource.port")
  database := viper.GetString("datasource.database")
  // 连接MySQL
  db, err := gorm.Open(mysql.New(mysql.Config{
    DSN: fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&parseTime=true&loc=Local", username, password, host, port, database),
  }), &gorm.Config{
    // 开启logger
    Logger: globalLogger,
  })
  db = db.Session(&gorm.Session{Logger: globalLogger})
  if err != nil {
    panic(fmt.Sprintf("MySQL连接失败：%v", err))
  }
  fmt.Println("MySQL连接成功")
  DB = db
}

func GetMySQL() *gorm.DB {
  return DB
}

```
