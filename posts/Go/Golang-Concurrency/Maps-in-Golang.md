---
title: "Golang中的Map"
description: "Golang中的Map"
date: 2022-09-20 18:10:00
slug: Maps-in-Golang
image:
categories:
    - Go
tags: ["Go"]
---

### Go内建的Map

```go
map[K]V
```

其中key必须为可比较的。

在 Go 语言中，bool、整数、浮点数、复数、字符串、指针、Channel、接口都是可比较的，包含可比较元素的 struct 和数组，这俩也是可比较的，而 slice、map、函数值都是不可比较的。

map中的key通常选择内建的基本类型例如整数、字符串。

### 坑：使用struct作为map的key

```go 
// 使用struct作为map的key
type mapKey struct {
	key int
}

func main() {
	var m = make(map[mapKey]string)
	var key = mapKey{10}
	m[key] = "Hello"
	fmt.Printf("map[mapKey]=%s\n", m[key]) // map[mapKey]=Hello
	// 修改key
	key.key = 20
	fmt.Printf("再次查询map[mapKey]=%s\n", m[key]) // 再次查询map[mapKey]=
}
```

### map[key]函数返回结果可以是一个值，也可以是两个值

在 Go 中，map[key]函数返回结果可以是一个值，也可以是两个值。

```go
// 在 Go 中，map[key]函数返回结果可以是一个值，也可以是两个值
func main() {
	var m = make(map[string]int)
	m["a"] = 0
	fmt.Printf("a=%d, b=%d\n", m["a"], m["b"]) // a=0, b=0
	av, aexisted := m["a"]
	bv, bexisted := m["b"]
	fmt.Printf("a=%d, existed:%t\nb=%d,existed:%t\n", av, aexisted, bv, bexisted) 
    // a=0, existed:true b=0,existed:false
}
```

### map常见错误

**常见错误1-未初始化**

```go
var m map[int]int
// m := make(map[int]int) // 解决
m[0] = 100
fmt.Println(m) // panic: assignment to entry in nil map
```

从 一个nil的map中获取值不会panic。

```go
var m1 map[int]int
fmt.Println(m1)      // map[]
fmt.Println(m1[100]) // 0
```

map作为struct字段很容易忘记初始化。

```go
func main() {
   var c Counter
    c.Website = "baidu.com"
    c.PageCounters = make(map[string]int) // 初始化
    c.PageCounters["/"]++                 // panic: assignment to entry in nil map
    fmt.Println(c)                        // {baidu.com 0001-01-01 00:00:00 +0000 UTC map[/:1]} 
}
type Counter struct {
	Website      string
	Start        time.Time
	PageCounters map[string]int
}
```

**常见错误2-map 并发读写导致panic**

```go
// map 并发读写导致panic的例子
func main() {
	var m = make(map[int]int, 10) // 初始化一个map
	go func() {
		for {
			m[1] = 1 // 设置key
		}
	}()

	go func() {
		_ = m[2] // 读取
	}()
	select {}
}

/*
WARNING: DATA RACE
Read at 0x00c000020030 by goroutine 7:
  runtime.mapaccess1_fast64()
  main.main.func2()
Previous write at 0x00c000020030 by goroutine 6:
  runtime.mapassign_fast64()
*/
```

### 实现线程安全的 map类型

- 加读写锁：扩展 map，支持并发读写
- 分片加锁：更高效的并发 map

#### 加读写锁：扩展 map，支持并发读写

```go
// 读写锁实现线程安全的map[int]int类型

// 一个读写锁保护的线程安全的map
type RWMap struct {
	sync.RWMutex
	m map[int]int
}

// 实例化 新建一个RWMap
func NewRWMap(n int) *RWMap {
	return &RWMap{
		m: make(map[int]int, n),
	}
}

// 从map中获取一个值
func (m *RWMap) Get(k int) (int, bool) {
	m.RLock()
	defer m.Unlock()
	v, existed := m.m[k]
	return v, existed
}

// 设置键值对
func (m *RWMap) Set(k int, v int) {
	m.Lock()
	defer m.Unlock()
	m.m[k] = v
}

// 删除一个键
func (m *RWMap) Delete(k int) {
	m.Lock()
	defer m.Unlock()
	delete(m.m, k)
}

// 获取map长度
func (m *RWMap) Len() int {
	m.Lock()
	defer m.Unlock()
	return len(m.m)
}

// 遍历map
func (m *RWMap) Each(f func(k, v int) bool) {
	// 遍历期间一直持有读锁
	m.RLock()
	defer m.RUnlock()
	for k, v := range m.m {
		if !f(k, v) {
			return
		}
	}
}
```

#### 分片加锁：更高效的并发 map

尽量减少锁的粒度和锁的持有时间，减少锁的粒度常用的方法就是分片（Shard），将一把锁分成几把锁，每个锁控制一个分片。Go 比较知名的分片并发 map 的实现是`orcaman/concurrent-map`（https://github.com/orcaman/concurrent-map/blob/master/concurrent_map.go）

