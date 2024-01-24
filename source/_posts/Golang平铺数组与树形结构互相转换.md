---
title: Golang平铺数组与树形结构互相转换
author: DuCheng
top: false
tags:
  - golang
  - 算法
categories:
  - golang
date: 2023-02-27 16:11:00
cover:
password:
---

# 平铺数组转换为树形结构

```go
  type T struct {
    model.DictGenre
    Children *[]*T `json:"children"`
  }

  treeList := func() []T {
    var treeList []T
    var flatPtr []T
    for _, i := range flat {
      t := T{}
      t.Pid = i.Pid
      t.ID = i.ID
      t.NameZh = i.NameZh
      t.NameEn = i.NameEn
      t.AliasZh = i.AliasZh
      t.AliasEn = i.AliasEn
      t.CreateTime = i.CreateTime
      t.UpdateTime = i.UpdateTime
      t.Children = nil
      flatPtr = append(flatPtr, t)
    }
    for m := range flatPtr {
      for n := range flatPtr {
        if flatPtr[m].ID == flatPtr[n].Pid {
          if flatPtr[m].Children == nil {
            flatPtr[m].Children = &[]*T{}
          }
          *(flatPtr[m].Children) = append(*(flatPtr[m].Children), &flatPtr[n])
        }
      }
    }
    for _, j := range flatPtr {
      if j.Pid == 0 {
        treeList = append(treeList, j)
      }
    }
    return treeList
  }()
```

# 树形结构转换为平铺数组

```go
  flat := func() []model.DictGenre {
    var flat []model.DictGenre
    for _, v := range genres {
      u := model.DictGenre{}
      u.Pid = v.Pid
      u.ID = v.ID
      u.NameZh = v.NameZh
      u.NameEn = v.NameEn
      u.AliasZh = v.AliasZh
      u.AliasEn = v.AliasEn
      u.CreateTime = v.CreateTime
      u.UpdateTime = v.UpdateTime
      flat = append(flat, u)
    }
    return flat
  }()
```
