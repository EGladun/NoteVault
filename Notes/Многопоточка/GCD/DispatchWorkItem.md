---
tags:
  - GCD
  - Multithreading
  - DispatchWorkItem
---
### Описание
**DispatchWorkItem** – класс, являющийся абстракцией над выполняемой задачей, который предоставляет нам ряд полезных методов. Например метод notify, позволяющий уведомить какую-либо очередь о выполнении задачи и следом выполнить какую-либо работу на уведомленной очереди

- workItem – работа, которую необходимо выполнить (в простом случае closure)
- можно отменить пока не начали выполнение (cancel)
- настраиваемые свойства — quality of service (приоритет) и DispatcWorkItemFlags (как эта задача будет исполняться)
### Примеры
```swift
// Создаем очередь
let serialQueue = DispatchQueue(label: "ru.denisegaluev.serial-queue")

// Создаем DispatchWorkItem и передаем в него замыкание (задачу)
let workItem = DispatchWorkItem {    print("DispatchWorkItem task")}

// Реализуем метод notify, передаем в него очередь, на которой необходимо будет выполнить задачу после завершения выполнения этого 
DispatchWorkItemworkItem.notify(queue: DispatchQueue.main) {
	print("DispatchWorkItem completed")
}

// Выполняем DispatchWorkItem на очереди serialQueue
serialQueue.async(execute: workItem)
```
Попробуем реализовать данную логику без использования `DispatchWorkItem`:
```swift
let serialQueue = DispatchQueue(label: "ru.denisegaluev.serial-queue")

serialQueue.async {
	print("task")
	
	DispatchQueue.main.sync {
		print("completed")
	}
}
```
Сравнивая данные примеры видно, что `DispatchWorkItem` позволяет нам более явно задать логику, без использования вложенных друг в друга замыканий и хаотичных вызовов методов `async / sync`.

Помимо notify, `DispatchWorkItem` дает нам возможность отменять задачу с помощью метода `cancel`. Важно понимать, что задачу можно отменить только в том случае, если она на момент отмены ожидает в очереди. Если поток уже начал выполнять задачу, она не будет отменена. Рассмотрим пример реализации метода `cancel`:
```swift
// Создаем очередь
let serialQueue = DispatchQueue(label: "ru.denisegaluev.serial-queue")

// Создаем DispatchWorkItem и передаем в него замыкание (задачу)
let workItem = DispatchWorkItem {    
	print("DispatchWorkItem task")
}

// Усыпляем serialQueue на 1 секунду и сразу возвращаем управление
serialQueue.async {
	print("zzzZZZZ")
	sleep(1)
	print("Awaked")
}

// Ставим workItem в очередь serialQueue и сразу возвращаем управление
serialQueue.async(execute: workItem)

// Отменяем workItem
workItem.cancel()
```
Пока `serialQueue` будет спать, мы успеем отменить `workItem`, тем самым удалив его из очереди `serialQueue`.