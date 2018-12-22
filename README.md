# ScopedContextPattern

# Паттерн блочного контекста

Суть: в момент исполнения кода у нас есть блоки - scope

    using(var contextScope = ScopeManager.CreateScope<SomeContextClass>("ScopeName1"))
    {
       contextScope.X = 10;
       contextScope.Text = "SomeText"; 
       ...
       SomeFunctionCallX(...);       
       ...
    }
    
    class SomeContextClass
    {
        public int X;
        public string Text;
    }
    
Внутри блока `ScopeName1` теперь у нас в любом методе любого класса будет 
доступен контекст `ScopeName1` и мы сможем в нём получить переданные переменные `X` и `Text`

Допустим у нас внутри SomeFunctionCall будет вызываться ещё функция `X2` некоторого класса.
Внутри неё `X3`, и так далее.

Теперь в функции X4 мы получаем контекст и данные из него.

    X4() {
        var context = ScopeManager.GetScope<SomeContextClass>("ScopeName1")
  
        var a = context.x;
        var b = context.y;
    }
    
Этот патерн позволяет передавать в любую глубину функций любые данные 
без прокидывания в аргументах функций по цепочке.

Например на основе этого паттерна можно сделать Dependanci Injection контейнер, который будет
сочетать в себе плюсы как статического контейнера, так и локального.

Мы можем внутрь любого блока кода прокидывать любые сервисы или объекты, и всё это может быть
легко проверено в тестах, так как контекст и сервисы в нём задаются внутри обределения блока.


    using(var contextScope = ScopeManager.CreateScope<SomeContextClass>("ScopeName1"))
    {
       contextScope.X = 10;
       contextScope.Text = "SomeText"; 
       ...   
    }
    
При этом доступ к контексту есть из любого места код без прокидывания через промежуточные функции.
