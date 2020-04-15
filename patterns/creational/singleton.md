# Паттерн "Singleton"

## Введение

`Singleton` - это паттерн, который гарантирует, что мы можем иметь один(в некоторых случаях не только один) экземпляр данного класса, без возможности прямого создания.
Полезен в случаях, когда нам необходимо иметь только один экземпляр, т.е например менеджер работы с БД, логгер и т.д
В таких случаях нам не только не надо иметь еще один менеджер или логгер, но и это может быть опасно, так как начнут появляться ошибки.

Самый простой пример, это:

```java
public enum Car{
    DODGE, LAND_ROVER, AUDI
}
``` 

Мы не можем создать никак новые инстансы класса `Car` и каждый из них существует в единственном экземпляре.

Разберем другие реализации паттерна:

## Реализации

### Попытка 0

Разумеется, если чуть-чуть подумать, то мы можем написать что-то подобное.

```java
class Singleton {
private static Singleton instance = new Singleton();;
private Singleton() {}

public static Singleton getInstance() {
return instance;
}
}
```

При желании можно даже обрабатывать исключения:

```java
public class StaticBlockSingleton {
    private static StaticBlockSingleton instance;

    private StaticBlockSingleton() {
    }

   static {
		try {
			instance = new StaticBlockSingleton();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	public static StaticBlockSingleton getInstance() {
		return instance;
	}
}
```

Минусы этого подхода в том, что мы не используем ленивую инициализацию - мы сразу создаем объект класса.

Тем не менее, это не самый плохой вариант.
**Пойдет**

### Попытка 1

```java
class Singleton {
  private static Singleton instance;
  private Singleton() {}

    public static Singleton getInstance() {
      if(instance == null) {
        instance = new Singleton();
      }
      return instance;
    }
}
```

Мы решили учесть замечание про ленивую инициализацию, поэтому вынесли создание экземпляра класса в метод, т.е по обращению.
Однако такой способ - плох, он будет правильно работать только без многопоточного доступа.
Почему?

Потому что, если у нас есть несколько потоков, и первый из них обращается к методу `getInstance()` нашего класса, он видит, что instance -  `null` и создает объект, но в это же время другой поток может также обратиться к  `getInstance()` увидеть, что instance -  `null`(так как еще не создался объект, он только начал создаваться).И второй поток также начнет создавать объект `Singleton()` т.е по сути мы дважды создали объект и в ссылке `instance` будет именно второй хранится. Это плохо, так как, если создание объекта довольно дорого, занимает время или еще что-нибудь такое, то мы дважды(или трижды, например) сделаем одну и ту же работу по созданию объекта.

Не подходит для многопоточности.
**Не рекомендуется.**

#### Попытка 2

Учтем прошлую попытку и добавим `synchronized` для `getInstance`, теперь сигнатура выглядит как:
`public static synchronize Singleton getInstance()`

В итоге:

```java
class Singleton {
private static Singleton instance;
private Singleton() {}

public static synchronize Singleton getInstance() {
  if(instance == null) {
    instance = new Singleton();
  }
  return instance;
  }
}
```

Или так:

```java
public class Singleton{

    private static Singleton self = null;

    private SingletonImpl(){
        self = this;
    }

    public static synchronized Singleton getInstance(){
        return (self == null) ? new SingletonImpl() : self;
    }
}
```

И это решает все наши проблемы.
Все, кроме той, что пригодится нам `synchronized` может только на этапе создания объекта, а в итоге использовать мы ее будем всегда. Т.е мы всегда будем бороться за монитор - независимо нужен он или нет.

В принципе, `synchronized` - не настолько дорогостоящая штука, но тот факт, что мы будем ее постоянно использовать, а также что она все-таки что-то да отъедает от производительности - не радует.
Мы уже создаем критическую секцию.

Не так хорошо! Но - терпимо, к тому же синхронизация по данным работает все-таки не так медленно.

Как можно улучшить этот пример еще? Самый видимый - это уменьшить *гранулярность* блока синхронизации.
Т.е уменьшить блок синхронизации - занести его в метод.

```java
class Singleton {
private static Singleton instance;
private Singleton() {}

public static synchronize Singleton getInstance() {
  if(instance == null) {
    synchronized (Singleton.class) {
      if (instance == null) {
        instance = new Singleton();
      }
    }
  }
  return instance;
  }
}
```

Теперь мы будем бороться за монитор *только* в случае, если ссылка - `null`, т.е при первой инициализации этого класса.

**Хорошее решение!**

### Попытка 3

```java
public class DoubleCheckedLockingSingleton {
	private static volatile DoubleCheckedLockingSingleton instance;

	public static DoubleCheckedLockingSingleton getInstance() {
		DoubleCheckedLockingSingleton localInstance = instance;
		if (localInstance == null) {
			synchronized (DoubleCheckedLockingSingleton.class) {
				localInstance = instance;
				if (localInstance == null) {
					instance = localInstance = new DoubleCheckedLockingSingleton();
				}
			}
		}
		return localInstance;
	}
}
```

Тут мы используем двойную проверку. Мы избавились от общей синхронизации, у нас будет работать в многопоточной среде это решение, но мы используем `volatile` - а он появился только в версиях `Java 1.5+`. Но сейчас все равно все пишут на старших версиях, так что это ничего страшного.

Но почему мы должны использовать `volatile`?

Что происходит когда мы создаем объект:

* Мы выделяем память для этого объекта.
* Инициализируем указатель
* Вызываем конструктор объекта

И иногда, с *очень-очень* маленькой долей вероятности(на некоторых `JVM` может быть такое старых) мы можем начать использовать наш объект **ДО** того как мы дойдем до конца списка, т.е до конструктора. Т.е какой-то другой поток уже начнет использовать объект по указателю(указатель то будет не нулевой уже), а сам объект не до конца сконструируется.
Использование же `volatile` предотвратит это. Опять же, я только читал, что такое возможно, но никогда не видел подтверждения. А в `Java 1.5+` говорят это и вовсе пофиксили.

**Хорошо!**

#### Попытка 4

```java
public class BillPughSingleton {

	private BillPughSingleton() {
	}

	private static class SingletonHelper {
		private static final BillPughSingleton INSTANCE = new BillPughSingleton();
	}

	public static BillPughSingleton getInstance() {
		return SingletonHelper.INSTANCE;
	}
}
```

Реализация Билла Пью. Используется `Holder-класс`.
Ленивая инициализация, нет проблем с производительностью, но не получится вне `static` использовать и обрабатывать ошибки будет тяжеловато.

**Хорошо!**

## Заключение и советы

* Попытки номер два и три - отличный выбор!
* Используйте также `final` для `Singleton` класса по умолчанию и используйте наследования от `Singleton` класса только в случае крайней необходимости, так как могут возникнуть проблемы.
