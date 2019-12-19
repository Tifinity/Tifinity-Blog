---
title: "Go语言使用GoConvey框架进行测试"
date: 2019-12-18T11:15:59+08:00
draft: false
tags: ["Go"] 
categories: ["服务计算"]             
author: "Tifinity"  
---

# Go语言使用GoConvey框架进行测试

本周作业为在go-online上完成一个最小堆算法，在完成之后我使用GoConvey进行测试。

要想写出好的的代码，必须学会测试框架，对于golang，可以使用自带的`go test`测试，也可以使用其他框架如

GoConvey，GoStub，GoMock，Monkey，本次我学习使用GoConvey。

### 安装GoConvey

```
go get github.com/smartystreets/goconvey
```

需要等待较长的一段时间，然后查看`$GOPATH/src/github.com`目录下增加了`smartystreets`文件夹即可。

### 使用GoConvey

将作业代码复制到go工作空间中命名为`myheap.go`，并且在同一目录下创建`myheap_test.go`。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190924171326479.png)

先`go build`或者`go install`生成包，不需main主函数。因为要测试的函数需要有返回值，所以简单修改一下作业中的函数，将nodes返回。

myheap_test.go文件如下：

```go
package myheap

import (
    "testing"
    . "github.com/smartystreets/goconvey/convey"
)

func TestInit(t *testing.T) {
    Convey("test", t, func() {
		nodes := []Node{
			Node{3},
            Node{6},
            Node{9},
            Node{1},
            Node{2},
            Node{5},
            Node{8},
            Node{4},
            Node{7},
		}

		Convey("Test Init()", func(){
			So(Init(nodes), ShouldResemble, []Node{
				Node{1},
                Node{2},
                Node{5},
                Node{4},
                Node{3},
                Node{9},
                Node{8},
                Node{6},
                Node{7},
			})
        })

        nodes = Init(nodes)

        Convey("Test Push()", func(){
			So(Push(Node{0}, nodes), ShouldResemble, []Node{
				Node{0},
                Node{1},
                Node{5},
                Node{4},
                Node{2},
                Node{9},
                Node{8},
                Node{6},
                Node{7},
                Node{3},
			})
        })

        nodes = Push(Node{0}, nodes)

        Convey("Test Pop()", func(){
			So(Pop(nodes), ShouldResemble, []Node{
				Node{1},
				Node{2},
				Node{5},
				Node{4},
				Node{3},
				Node{9},
				Node{8},
				Node{6},
                Node{7},
			})
		})
	})
}
```



myheap.go文件如下：

```go
package myheap

import "fmt"

type Node struct {
	Value int
}

// 用于构建结构体切片为最小堆，需要调用down函数
func Init(nodes []Node)  []Node{
	for i := len(nodes)/2 - 1; i >= 0; i-- {
		down(nodes, i, len(nodes))
	}
	return nodes
}

// 需要down（下沉）的元素在切片中的索引为i，n为heap的长度，将该元素下沉到该元素对应的子树合适的位置，从而满足该子树为最小堆的要求
func down(nodes []Node, i, n int) {
	var parent int = i
	var child int = i*2+1
	var tmp int = nodes[i].Value
	for child < n {
		//取较小的孩子
		if (child < n-1) && (nodes[child].Value > nodes[child+1].Value) {
			child++
		}
		if tmp > nodes[child].Value {
			nodes[parent].Value = nodes[child].Value
			parent = child
			child = parent*2 + 1
		} else {
			break
		}
	}
	nodes[parent].Value = tmp
}

// 用于保证插入新元素(j为元素的索引,切片末尾插入，堆底插入)的结构体切片之后仍然是一个最小堆
func up(nodes []Node, j int) {
	var child int = j
	var parent int = (j-1)/2
	var tmp int = nodes[j].Value
	for parent >= 0 && child >= 1 {
		if tmp < nodes[parent].Value {
			nodes[child].Value = nodes[parent].Value
			child = parent
			parent = (child-1)/2
		} else {
			break
		}
	}
	nodes[child].Value = tmp
}

// 弹出最小元素，并保证弹出后的结构体切片仍然是一个最小堆，第一个返回值是弹出的节点的信息，第二个参数是Pop操作后得到的新的结构体切片
func Pop(nodes []Node)  []Node {
	//min := nodes[0]
    nodes[0].Value = nodes[len(nodes)-1].Value
    nodes = nodes[ :len(nodes)-1]
    down(nodes,0,len(nodes)-1)
    return nodes
}

// 保证插入新元素时，结构体切片仍然是一个最小堆，需要调用up函数
func Push(node Node, nodes []Node) []Node {
	nodes = append(nodes, node)
    up(nodes, len(nodes)-1)
    return nodes
}

// 移除切片中指定索引的元素，保证移除后结构体切片仍然是一个最小堆
func Remove(nodes []Node, node Node) []Node {
	for i := 0; i < len(nodes); i++ {
        if node.Value == nodes[i].Value {
            nodes[i].Value = nodes[len(nodes)-1].Value
            nodes = nodes[:len(nodes)-1]
            down(nodes, i, len(nodes)-1)
        }
    }
    return nodes
}

//用于打印堆
func PrintList(nodes []Node) {
    for _, ele := range nodes {
        fmt.Printf("%d ", ele.Value)
    }
    fmt.Printf("\n")
}
```

进入该目录并执行`go test`即可，与原生命令相同，然后可以看到输出：

通过：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190924171228540.png)



没通过：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190924171243481.png)

上面有具体信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190924171311145.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