```go
var SHARD_COUNT = 32

// A "thread" safe map of type string:Anything.
// To avoid lock bottlenecks this map is dived to several (SHARD_COUNT) map shards.
// 分成SHARD_COUNT个分片的map
type ConcurrentMap[K comparable, V any] struct {
	shards   []*ConcurrentMapShared[K, V]
	sharding func(key K) uint32
}

// A "thread" safe string to anything map.
// 通过RWMutex保护的线程安全的分片，包含一个map
type ConcurrentMapShared[K comparable, V any] struct {
	items        map[K]V
	sync.RWMutex // Read Write mutex, guards access to internal map.
}

func create[K comparable, V any](sharding func(key K) uint32) ConcurrentMap[K, V] {
	m := ConcurrentMap[K, V]{
		sharding: sharding,
		shards:   make([]*ConcurrentMapShared[K, V], SHARD_COUNT),
	}
	for i := 0; i < SHARD_COUNT; i++ {
		m.shards[i] = &ConcurrentMapShared[K, V]{items: make(map[K]V)}
	}
	return m
}

// Creates a new concurrent map.
// 创建一个并发map,使用了泛型
func New[V any]() ConcurrentMap[string, V] {
	return create[string, V](fnv32)
}
func fnv32(key string) uint32 {
	hash := uint32(2166136261)
	const prime32 = uint32(16777619)
	keyLength := len(key)
	for i := 0; i < keyLength; i++ {
		hash *= prime32
		hash ^= uint32(key[i])
	}
	return hash
}

// GetShard returns shard under given key
// GetShard 是一个关键的方法，能够根据 key 计算出分片索引

func (m ConcurrentMap[K, V]) GetShard(key K) *ConcurrentMapShared[K, V] {
	return m.shards[uint(m.sharding(key))%uint(SHARD_COUNT)]
}

// Sets the given value under the specified key.
func (m ConcurrentMap[K, V]) Set(key K, value V) {
	// Get map shard.
	// 根据key计算出对应的分片
	shard := m.GetShard(key)
	shard.Lock()
	shard.items[key] = value
	shard.Unlock()
}

// Get retrieves an element from map under given key.
func (m ConcurrentMap[K, V]) Get(key K) (V, bool) {
	// Get shard
	// 根据key计算出对应的分片
	shard := m.GetShard(key)
	shard.RLock()
	// Get item from shard.
	// 从这个分片中获取对应的值
	val, ok := shard.items[key]
	shard.RUnlock()
	return val, ok
}
```

在使用并发 map 的过程中，加锁和分片加锁这两种方案都比较常用。

如果是追求更高的性能，显然是分片加锁更好，因为它可以降低锁的粒度，进而提高访问此 map 对象的吞吐。

如果并发性能要求不是那么高的场景，简单加锁方式更简单。

## sync.Map的使用

`sync.Map`是生产环境中很少使用的同步原语。

源码：sync/map.go

```go
type Map struct {
	mu Mutex
    // 基本上可以看成一个安全只读的map
    // 它包含的元素其实也是通过原子操作更新的，但是已删除的 entry就需要加锁操作了
    read atomic.Value // readOnly
    // 包含需要加锁才能访问的元素
    // 包含所有在read字段中但未被expunged(删除)的元素以及新加的元素
    dirty map[any]*entry
    // 记录从read中读取miss的次数，一旦miss数和dirty长度一样了，就会把dirty提升为read
    misses int
}
// readOnly is an immutable struct stored atomically in the Map.read field.
// readOnly 是一个不可变的结构，以原子方式存储在 Map.read 字段中。
type readOnly struct {
	m       map[any]*entry
    // 当dirty中包含read中没有的数据时为 true,比如新增一条数据
	amended bool // true if the dirty map contains some key not in m.
}

// expunged is an arbitrary pointer that marks entries which have been deleted
// from the dirty map.
// expunged用来标识此项已经从dirty中删除了的指针
// 当map中得一个项目被删除了，只是把它标记为expunged，以后才有机会删除此项
var expunged = unsafe.Pointer(new(any))
// 代表一个值
type entry struct {
    p unsafe.Pointer // *interface{}
}
```

### Store方法

用来设置键值对或者更新键值对

