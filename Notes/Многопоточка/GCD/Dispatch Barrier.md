---
tags:
  - GCD
  - Multithreading
  - DispatchBarrier
  - Queue
---
### Описание
**Dispatch Barrier** – механизм синхронизации задач в очереди. Для того, чтобы добавить барьер, необходимо передать соответствующий флаг в метод async. 
```swift
// Создаем параллельную очередь
let concurrentQueue = DispatchQueue(label: "ru.anyname.concurrent-queue", attributes: .concurrent)

// Помечаем асинхронный вызов флагом .barrier
concurrentQueue.async(flags: .barrier) {    
	// ...
}
```
Когда мы добавляем барьер в параллельную очередь, она откладывает выполнение задачи, помеченной барьером (и все остальные, которые поступят в очередь во время выполнения такой задачи), до тех пор, пока все предыдущие задачи не будут выполнены. После того, как все предыдущие задачи будут выполнены, очередь выполнит задачу, помеченную барьером самостоятельно. Как только задача с барьером будет выполнена, очередь вернется к своему нормальному режиму работы.
![[Pasted image 20250201003510.png]]
### Примеры
Как работать с барьером на примере реализации read write lock:
```swift
class DispatchBarrierTesting {    
	// Создаем параллельную очередь    
	private let concurrentQueue = DispatchQueue(label: "ru.anyname.concurrent-queue", attributes: .concurrent)        

	// Создаем переменную _value для внутреннего использования    
	private var _value: String = ""        

	// Создаем thread safe переменную value для внешнего использования    
	var value: String {        
		get {            
			var tmp: String = ""                        
				concurrentQueue.sync {                
					tmp = _value            
				}                        
			return tmp        
		}
		
		set {            
			concurrentQueue.async(flags: .barrier) {                
				self._value = newValue            
			}        
		}    
	}
}
```
Данная реализация позволяет гарантировать, что в момент чтения, свойство value не будет изменено из другой очереди.