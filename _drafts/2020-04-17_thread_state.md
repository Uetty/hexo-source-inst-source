# Java线程状态

`LockSupport.park`调用`Unsafe.park`会产生			WAITING					(parking)								驻留

`LockSupport.parkNanos`调用`Unsafe.park`会产生	TIMED_WAITING	(parking)								驻留

`Object.wait(milliSeconds)`会产生							TIMED_WAITING	(on object monitor)			等待

`Object.wait()`会产生													WAITING					(on object monitor)			等待		

`Thead.sleep()`会产生													TIMED_WAITING		(sleeping)							休眠

等待synchronized会产生												BLOCKED 					(on object monitor)			监视

正在运行的，会产生														RUNNABLE																运行





新建 -> 就绪 -> 运行 -> 死亡

运行 -> 阻塞 -> 就绪