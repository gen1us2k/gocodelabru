# Шаг 1. Делаем структуру и архитектуру проекта

При старте проекта нужно сделать структуру проекта и запланировать архитектуру.

Мы сделаем два пакета:

1. `storage` для "главного хранилища
2. `storage/lru` для LRU хранилища

Вы можете начать писать код в пакете  [storage](./storage)

## Думаем над функционалом и планируем архитектуру
### LRU
Начнем, пожалуй с LRU кеша. Кеш у нас будет определенного размера. При инициализации его нужно будет указать сколько элементов хранить. В этом случае у нас в кеше всегда будет N элементов. При добавлении элемента, когда хранилище заполнено, мы удалим менее используемый элемент. От кеша нам нужны будут следующие методы:

1. New(size) - Метод, который создаст там LRU кеш
2. Add(key, value) - для добавления элемента 
3. Get(key) - для получения элемента
4. Contains(key) - для проверки, существует элемент или нет
5. Remove(key) - для удаления элемента из кеша
6. RemoveOldest() - для удаления самого старого элемента
7. Purge() - для очистки всего кеша
8. Len() - для того, чтобы знать какой у нас размер хранилища
9. Keys() - для того, чтобы получить все ключи из хранилища

### Storage

От хранилища нам потребуются следующие методы:

1. New(size) - для инциализации стораджа
2. Set(key, value) - для добавления или обновления элемента
3. Delete(key) - для удаления 
4. Nearest() - для получения блищайших элементов

## Что хранить?
Хранить будем следующее
```Go
type (
  Location struct {
    Lat float64
    Lon float64
  }
  Driver struct {
    ID int 
    LastLocation Location
    Locations *lru.LRU
  }
```

## Поздравления
Вы прошли первый шаг. Мы определились с тем, что нам нужно будет реализовывать и какие данные хранить. Давайте разберемся с тем, как тестировать и что нужно знать о тестах в Go.
Переходим ко [второму](../step02/README.md) шагу
