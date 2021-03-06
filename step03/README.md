## Шаг 3. Имплементируем LRU (часть 1)
И вот теперь мы подобрались к интересным моментам нашей кодлабы. У вас уже есть структура проекта и теперь осталось написать  код :)
Поэтому открываем пакет [storage](storage) в вашей IDE или редакторе и начнем с пакета `storage/lru`

## Структуры данных 

Нам нужно подумать, как же реализовывать кеш. С одной стороны нам нужно key-value хралилище и стандартный `map[interface{}]interface{}` нам подойдет. В этом случае мы сможем и добавлять и удалять элементы в кеше быстро. С другой стороны нам нужен какой-то список из элементов, в котором мы бы могли двигать элементы как вверх стопки, так и забирать последний. Плюс ко всему нам нужна возможность удалять значения из этого списка. Кажется, что [container/list](https://golang.org/pkg/container/list/) нам подойдет.

Получается кеш мы сможем описать следующим образом

```Go
type LRU struct {
  size int
  evictList *list.List
  items map[interface{}]*list.Element
}
```
Также, нам нужна структура, в которой мы будем хранить ключ и значение и хранить структуру будем в нашем списке.
```Go
// entry used to store value in evictList
type entry struct {
	key   interface{}
	value interface{}
}
```

в файле `lru/lru.go` следующий код
```Go
package lru

import (
	"container/list"
)

type (
	LRU struct {
		size      int
		evictList *list.List
		items     map[interface{}]*list.Element
	}
	// entry used to store value in evictList
	entry struct {
		key   interface{}
		value interface{}
	}
)

```

## New
Для того, чтобы создать кеш нам нужно передать его размер в параметрах и проинициализировать все структуры храненния.
Получается следующее
```Go
// New initialized a new LRU with fixed size
func New(size int) (*LRU, error) {
	if size <= 0 {
		return nil, errors.New("Size must be greater than 0")
	}
	c := &LRU{
		size:      size,
		evictList: list.New(),
		items:     make(map[interface{}]*list.Element),
	}
	return c, nil
}
```

## Add

```Go
// Add adds a value to the cache. Return true if eviction occured
func (l *LRU) Add(key, value interface{}) bool {
	if ent, ok := l.items[key]; ok {
		l.evictList.MoveToFront(ent)
		ent.Value.(*entry).value = value
		return false
	}
	ent := &entry{key, value}
	entry := l.evictList.PushFront(ent)
	l.items[key] = entry
}

```
Тут у нас Add делает по сути две вещи. Добавляет и обновляет значение по ключу. При этом с помощью API `container/list` он управляет положением элемента в списке. Не хватает лишь удаления элемента, если у нас элементов в нашем кеше больше чем его размер

## Удаление наименее используемого элемента
```Go
func (l *LRU) removeOldest() {
	ent := l.evictList.Back()
	if ent != nil {
		l.removeElement(ent)
	}
}
func (l *LRU) removeElement(e *list.Element) {
	l.evictList.Remove(e)
	kv := e.Value.(*entry)
	delete(l.items, kv.key)
}
```
По закладываемой логике, нам нужно удалить всего-лишь последний элемент из списка. Удалять нужно будет его и из нашего списка и из карты.

Вернемся к добавлению элемента и внедрим удаление самого старого элемента, если мы превышаем размер хранилища. Получится следующий код

```Go
// Add adds a value to the cache. Return true if eviction occured
func (l *LRU) Add(key, value interface{}) bool {
	if ent, ok := l.items[key]; ok {
		l.evictList.MoveToFront(ent)
		ent.Value.(*entry).value = value
		return false
	}
	ent := &entry{key, value}
	entry := l.evictList.PushFront(ent)
	l.items[key] = entry
	evict := l.evictList.Len() > l.size
	if evict {
		l.removeOldest()
	}
	return evict
}
```
Ну и сразу тест на него.
```Go
// Test that Add returns true/false if an eviction occurred
func TestLRU_Add(t *testing.T) {

	l, err := New(1)
	if err != nil {
		t.Fatalf("err: %v", err)
	}

	if l.Add(1, 1) == true {
		t.Errorf("should not have an eviction")
	}
	if l.Add(2, 2) == false {
		t.Errorf("should have an eviction")
	}
}

```
## Len
С Len() все просто. Нам нужно вернуть только длину списка
```Go
// Len returns the number of items in cache
func (l *LRU) Len() int {
	return l.evictList.Len()
}
```

# Поздравляю

Вы реализовали создание, добавление и удаление самого старого элемента из кеша и написали тест на добавление элемента. Мы продолжим работу в [следующей части](../step04/README.md)
