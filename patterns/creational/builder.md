# Паттерн Builder

## Введение

Представим ситуацию, в которой нам надо работать с классом, содержащим большое количество параметров.
Нашей задачей является присвоение объекту некоторого состояния наиболее удобным и безопасным путем.

Давайте разберем - какие возможные варианты есть для работы с такими классами в `Java`?

Для примера будем оперировать некоторой сущностью: телефоном.
Класс `Telephone` содержит серийный номер, имя телефона, марку и т.д.

Первый способ присваивания этим параметрам значений очевиден - это работа с конструктором класса.

## Telescoping constructor pattern

В данном подходе мы объявляем конструкторы, в которые передаем значения и присваиваем их полям класса.

```java
public class TelephoneTelescopingPattern {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;

    public TelephoneTelescopingPattern(int serialnumber, String name, String screenName, String mark) {
        this.serialnumber = serialnumber;
        this.name = name;
        this.screenName = screenName;
        this.mark = mark;
    }
}
```

Подход прост и хорош в ситуациях, когда количество параметров, передаваемых в конструктор, не превышает разумные пределы.
Обычно считается, что более пяти параметров - это уже перебор.

Однако, в ситуациях, когда передаваемых аргументов много данный подход **не рекомендуется**.

Так как мы должны передать большое количество аргументов, то подобный код становится тяжело читать, сложнее разбираться в передаваемых параметрах, становится легче ошибиться и передать не тот параметр и т.д.

Как видите, минусов более чем достаточно, чтобы задуматься о более простом и безопасном способе.

## Java Beans pattern

Этот способ основывается на том, что сначала создается объект(с помощью пустого конструктора или конструктора, принимающего необходимые и обязательные параметры), а уже после этого происходит наполнение уже созданного объекта данными с помощью `set`-ов.

```java
public class TelephoneJavaBeansPattern {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;

    public void setSerialnumber(int serialnumber) {
        this.serialnumber = serialnumber;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setScreenName(String screenName) {
        this.screenName = screenName;
    }

    public void setMark(String mark) {
        this.mark = mark;
    }
}
```

Такой код проще читать, однако из-за разделения вызовов мы имеем объект, который некоторое время находится в неустойчивом состоянии(пока там эти все set-ры выполняются объект находится в полуподвешенном состоянии).
Также такой объект не может быть неизменным.

## Builder Pattern

Как же соединить два эти подхода, но так, чтобы мы обходили недостатки каждого, а использовали бы плюсы?
Суть заключается в том, что мы создаем внутренний класс, часто называемый Builder, через которого выставляем параметры, а после делаем `build()`. Т.е пока мы не сделали build - мы работаем с промежуточным объектом.

Благодаря этому подходу мы можем конструировать наш объект в нужном порядке, более тонкий контроль над процессом конструирования, изолирует код, реализующий конструирование и представление.

Можно по разному сделать Builder и поэтому внизу - два примера.

```java
public class TelephoneBuilderPattern {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;


    private TelephoneBuilderPattern() {

    }

    public class Builder {

        private Builder() {
        }

        public Builder setName(String name) {
            TelephoneBuilderPattern.this.name = name;

            return this;
        }

        public Builder setScreenName(String screenName) {
            TelephoneBuilderPattern.this.screenName = screenName;

            return this;
        }

        public Builder setMark(String mark) {
            TelephoneBuilderPattern.this.mark = mark;
            return this;

        }

        public Builder setSerialnumber(int serialnumber) {
            TelephoneBuilderPattern.this.serialnumber = serialnumber;

            return this;
        }

        public TelephoneBuilderPattern build() {
            return TelephoneBuilderPattern.this;
        }
    }

    public static Builder getBuilder() {
        return new TelephoneBuilderPattern().new Builder();
    }
}
```

Либо вот так:

```java
public class TelephoneBuilderPattern2 {
    private final int serialnumber;
    private final String name;
    private final String screenName;
    private final String mark;

    private TelephoneBuilderPattern2(int serialnumber, String name, String screenName, String mark) {
        this.serialnumber = serialnumber;
        this.name = name;
        this.screenName = screenName;
        this.mark = mark;
    }

    public static class Builder {

        private int serialnumber;
        private String name;
        private String screenName;
        private String mark;

        private Builder() {
        }

        public Builder setName(String name) {
            this.name = name;

            return this;
        }

        public Builder setScreenName(String screenName) {
            this.screenName = screenName;

            return this;
        }

        public Builder setMark(String mark) {
            this.mark = mark;
            return this;

        }

        public Builder setSerialnumber(int serialnumber) {
            this.serialnumber = serialnumber;

            return this;
        }

        public TelephoneBuilderPattern2 build() {
            return new TelephoneBuilderPattern2(serialnumber, name, screenName, mark);
        }
    }

    public static Builder getBuilder() {
        return new Builder();
    }
}
```

Читать и работать с таким кодом гораздо проще, чем при использовании Telescoping, а также безопаснее, чем при использовании Java Beans.

Примеры использования каждого подхода:

```java
public static void main(String[] args) {
    //example of telescoping pattern
    TelephoneTelescopingPattern telephone1 = new TelephoneTelescopingPattern(1, "Sony", "TFT", "X");

    //example of java beans pattern
    TelephoneJavaBeansPattern telephone2 = new TelephoneJavaBeansPattern();
    telephone2.setSerialnumber(2);
    telephone2.setName("Sony");
    telephone2.setScreenName("TFT");
    telephone2.setMark("X");

    //example of builder
    TelephoneBuilderPattern telephone3 = TelephoneBuilderPattern.getBuilder().setName("Sony")
                                                                             .setSerialnumber(3)
                                                                             .setMark("X")
                                                                             .setScreenName("TFT")
                                                                             .build();

    //example of builder2
    TelephoneBuilderPattern2 telephone4 = TelephoneBuilderPattern2.getBuilder().setMark("X")
                                                                               .setName("Sony")
                                                                               .setScreenName("TFT")
                                                                               .setSerialnumber(4)
                                                                               .build();
}
```
