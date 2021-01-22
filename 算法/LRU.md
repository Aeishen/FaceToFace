# LRU(最近最少使用算法)

### 思想

`LRU`算法根据数据的历史访问记录来进行淘汰数据，如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很小。所以，当指定的空间已存满数据时，应当把最久没有被访问到的数据淘汰。

### 算法

#### 1.数组(非Go语言)

用一个数组来存储数据，给每一个数据项标记一个访问时间戳（或者自增数字），每次插入新数据项的时候，先把数组中存在的数据项的时间戳（数字）自增，并将新数据项的时间戳（数字）置为0并插入到数组中。每次访问数组中的数据项的时候，将被访问的数据项的时间戳（数字）置为0。当数组空间已满时，将时间戳（数字）最大的数据项淘汰。

> 数组缺点：
>
> ①需要不断维护每个数据项的时间戳（标记）
>
> ②每一次操作（插入、删除以及访问）缓存都需要遍历一遍数组，时间复杂度为O(n).

#### 2.切片(Go语言)

用一个切片来存储数据，新数据项添加在切片末尾；原有切片中的数据项被访问时，将被访问的数据项从切片中移除，再添加到切片末尾；当达到最大容量限制时，直接移除切片的第一个数据项。

```go
// 用切片实现LRU
package LRU

import (
	"fmt"
	"sync"
)

type sliceLRU struct{
	lock *sync.RWMutex
	maxNum int
	data []interface{}
}

var SliceLRUCache *sliceLRU

func InitSliceLRU(baseSize int){
	SliceLRUCache = new(sliceLRU)
	SliceLRUCache.data = make([]interface{}, 0)
	SliceLRUCache.maxNum = baseSize
	SliceLRUCache.lock = new(sync.RWMutex)
}


// 访问某个元素
func (l *sliceLRU) Offer(v interface{}){
	isFind, index := l.isExist(v)
    if isFind {
		l.update(v,index)
	} else {
		if l.isFull() {
			l.clear()
		}
		l.insert(v)
	}
}


// 缓存打印
func (l *sliceLRU) Print(){
	l.lock.RLock()
	defer l.lock.RUnlock()
	fmt.Printf("%v\n",l.data)
	return
}


// 缓存中添加新元素
func (l *sliceLRU) insert(v interface{}){
	l.lock.Lock()
	defer l.lock.Unlock()
	l.data = append(l.data, v)
}


// 清理缓存中最就没使用的元素
func (l *sliceLRU) clear(){
	l.lock.Lock()
	defer l.lock.Unlock()
	l.data = l.data[1:]
}


// 更新缓存中元素的位置
func (l *sliceLRU) update(v interface{}, index int) {
	l.lock.Lock()
	defer l.lock.Unlock()
	l.data = append(l.data[:index], l.data[index + 1:]...)
	l.data = append(l.data, v)
	return
}


// 查询缓存是否已满
func (l *sliceLRU) isFull() bool {
	l.lock.RLock()
	defer l.lock.RUnlock()
	return len(l.data) >= l.maxNum
}


// 查询元素是否已经存在于缓存中
func (l *sliceLRU) isExist(v interface{}) (isFind bool, index int){
	l.lock.RLock()
	defer l.lock.RUnlock()
	for i, item := range l.data {
		if item == v {
			index = i
			isFind = true
			return
		}
	}
	return
}
```

> 切片缺点：
>
> ①每一次访问缓存都需要遍历一遍切片，时间复杂度为O(n).

#### 3.链表

用一个链表来存储数据，新数据项添加在链表末尾；原有链表中的数据项被访问时，将被访问的数据项从链表中移除，再插入到链表末尾；当达到最大容量限制时，直接移除链表的第一个数据项。（跟切片简直一模一样的）

```go
// 用链表实现LRU
```



> 链表缺点：
>
> ①每一次访问缓存都需要遍历一遍链表，时间复杂度为O(n).