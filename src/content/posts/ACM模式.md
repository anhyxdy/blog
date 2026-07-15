---
title: ACM模式
published: 2026-07-15
description: ''
image: ''
tags: [算法]
category: '技术笔记'
draft: false 
lang: ''
---

# ACM模式：读取至EOF模板笔记

## 核心关键点

ACM OJ 题目**测试用例组数不固定，需要持续读取输入直到 EOF（输入结束标记），不能固定循环次数读取**

### 基础模板：fmt.Scan + EOF 判断

```go
package main

import (
    "fmt"
    "io"
)

func add(a, b int) int {
    return a + b
}

func main() {
    var a, b int
    // 循环读取，直到读到EOF结束
    for {
        _, err := fmt.Scan(&a, &b)
        if err != nil {
            // 判断是否为正常输入结束
            if err == io.EOF {
                break
            }
            // 非EOF错误，抛出异常
            panic(err)
        }
        result := add(a, b)
        fmt.Println(result)
    }
}
```

### 重要注意事项

1. 题目结构参考
   - 题目示例：查看输入格式、每行数据结构、字段数量和类型
   - 自测用例：多行连续输入，无固定行数，模拟真实OJ评测输入
   - 完整流程：导包 → 循环读取并解析输入 → 算法计算 → 逐行输出结果
2. 错误判断逻辑
   - `fmt.Scan` 返回 `err`：读到末尾时错误为 `io.EOF` → 正常退出循环
   - 其他错误（格式错误等）用 `panic` 抛出，方便调试
3. 大数据版本：bufio快读 + EOF循环
   ```go
   package main

   import (
       "bufio"
       "fmt"
       "os"
       "strconv"
   )

   func main() {
       scanner := bufio.NewScanner(os.Stdin)
       scanner.Split(bufio.ScanWords)
       scanner.Buffer(make([]byte, 1024*1024), 1024*1024)

       readInt := func() int {
           scanner.Scan()
           num, _ := strconv.Atoi(scanner.Text())
           return num
       }

       for scanner.Scan() {
           a := readInt()
           b := readInt()
           fmt.Println(a + b)
       }
       // 检查扫描器是否存在非EOF错误
       if err := scanner.Err(); err != nil {
           panic(err)
       }
   }
   ```
4. 易错坑
   - ❌ 不要直接写固定n次循环（`for i:=0;i<n;i++`），无法适配未知组数用例
   - ✅ 用无限循环 + EOF检测，保证读取全部输入
   - ✅ 保证输出格式和题目要求完全一致（换行、空格格式）

### EOF含义

EOF = End of File，代表输入流结束，在线评测系统会一次性输入全部测试数据并以EOF结尾；Windows控制台可按 `Ctrl+Z`，Mac/Linux控制台可按 `Ctrl+D` 手动触发EOF用于本地自测。
