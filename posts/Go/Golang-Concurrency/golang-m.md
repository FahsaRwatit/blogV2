---
title: "M找工作"
description: "M找工作"
date: 2022-09-20 18:10:00
slug: golang-m
image:
categories:
    - Go
tags: ["Go"]

---

### 

### 从本地队列中找goroutine

```go
// Get g from local runnable queue.
// If inheritTime is true, gp should inherit the remaining time in the
// current time slice. Otherwise, it should start a new time slice.
// Executed only by the owner P.
// 从本地可运行队列中获取 g。 如果inheritTime为true，gp应该继承当前时间片的剩余时间。 否则，它应该开始一个新的时间片。 仅由所有者 P 执行。
func runqget(_p_ *p) (gp *g, inheritTime bool) {
	// If there's a runnext, it's the next G to run.
    // 如果有 runnext，它就是下一个要运行的 G。
	next := _p_.runnext
	// If the runnext is non-0 and the CAS fails, it could only have been stolen by another P,
	// because other Ps can race to set runnext to 0, but only the current P can set it to non-0.
	// Hence, there's no need to retry this CAS if it falls.
    // 如果runnext不为空
    // 用原子操作获取 runnext，并将其值修改为 0，也就是空。这里用到原子操作的原因是防止在这个过程中，有其他线程过来“偷工作”，导致并发修改 runnext 成员。
	if next != 0 && _p_.runnext.cas(next, 0) {
        // 返回next所指向的G,且需要继承时间片
		return next.ptr(), true
	}
    // 在尝试获取runnext失败后，尝试从本地队列中返回队列头的goroutine
	for {
        // 获取队列头，原子操作
		h := atomic.LoadAcq(&_p_.runqhead) // load-acquire, synchronize with other consumers
		// 获取队列尾，不使用原子操作是因为不用担心其他线程同时修改，（偷工作只会修改队列头）
        t := _p_.runqtail
		if t == h {
            // 队列头和尾相等，说明队列为空，找不到g
			return nil, false
		}
        // 获取队列头的g,算出队列头指向的goroutine
		gp := _p_.runq[h%uint32(len(_p_.runq))].ptr()
        // CAS原子操作尝试修改队列头，防止这中间被其他线程因为偷工作而修改
		if atomic.CasRel(&_p_.runqhead, h, h+1) { // cas-release, commits consume
			return gp, false
		}
	}
}
```

### 从全局队列中找goroutine

```go
// Check the global runnable queue once in a while to ensure fairness.
// Otherwise two goroutines can completely occupy the local runqueue
// by constantly respawning each other.
// 为了公平，每调用schedule调度函数61次，就要从全局可运行goroutine队列中获取
// 从全局队列中获取需要上锁
if _p_.schedtick%61 == 0 && sched.runqsize > 0 {
    lock(&sched.lock)
    // 从全局队列中最大获取1个goroutine
    gp = globrunqget(_p_, 1)
    unlock(&sched.lock)
    if gp != nil {
        return gp, false, false
    }
}
```

从全局队列中获取goroutine：

首先根据全局队列的可运行 goroutine 长度和 P 的总数，来计算一个数值，表示每个 P 可平均分到的 goroutine 数量。

然后根据函数参数中的 max 以及 P 本地队列的长度来决定把多少全局队列中的 goroutine 转移到 P 本地。

最后，for 循环挨个把全局队列中 n-1 个 goroutine 转移到本地，并且返回最开始获取到的队列头所指向的 goroutine，毕竟它最需要得到运行的机会。

```go
// Try get a batch of G's from the global runnable queue.
// sched.lock must be held.
// 尝试从全局队列中获取一批goroutine
func globrunqget(_p_ *p, max int32) *g {
	assertLockHeld(&sched.lock)
	// 如果队列为空，返回nil
	if sched.runqsize == 0 {
		return nil
	}
	// 根据p的数量，平分全局运行队列中的goroutines
	n := sched.runqsize/gomaxprocs + 1
	if n > sched.runqsize {
		n = sched.runqsize // 如果 gomaxprocs 为 1
	}
    // 修正"偷"的数量
	if max > 0 && n > max {
		n = max
	}
    // 最多只能"偷"本地工作队列一半的数据
	if n > int32(len(_p_.runq))/2 {
		n = int32(len(_p_.runq)) / 2
	}
	// 更新全局可运行队列的长度
	sched.runqsize -= n
    // 从队列头中获取g
	gp := sched.runq.pop()
	n--
    // 批量将全局队列中的g转移到本地队列
	for ; n > 0; n-- {
		gp1 := sched.runq.pop()
        // 将gp1放入p本地,使全局队列得到更多的执行机会
		runqput(_p_, gp1, false)
	}
    // 返回最开始获取到的队列头所指向的 goroutine
	return gp
}
```

### 从其他P中窃取

