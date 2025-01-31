---
tags:
  - GCD
  - Multithreading
  - DispatchGroup
---
### Описание
**DispatchGroup** – объект, позволяющий объединить задачи в группу и синхронизировать их поведение. Группа позволяет присоединить к ней несколько задач или [[DispatchWorkItem]] и запланировать их асинхронное выполнение на одной или нескольких очередях. Когда все задачи в группе будут выполнены, группа уведомит об этом какую-либо очередь и выполнит на ней completion handler. Так же группа позволяет нам дождаться выполнения задач в группе синхронно, без использования уведомления.

### Примеры
Пример использования `DispatchGroup`:
```swift
// Создаем очередь
let serialQueue = DispatchQueue(label: "ru.denisegaluev.serial-queue")

// Создаем 2 DispatchWorkItem
let workItem1 = DispatchWorkItem {    
	print("workItem1: zzzZZZ")    
	sleep(3)   
	print("workItem1: awaked")
}
let workItem2 = DispatchWorkItem {    
	print("workItem2: zzzZZZ")    
	sleep(3)    
	print("workItem2: awaked")
}

// Создаем группу
let group = DispatchGroup()

// Добавляем workItem в группе, планируем его выполнение на очереди serialQueue и сразу возвращаем управление
serialQueue.async(group: group, execute: workItem1)
serialQueue.async(group: group, execute: workItem2)

// Устанавливаем уведомление. Замыкание будет выполнено на главной очереди сразу после того, как все задачи в группе будут выполнены.
group.notify(queue: DispatchQueue.main) {    
	print("All tasks on group completed")
}

// Console: 
// workItem1: zzzZZZ
// workItem1: awaked
// workItem2: zzzZZZ
// workItem2: awaked
// All tasks on group completed
```
Рассмотрим, как добиться такого же поведения, но уже использую `enter` и `leave` вместо уведомления:
```swift
// Создаем параллельную очередь
let concurrentQueue = DispatchQueue(label: "ru.denisegaluev.concurrent-queue", attributes: .concurrent)

// Создаем группу
let group = DispatchGroup()

// Создаем DispatchWorkItem
let workItem1 = DispatchWorkItem {   
	print("workItem1: zzzZZZ")    
	sleep(3)    
	print("workItem1: awaked")        
	// Покидаем группу    
	group.leave()
}
let workItem2 = DispatchWorkItem {    
	print("workItem2: zzzZZZ")    
	sleep(3)    
	print("workItem2: awaked")        
	group.leave()
}

// Входим в группу
group.enter()

// Вызываем
concurrentQueue.async(execute: workItem1)

group.enter()
concurrentQueue.async(execute: workItem2)

// Ожидаем, пока все задачи в группе закончат свое выполнение
group.wait()
print("All tasks on group completed")

// Console:
// workItem1: zzzZZZ
// workItem2: zzzZZZ
// workItem1: awaked
// workItem2: awaked
// All tasks on group completed
```
Обратите внимание, что в данном случае нам не нужно добавлять задачи в группу (в аргумент group метода async). Вместо этого мы вызываем метод группы `enter`, тем самым указывая явно, что задача вошла в группу, а в конце выполнения задачи вызываем метод `leave`, тем самым явно указывая, что задача завершила свое выполнение. Таким образом очередь в которой был вызван `wait` (в нашем случае главная очередь), будет ожидать до тех пор, пока все задачи в группе не завершат свое выполнение и не вызовут метод `leave`.