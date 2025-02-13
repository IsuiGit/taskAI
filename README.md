Кусочно-генеративный подход к построению задач с помощью LLM

# Теоретико-методологическая база

### Почему LLM?

```
В: Может ли ИИ сгенерировать решаемую задачу?
О: Не без труда, но это возможно...
```

LLM (large language model) - это [языковая модель](https://ru.wikipedia.org/wiki/%D0%AF%D0%B7%D1%8B%D0%BA%D0%BE%D0%B2%D0%B0%D1%8F_%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C "Языковая модель"), состоящая из [нейронной сети](https://ru.wikipedia.org/wiki/%D0%9D%D0%B5%D0%B9%D1%80%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F_%D1%81%D0%B5%D1%82%D1%8C "Нейронная сеть") со множеством параметров (обычно миллиарды весовых коэффициентов и более), обученной на большом количестве неразмеченного текста с использованием [обучения без учителя](https://ru.wikipedia.org/wiki/%D0%9E%D0%B1%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D0%B5_%D0%B1%D0%B5%D0%B7_%D1%83%D1%87%D0%B8%D1%82%D0%B5%D0%BB%D1%8F "Обучение без учителя").[^1]

В рамках использования ИИ для обучения и генерации обучающих задач существуют две основополагающих проблемы: ограниченность LLM на уровне сложной логики и низкий уровень доверия к результатам решения частных задач. Однако, ИИ - мощный инструмент для генерации и компоновки текстов, что в сочетании с итеративным подходом позволяет генерировать полную и согласованную задачу.

### Теоретическая модель задачи

Задача - это ситуация, с явно заданной целью, начальными условиями и гипотетически возможная к решению, в рамках установленных полных или неполных начальных данных. Изначально правильно построенная задача имеет под собой вероятность решения, исходя из предположения о решении близких, к поставленной, задач.

Саму задачу можно разбить на следующие компоненты:
- **текст задачи** - человекопонятное словесное описание изначальной проблемы;
- **начальные данные** - условия, в рамках которых существует задача;
- **практическая область** - условия, в рамках которых задачу необходимо привести к разрешенному состоянию;
- **искомый результат** - условие выполнения поставленной задачи.

Все вышеперечисленные компоненты ясны человеку, но не факт, что ИИ сможет выстроить цепь взаимосвязанных действий, необходимых для решения, полагаясь только на указанные выше параметры. В связи с чем, для более точного описания вышеуказанную модель необходимо привести в параметрический вид, более наполненный как технически, так и логически.

### Параметрическая модель задачи

Пусть `Task` - модель задачи, с параметрами:
- `Tempale<topic, markers[], steps[]>` - шаблон для генерации полной задачи;
- `topic` - краткая постановка задачи;
- `markers[]` - набор "маркеров", согласно которым устанавливается практическая область поиска решения *(прим.: "Создание собственной графической библиотеки на Python", где `Python` и `графическая библиотека` - явные маркеры на практическую область для поиска решения)*
- `steps[]` - набор "шагов", необходимых для принятия решения;
- `text` - текст задачи;
- `hypothetical_solution` - решение, генерируемое ИИ в каждой итерации
- `solution` - полное, согласованное и подтвержденное решение задачи

Таким образом модель `T` можно определить как класс вида:

```
class <Task(
	Template<topic, markers[], steps[]>,
	text,  
	hypothetical_solution, 
	solution
)>
```

### Методология генерации задач

Пусть `P` - программа, принимающая на вход объекты класса `Task` и отправляющая запросы к ИИ. Для успешного применения генеративного подхода к процессу формирования задачи, необходимо выполнить следующие шаги:
1. Получение на вход объект класса `Task` с параметрами: `topic`, `markers[]`, `steps[]`, gри чем, параметры должны быть выбраны в соответствии с ожидаемой задачей и применимы в практическом поле **хотя-бы** гипотетического решения.
2. Обработка объекта с параметрами и отправка запроса на генерацию к ИИ; формирование `prompt` запроса из переданных в классе `Task` параметров *(прим.: `topic="создание ИИ с помощью Python"`, `markers=\["ИИ", "AI", "Python", "нейронные сети"]`, `steps=\["проверка совместимости компонент технической базы", "генерация простого кода", "проверка кода", "генерация текста задачи"]`)*.
3. Проверка полученного текста задачи на достоверность и полноту (прим.: переадресация полученного текста в другой ИИ, с целью проверки решаемости составленной задачи) с установленным порогом.
4. При достижении порога достоверности и полноты, объекту класса `Task` в поле `text` присваивается полученный текст задачи, а в поле `solution` - решение этой задачи. Если порог не был достигнут, то решение помещается в поле `hypothetical_solution`, а шаги 2 и 3 повторяются.

При варьировании уровня достоверности, набора маркеров и шагов, можно получать задачи различной сложности и уровня "открытости" к решению.

#### Псевдо-схема генерации

```
#pseudo code

func taskGenerate(T<Task>){
	prompt = promptGenerate(
		T.Template.topic, 
		T.Template.markers,
		T.Template.steps
	)
	text = textGenerate(prompt)
	solution = getSolution(text)
	if getScores(text, solution) > 0.9 {
		T.text = text
		T.solution = solution
		return T
	}
	else {
		taskGenerate(T<Task>)
	}
}

```

# Метрики оценки генерируемых задач

Для оценки удобоваримости сгенерированной задачи выставляются баллы в рамках списка метрик, после чего формируется рейтинг на общий уровень, по суммарным состояниям системы.

#### **Таблица метрик**

| Условное обозначение | Наименование                        | Рейтинг | Вес |
| -------------------- | ----------------------------------- | ------- | --- |
| ПУ                   | Полнота условий                     | 0-5     | 0,2 |
| РА                   | Разрешимость                        | 0-5     | 0,2 |
| СИП                  | Соответствие изначальным параметрам | 0-2     | 0,5 |
| ИНТ                  | Интерпретируемость                  | 0-1     | 1   |

`S = sum_p(pR*pW)` 

где `pR` - рейтинг параметра, `pW` - вес параметра.

#### Описание метрик

**Полнота условий** - наличие в задании всех необходимых *начальных условий* и данных из *практической области*.

**Разрешимость** - наличие у задачи фактического решения, не превышающего установленный порог сложности в системе.

**Соответствие изначальным параметрам** - сформированная задача удовлетворяет практической области (и её специализации) в полной мере, без "перехода" из одной в смежную или противоположную.

**Интерпретируемость** - возможность к понимаю задачи человеком и логичному поиску решения.

[^1]: [Wikipedia](https://ru.wikipedia.org/wiki/%D0%91%D0%BE%D0%BB%D1%8C%D1%88%D0%B0%D1%8F_%D1%8F%D0%B7%D1%8B%D0%BA%D0%BE%D0%B2%D0%B0%D1%8F_%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C)