```go
// 从其他地方找goroutine来执行
func findRunnable() (gp *g, inheritTime, tryWakeP bool) {
	_g_ := getg()

	// The conditions here and in handoffp must agree: if
	// findrunnable would return a G to run, handoffp must start
	// an M.

top:
    // 省略了一部分代码
	....

	// local runq
    // 从本地队列中获取
	if gp, inheritTime := runqget(_p_); gp != nil {
		return gp, inheritTime, false
	}

	// global runq
    // 从全局队列的头部获取一个g
	if sched.runqsize != 0 {
		lock(&sched.lock)
		gp := globrunqget(_p_, 0)
		unlock(&sched.lock)
		if gp != nil {
			return gp, false, false
		}
	}
	// 省略代码
	............

	// Spinning Ms: steal work from other Ps.
	procs := uint32(gomaxprocs)
   
	if _g_.m.spinning || 2*atomic.Load(&sched.nmspinning) < procs-atomic.Load(&sched.npidle) {
		if !_g_.m.spinning {
            // 设置为自旋状态
			_g_.m.spinning = true
            // 自旋状态加1
			atomic.Xadd(&sched.nmspinning, 1)
		}
		// 窃取工作，从其他p的本地队列中窃取goroutine
		gp, inheritTime, tnow, w, newWork := stealWork(now)
		now = tnow
		if gp != nil {
			// Successfully stole.
			return gp, inheritTime, false
		}
		if newWork {
			// There may be new timer or GC work; restart to
			// discover.
			goto top
		}
		if w != 0 && (pollUntil == 0 || w < pollUntil) {
			// Earlier timer to wait for.
			pollUntil = w
		}
	}

	// 省略代码
    .......

	// return P and block
	lock(&sched.lock)
	if sched.gcwaiting != 0 || _p_.runSafePointFn != 0 {
		unlock(&sched.lock)
		goto top
	}
	if sched.runqsize != 0 {
		gp := globrunqget(_p_, 0)
		unlock(&sched.lock)
		return gp, false, false
	}
    // // 当前工作线程解除与 p 之间的绑定，准备去休眠
	if releasep() != _p_ {
		throw("findrunnable: wrong p")
	}
    // 把p放入空闲队列
	now = pidleput(_p_, now)
	unlock(&sched.lock)

	wasSpinning := _g_.m.spinning
	if _g_.m.spinning {
         // M即将休眠不在处于自旋状态
		_g_.m.spinning = false
		if int32(atomic.Xadd(&sched.nmspinning, -1)) < 0 {
			throw("findrunnable: negative nmspinning")
		}

		// Check all runqueues once again.
        // 再次检查所有运行队列
		_p_ = checkRunqsNoP(allpSnapshot, idlepMaskSnapshot)
		if _p_ != nil {
			acquirep(_p_)
			_g_.m.spinning = true
			atomic.Xadd(&sched.nmspinning, 1)
			goto top
		}
		// 省略代码
		.....
	}
	// 省略代码
	.....
    // 休眠
	stopm()
	goto top
}
```



```go
// 窃取工作，从其他p的本地队列中窃取goroutine
// 
func stealWork(now int64) (gp *g, inheritTime bool, rnow, pollUntil int64, newWork bool) {
	pp := getg().m.p.ptr()

	ranTimer := false
	// 会尝试4次扫遍所有的G
	const stealTries = 4
	for i := 0; i < stealTries; i++ {
		stealTimersOrRunNextG := i == stealTries-1
		// 第二层循环，不是每次按照固定的顺序去遍历所有的G
        // 遍历所有的g,每次遍历开始时随机选择一个位置，直到所有p遍历完
		for enum := stealOrder.start(fastrand()); !enum.done(); enum.next() {
			if sched.gcwaiting != 0 {
				// GC work may be available.
				return nil, false, now, pollUntil, true
			}
            // 随机获取一个p
			p2 := allp[enum.position()]
			if pp == p2 {
				continue
			}

			// Steal timers from p2. This call to checkTimers is the only place
			// where we might hold a lock on a different P's timers. We do this
			// once on the last pass before checking runnext because stealing
			// from the other P's runnext should be the last resort, so if there
			// are timers to steal do that first.
			//
			// We only check timers on one of the stealing iterations because
			// the time stored in now doesn't change in this loop and checking
			// the timers for each P more than once with the same value of now
			// is probably a waste of time.
			//
			// timerpMask tells us whether the P may have timers at all. If it
			// can't, no need to check at all.
			if stealTimersOrRunNextG && timerpMask.read(enum.position()) {
				tnow, w, ran := checkTimers(p2, now)
				now = tnow
				if w != 0 && (pollUntil == 0 || w < pollUntil) {
					pollUntil = w
				}
				if ran {
					// Running the timers may have
					// made an arbitrary number of G's
					// ready and added them to this P's
					// local run queue. That invalidates
					// the assumption of runqsteal
					// that it always has room to add
					// stolen G's. So check now if there
					// is a local G to run.
					if gp, inheritTime := runqget(pp); gp != nil {
						return gp, inheritTime, now, pollUntil, ranTimer
					}
					ranTimer = true
				}
			}

			// Don't bother to attempt to steal if p2 is idle.
            // 确定好准备偷的对象后
			if !idlepMask.read(enum.position()) {
                // 调用runqsteal执行
				if gp := runqsteal(pp, p2, stealTimersOrRunNextG); gp != nil {
					return gp, false, now, pollUntil, ranTimer
				}
			}
		}
	}

	// No goroutines found to steal. Regardless, running a timer may have
	// made some goroutine ready that we missed. Indicate the next timer to
	// wait for.
	return nil, false, now, pollUntil, ranTimer
}
```

