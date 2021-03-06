# ScopedContextPattern

# Паттерн блочного контекста

Это статья это не реализация данного паттерна, а текстовое описание основной сути этой идеи.
Весь код в данном примере следует рассматривать не как конечный вариант синтаксиса, а как
наглядное пояснение самой идеи.

В момент исполнения кода у нас есть блоки - scope

    // создание блока контекста
    using(var contextScope = ScopeManager.CreateScope<SomeContextClass>("ScopeName1"))
    {
       // задание контекста
       contextScope.X = 10;
       contextScope.Text = "SomeText"; 
       ...
       X1(...); // вызов некоторой функции       
       ...
    }
    
    // Класс контекста
    class SomeContextClass
    {
        public int X;
        public string Text;
    }
    
Внутри блока `"ScopeName1"`, в любом методе любого класса будет можно получить 
контекст `SomeContextClass` с именем `"ScopeName1"` и любые данные из него, в данном примере
`X` и `Text`.

Допустим у нас внутри `X1` будет вызываться ещё функция `X2` некоторого класса.
Внутри неё `X3`, потом `X4`.

Теперь в функции `X4` мы получаем контекст и данные из него.


    // Неокторая функция которая будет вызвана в глубине вызовов `X1`
    X4() {
        // Получение контекста по заданной сигнатуре и имени
        var context = ScopeManager.GetScope<SomeContextClass>("ScopeName1");
  
        // использование данных контекста
        var a = context.x;
        var b = context.Text;
    }
    
Этот патерн позволяет передавать в любую глубину функций любые данные 
без прокидывания в аргументах функций `X1`, `X2`, `X3`, `X4` по цепочке.

Например на основе этого паттерна можно сделать Dependancy Injection контейнер, который будет
сочетать в себе плюсы как статического контейнера, так и локального.

Плюсы статического
  - Доступность из любого места
  - Нет необходимости прокидывать через аргументы функций внтурь глубины вложенности функций

Плюсы локального
  - Можно отлаживать легко подменяя данные контекста.
  
Пример кода для тестов:

    using(var contextScope = ScopeManager.CreateScope<SomeContextClass>("ScopeName1"))
    {
       contextScope.X = 5000;  // можем в коде теста передать любые данные
       contextScope.Text = "TextText"; 
       
       // Код теста
    }

    
В данной статье определена основная идея паттерна, её не стоит рассматривать как конечную
форму для реализации.
Синтаксис взят для примера и понимания сути.
Перед реализацией надо будет более детально продумать различные моменты, например способ
идентификации блока, в данном примере это класс контекста и строка названия блока
`<SomeContextClass>("ScopeName1")`. 

Для методов, классов которые используют подключённые через ScopedContextPattern сервисы необходимо
задавать атрибут `UseServices` в котором будет указано какие именно сервисы используются внутри кода.  
`[UseServices(typeof(Service1),typeof(Service2))]`
Это нужно что бы сразу по сигнатуре методов было известно какие сервисы он использует.  
При попытке использовать сервис не объявленный через `UseServices`- исключение.