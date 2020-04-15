# Паттерн "Фабрика"

## Введение

Когда вам требуется получать какие-то объекты, например телефон, пакет сока, вам совершенно не нужно знать как их делают на фабрике. Вы просто говорите "сделайте мне айфон" или "сделайте мне sony" , а фабрика возвращает вам требуемый телефон.  При этом как она его делает - уже дело фабрики.

Можно использовать по-разному, например, создавать фабрику соединений к бд, которая будет вам в зависимости от требований возвращать query к oracle бд или к postgresql.

Для простейшего примера я выбрал просто фабрику, производящую компьютеры.

## Реализация

В зависимости от типа запроса мы возвращаем либо стационарный компьютер, либо ноутбук и т.д
При этом нам не важно что там происходит, мы просто говорим, `getComputer()` - и получаем Computer, с которым уже работаем.

```java
public class ComputerFactory {
    public Computer getComputer(ComputerType type) {
        Computer computer = null;
        switch (type) {
            case DESKTOP:
                computer = new DesktopComputer("HP");
                break;
            case NOTEBOOK:
                computer = new NotebookComputer("Acer");
                break;
            case TABLET:
                computer = new TabletComputer("ASUS");
                break;
        }

        return computer;
    }
}
```

/todo переписать