runqsteal：

```go
// 从 p2 的本地可运行队列中窃取一半元素并放入 p 的本地可运行队列中。 返回被盗元素之一（如果失败则返回 nil）。
// 从p2中偷走一半的工作放入到_p_的本地
func runqsteal(_p_, p2 *p, stealRunNextG bool) *g {
    // 队尾
	t := _p_.runqtail
    // 从p2中偷取工作放入到_p_.runq队尾
	n := runqgrab(p2, &_p_.runq, t, stealRunNextG)
	if n == 0 {
		return nil
	}
	n--
    // 找到最后一个g准备返回
	gp := _p_.runq[(t+n)%uint32(len(_p_.runq))].ptr()
	if n == 0 {
		return gp
	}
    // 队列头
	h := atomic.LoadAcq(&_p_.runqhead) // load-acquire, synchronize with consumers
    // 判断是否偷太多
	if t-h+n >= uint32(len(_p_.runq)) {
		throw("runqsteal: runq overflow")
	}
    // 更新队尾，将偷来的工作加入队列
	atomic.StoreRel(&_p_.runqtail, t+n) // store-release, makes the item available for consumption
	return gp
}
```

`runqgrab`:

```go
// 从p2中偷取工作放入到_p_.runq队尾
n := runqgrab(p2, &_p_.runq, t, stealRunNextG)
```



```go
// Grabs a batch of goroutines from _p_'s runnable queue into batch.
// Batch is a ring buffer starting at batchHead.
// Returns number of grabbed goroutines.
// Can be executed by any P.
// 从_p_中获取可运行的 goroutine 放到batch数组里
// batch是一个环，起始于batchead
// 返回偷的数量，返回的goroutine可被任何p执行
func runqgrab(_p_ *p, batch *[256]guintptr, batchHead uint32, stealRunNextG bool) uint32 {
	for {
        // 队列头
		h := atomic.LoadAcq(&_p_.runqhead) // load-acquire, synchronize with other consumers
		// 队列尾
        t := atomic.LoadAcq(&_p_.runqtail) // load-acquire, synchronize with the producer
		//g的数量
        n := t - h
        // 取一半
		n = n - n/2
		if n == 0 {
			if stealRunNextG {
				// Try to steal from _p_.runnext.
                // 从runnext中偷，没有人性
				if next := _p_.runnext; next != 0 {
					if _p_.status == _Prunning {
				 // 这里是为了防止 _p_ 执行当前 g，并且马上就要阻塞，所以会马上执行 runnext，
                    // 这个时候偷就没必要了，因为让 g 在 P 之间"游走"不太划算，
                    // 就不偷了，给他们一个机会。
                    // channel 一次同步的的接收发送需要 50ns 左右，因此 3us 差不多给了他们 50 次机会了，做得还是不错的
						if GOOS != "windows" && GOOS != "openbsd" && GOOS != "netbsd" {
							usleep(3)
						} else {
							// On some platforms system timer granularity is
							// 1-15ms, which is way too much for this
							// optimization. So just yield.
							osyield()
						}
					}
					if !_p_.runnext.cas(next, 0) {
						continue
					}
                    // 真的偷走了next
					batch[batchHead%uint32(len(batch))] = next
                    // 返回偷的数量，只有1个
					return 1
				}
			}
			return 0
		}
       // 如果 n 这时变得太大了，重新来一遍了，不能偷的太多，做得太过分了
		if n > uint32(len(_p_.runq)/2) { // read inconsistent h and t
			continue
		}
        // 将G放入到batch中
		for i := uint32(0); i < n; i++ {
			g := _p_.runq[(h+i)%uint32(len(_p_.runq))]
			batch[(batchHead+i)%uint32(len(batch))] = g
		}
        // 工作被偷走了，更新一下队列头指针
		if atomic.CasRel(&_p_.runqhead, h, h+n) { // cas-release, commits consume
			return n
		}
	}
}
```

#### releasep

解出P与M的关联

```go
// 解除 p 与 m 的关联
func releasep() *p {
	_g_ := getg()
	// .... 省略的代码
	_p_ := _g_.m.p.ptr()
	// .... 省略的代码
    // 清空一些字段
	_g_.m.p = 0
	_p_.m = 0
	_p_.status = _Pidle
	return _p_
}
```



