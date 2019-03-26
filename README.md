# Лабораторная работа 9

## Задание
Создать библиотеку для расчета сопротивления электрической цепи переменного тока.

Библиотека должна расчитывать сопротивления:
* последовательных соединений элементов (`SequentialConnections`);
* параллельных соединений (`ParallelConnections`);
* композицию последовательных и параллельных соединений.

В качестве базовых элементов должна быть возможность использовать:
* резисторы (`Resistor`);
* конденсаторы (`Capacitor`);
* инструкции (`Inductor`).

Реализовать модульные тесты, используя Gtest.

### Пример использования библиотеки
```cc
SequentialConnections conn;

conn.AddElement(new Resistor(1f));
conn.AddElement(new Resistor(2f));
conn.AddElement(new Capacitor(0.01f));
ParallelConnections * parallel = new ParallelConnections();
parallel->AddElement(new Resistor(1f));
parallel->AddElement(new Capacitor(0.01f));
parallel->AddElement(new Inductor(3f));
conn.AddElement(parallel);
conn.AddElement(new Resistor(1f));

Power power(1000f);
float resistance = conn.CalculateResistance(power);
```

## Рекомендации по выполнению
Все объекты следует наследовать от абстрактного класса
```cc
class Element {
 public:
  virtual float CalculateResistance(const Power&) const;
};
```

Класс `Power` представляет источник переменного тока. Частота передается в
конструктор класса `Power` и используется для расчета сопротивлений,
конденсаторов и инструкций.

Конструктор класса `Resistor` принимает значение своего сопротивления в Омах.

Конструктор класса `Capacitor` принимает значение своей емкости.

Конструктор класса `Inductor` принимает значение своей индуктивности.

Классы `SequentialConnections` и `ParallelConnections` так же следует
наследовать от класса `Element`. Так же, эти классы принимают аргумент метода
`AddElement` **во владение**, т.е. отвечают за его удаление. Чтобы избежать
лишних действий в деструкторе, рекомендуем использовать `std::unique_ptr` -
класс уникального владения объектом. Т.е. `std::unique_ptr` отвечает за удаление
объекта, на который он указывает.

Если использовать `std::unique_ptr`, то код изменится следующим образом:
```cc
SequentialConnections conn;

conn.AddElement(std::make_unique<Resistor>(1f));
conn.AddElement(std::make_unique<Resistor>(2f));
conn.AddElement(std::make_unique<Capacitor>(0.01f));
auto parallel = std::make_unique<ParallelConnections>();
parallel->AddElement(std::make_unique<Resistor>(1f));
parallel->AddElement(std::make_unique<Capacitor>(0.01f));
parallel->AddElement(std::make_unique<Inductor>(3f));
conn.AddElement(std::move(parallel));
conn.AddElement(std::make_unique<Resistor>(1f));

Power power(1000f);
float resistance = conn.CalculateResistance(power);
```

Пример добавления `std::unique_ptr` в контейнер `std::set`:
```cc
struct Example {

  void AddToSet(std::unique_ptr<Element> element) {
    set.insert(std::move(element));
  }
  std::set<std::unique_ptr<Element>> elements_;
};

Example e;
e.AddToSet(std::make_unique<Resistor>(12f));
auto parallel = std::make_unique<ParallelConnections>();
e.AddToSet(std::move(parallel));
```