```go
// Store sets the value for a key.
func (m *Map) Store(key, value any) {
	read, _ := m.read.Load().(readOnly)
    // 如果read字段包含这个项，说明是更新，cas更新项目的值即可
	if e, ok := read.m[key]; ok && e.tryStore(&value) {
		return
	}
	// read中不存在或者cas更新失败，就需要加锁访问dirty了
	m.mu.Lock()
	read, _ = m.read.Load().(readOnly)
    // 双检查，看看read是否已经存在
	if e, ok := read.m[key]; ok {
		if e.unexpungeLocked() {
			// The entry was previously expunged, which implies that there is a
			// non-nil dirty map and this entry is not in it.
            // 此项先前已经被删除，通过将它的值设为nil,标记为unexpunged
			m.dirty[key] = e
		}
		e.storeLocked(&value) // 更新
	} else if e, ok := m.dirty[key]; ok {
		e.storeLocked(&value) // 直接更新
	} else {
        // 否则就是一个新的key
		if !read.amended {
			// We're adding the first new key to the dirty map.
			// Make sure it is allocated and mark the read-only map as incomplete.
            // 如果dirty为nil,需要创建dirty对象 并标记amended为true,说明有元素它不包含，而dirty包含
			m.dirtyLocked()
			m.read.Store(readOnly{m: read.m, amended: true})
		}
        // 将值增加到dirty对象中
		m.dirty[key] = newEntry(value)
	}
	m.mu.Unlock()
}
```

从`Store()`来看，sync.Map 适合那些只会增长的缓存系统，可以进行更新，但是不要删除，并且不要频繁地增加新元素。

新加入的元素需要放 入dirty中，如果dirty为nil，则需要从read中复制出一个dirty对象：

```go
func (m *Map) dirtyLocked() {
    // 如果不为空，直接返回
	if m.dirty != nil {
		return
	}

	read, _ := m.read.Load().(readOnly)
	m.dirty = make(map[any]*entry, len(read.m))
    // 遍历read
	for k, e := range read.m {
        // 把没有punged字段复制到dirty中
		if !e.tryExpungeLocked() {
			m.dirty[k] = e
		}
	}
}
```

### Load方法

用来读取一个key对应的值，也是从read开始处理，一开始并不需要锁

```go
// Load returns the value stored in the map for a key, or nil if no
// value is present.
// The ok result indicates whether value was found in the map.
func (m *Map) Load(key any) (value any, ok bool) {
    // 从read中读取
	read, _ := m.read.Load().(readOnly)
	e, ok := read.m[key]
    // 如果不存在并且dirty不为nil(有新元素)
	if !ok && read.amended {
		m.mu.Lock()
		// Avoid reporting a spurious miss if m.dirty got promoted while we were
		// blocked on m.mu. (If further loads of the same key will not miss, it's
		// not worth copying the dirty map for this key.)
        // 双检查，从read中读取次key
		read, _ = m.read.Load().(readOnly)
		e, ok = read.m[key]
        // read中不存在并且dirty不为nil
		if !ok && read.amended {
            // 从dirty中读取
			e, ok = m.dirty[key]
			// Regardless of whether the entry was present, record a miss: this key
			// will take the slow path until the dirty map is promoted to the read
			// map.
            // 不管dirty存不存在dirty都加1
			m.missLocked()
		}
		m.mu.Unlock()
	}
	if !ok {
		return nil, false
	}
    // 返回读取的对象，e既可能从read获取的也可能从dirty获得的
	return e.load()
}
```

m.missLocked()增加miss时，如果miss数等于dirty长度，会将dirty提升为read，并将dirty置空 

```go
func (m *Map) missLocked() {
    // miss数加1
	m.misses++
    // miss数如果没有到达dirty的长度，返回
	if m.misses < len(m.dirty) {
		return
	}
	m.read.Store(readOnly{m: m.dirty}) // 把dirty字段的内存提升为read
	m.dirty = nil // 清空dirty
	m.misses = 0  // miss数重置为0
}
```

### Delete方法

`LoadAndDelete`为欧长坤在1.15提供的方法

```go
// LoadAndDelete deletes the value for a key, returning the previous value if any.
// The loaded result reports whether the key was present.
func (m *Map) LoadAndDelete(key any) (value any, loaded bool) {
    // 从read中读取
	read, _ := m.read.Load().(readOnly)
	e, ok := read.m[key]
	if !ok && read.amended {
		m.mu.Lock()
        // 双检查
		read, _ = m.read.Load().(readOnly)
		e, ok = read.m[key]
		if !ok && read.amended {
			e, ok = m.dirty[key]
			delete(m.dirty, key)
			// Regardless of whether the entry was present, record a miss: this key
			// will take the slow path until the dirty map is promoted to the read
			// map.
            // 无论是否存在都记录为未命中，这个键将走慢路径，直到dirty提升为read
            // miss加1
			m.missLocked()
		}
		m.mu.Unlock()
	}
	if ok {
        // read中存在则删除元素
		return e.delete()
	}
	return nil, false
}

// Delete deletes the value for a key.
func (m *Map) Delete(key any) {
	m.LoadAndDelete(key)
}
// 删除元素
func (e *entry) delete() (value any, ok bool) {
	for {
		p := atomic.LoadPointer(&e.p)
        // 如果不为nil或者被标记为expunged，则返回nil,false
		if p == nil || p == expunged {
			return nil, false
		}
        // cas原子操作，&e.p 和 p相等，则置为nil
		if atomic.CompareAndSwapPointer(&e.p, p, nil) {
			return *(*any)(p), true
		}
	}
}
```