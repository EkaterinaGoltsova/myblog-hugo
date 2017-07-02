+++
categories = [
	"testing"
]
date = "2017-06-25T18:30:25+07:00"
title = "TDD и BDD с примерами"

+++

Многие разработчики очень любят пренебрегать тестированием собственных задач. Признайтесь, отдавали ли вы задачи в тест, при этом практически не проверяя их? А потом доделывали и опять отдавали, доделывали и отдавали.. А ведь лучше вас unit тестирование никто не проведет, ведь лучше всего знаете и понимаете код — именно вы. Если программист захочет, то с легкостью придумает и воспроизведет граничные ситуации и сразу же сможет их проверить и пофиксить. Роль тестирования сложно недооценить, в наше время умненькие дяденьки даже методологии разработки придумали, главным смыслом которых является написание тестов, а потом уже и самого кода. Рассмотрим на примере, что значит TDD и BDD, а также как выглядят код и тесты, полученные в результате разработки с помощью этих методологий.

Начнем с небезызвестной методологии под названием *TDD (Test-driven development)*. В соответствии с ее принципами, сначала разработчик пишет тест, затем заставляет тест работать, далее рефакторит получившийся код и так по кругу, пока не будут реализованы все необходимые функции. Посмотрим, как выглядит разработка через тестирование на примере разработки калькулятора. «О, нет, опять калькулятор! Когда же в статьях будут реалистичные примеры?» - воскликните вы. Реалистичных примеров не обещаю в рамках этой статьи, а вот калькулятор будет работать не просто с числами, а с калориями. Калькуляторы калорий для человеческих особей существуют, а вот для бегемотиков нет. Исправим это недоразумение.  Для написания unit тестов воспользуемся фреймовроком PHPUnit, следовательно, тестируемый код будем писать на php. 

Итак, есть тривиальный пример - бегемотик желает знать, сколько грамм он наберет, если съест  14 килограмм молодого австралийского крокодила. Опытные британские диетологи утверждают, что после такого обеда бегемотик наберет 252 грамма жира. Проверим!

```php
class Hippopotamus_Fitness_CalculatorTest extends PHPUnit_Framework_TestCase
{

	private $_calculator;

	public function setUp()
	{	
		$this->_calculator = $calculator = new  Hippopotamus_Fitness_Calculator();
	}

	public function testGetWeightGainAfterDinnerCrocodile()
	{
		$weightGain = $this->_calculator->getWeightGain('crocodile', 14);
		$this->assertEquals(252, $weightGain);
	}
}
```
Для того, чтобы похудеть, бегемотикам необходимо заниматься спортом. Африканские гроссмейстеры утверждают, что 60-минутная игра в шахматы помогает сжечь 174 грамма жира. Коду мы не верим, код мы проверим! 

```php
	public function testGetWeightReductionAfterPlayingChess()
	{
		$weightReduction = $this->_calculator->getWeightReduction('chess', 60);
		$this->assertEquals(174, $weightReduction);
	}
```
Для полноты картины, попробуем посчитать, сколько грамм сбросит бегемотик, если часок драгой поиграет в доту2. 
```php
	public function testGetWeightReductionAfterUndefinedGame()
	{
		$weightReduction = $this->_calculatorgetWeightReduction('dota2', 60);
		$this->assertEquals(null, $weightReduction);
	}
```
Эх, нисколько! Потому что бегемотики в доту не играют.

Запускаем тесты и видим, что все тесты проваливаются. Пока все идет по плану! 

Ну а теперь пришло время заставить тесты работать и написать сам фитнес калькулятор для бегемотиков. 

Давайте для начала, для простоты изложения, выделим все необходимые данные в отдельный класс и будем использовать его при разработке сценариев. Это не самое лучшее архитектурное решение, но остановимся на этом варианте.
```php
class Hippopotamus_Fitness_Calories
{

    /**
     * Количество калорий, сжигаемых за 1 минуту игры
     *
     * @var array
     */
	CONST COUNT_IN_GAMES = [
		'hide_and_seek' => 132,
		'сatch-up' => 432,
		'chess' => 87 
	];

	
    /**
     * Количество калорий в 1 кг продукта
     *
     * @var array
     */
	CONST COUNT_IN_FOODS = [
		'grass' => 21,
		'water_plant' => 32,
		'crocodile' => 540
	];

    /**
     * Количество калорий, необходимых для сжигания 1 грамма жира
     *
     * @var int
     */
	CONST СOUNT_FOR_BURN_FAT = 30;
}
```
А теперь начнем писать код, и будем писать до тех пор, пока все тесты не будут зелеными. Вот что у меня получилось. 
```php
class Hippopotamus_Fitness_Calculator
{

	public function getWeightGain($food, $weight)
	{
		if (!isset(Hippopotamus_Fitness_Calories::COUNT_IN_FOODS[$food])) {
			return null;
		}

		return (Hippopotamus_Fitness_Calories::COUNT_IN_FOODS[$food] * $weight) / 		 
				Hippopotamus_Fitness_Calories::СOUNT_FOR_BURN_FAT;
	}


	public function getWeightReduction($game, $time)
	{
		if (!isset(Hippopotamus_Fitness_Calories::COUNT_IN_GAMES[$game])) {
			return null;
		}

		return (Hippopotamus_Fitness_Calories::COUNT_IN_GAMES[$game] * $time) / 
			Hippopotamus_Fitness_Calories::СOUNT_FOR_BURN_FAT;
	}
}
```
Запускаем тесты и видим:

```
Time: 183 ms, Memory: 14.25Mb

OK (3 tests, 3 assertions)
```

Все тесты зеленые! Теперь, по всем канонам, код необходимо порефакторить и сделать читаемым, но дабы статья не разрасталась до немыслимых размеров, на этом мы остановимся. Давайте лучше сформулируем основные преимущества и недостатки этой методологии. 

Итак, основные преимущества TDD:
 + программист продумывает детали и интерфейс еще до реализации, что помогает ему абстрагироваться и посмотреть на функциональность со стороны пользователя.
 + если есть тесты, программист может менять легаси-код, не боясь что-то отвалить (если и отвалит, тесты скажут ему об этом).
 + уменьшаются ошибки в коде, дефекты, время отладки и т.д. и т.п.

Без дегтя нет и меда, поэтому пара слов о недостатках:
 + в некоторых случаях увеличивается время разработки
 + unit тесты обычно пишутся тем же человеком, который пишет и тестируемый код. Если разработчик не разобрался в требованиях, то ошибки будут как в тестах, так и в коде. 
 + не всегда руководство соглашается понимать, что tdd — это круто и стоит увеличивать расходы на разработку на этапах внедрения этой практики. 

Теперь рассмотрим более ~~модный~~ молодой подход *BDD - Behaviour Driven Development*. Он тоже предполагает написание сначала тестов, а потом кода, но с некоторыми отличием в подходе написания тестов. Ой, не тестов! А сценариев. В этом подходе не существует слова «тест». Все, что мы привыкли считать тестом, здесь считается сценарием или поведением. Поэтому исключаем из названий функций слово «test» и пишем названия понятным даже для пм-ов языком. Название шагов сценария должны показывать, что должен делать этот сценарий, какое у него поведение. 

Итак, поведенческие сценарии будем писать с использованием Behat. Сначала опишем тестируемую функцию:

```
Feature: getWeightReduction
  In order to calculate weight loss after games
  As a hippopotamus
  I need to be able to know how many grams will help to lose a different kind of sport activity
```
Теперь опишем шаги сценария:
```
Scenario: Calculation of the loss of kilograms during a game of chess within an hour
  Given calculator 
  And I have a game named "chess"
  And I have a time - 60 minutes
  When I call getWeightReduction
  Then I should get: 174 grams 
```

Запустим наш сценарий:

![Images](https://ekaterinagoltsova.github.io/img/testing/1.png)

Даже на этом этапе мы уже можем запускать тесты, которые проваливаются. Заметьте, что умненький behat выдает шаблон шагов, которые необходимо реализовать. Реализуем описанный сценарий с помощью behat:

```php
class FeatureContext extends BehatContext
{

    private $_calculator;

    private $_output;

    /**
     * @Given /^calculator$/
     */
    public function iCreateCalculator()
    {
        $this->_calculator = new Hippopotamus_Fitness_Calculator();
    }

    /**
     * @Given /^I have a game named "([^"]*)"$/
     */
    public function iHaveAGame($arg1)
    {
        $this->_calculator->setGame($arg1);
    }

    /**
     * @Given /^I have a time - (\d+) minutes$/
     */
    public function iHaveATime($arg1)
    {
        $this->_calculator->setTime($arg1);
    }

    /**
     * @When /^I call getWeightReduction$/
     */
    public function iCallGetWeightreduction()
    {
        $this->_output = $this->_calculator->getWeightReduction();
    }

    /**
     * @Then /^I should get: (\d+) grams$/
     */
    public function iShouldGetGrams($arg1)
    {
         if ((int)$arg1 !== $this->_output) {
            throw new Exception("Actual grams is:\n" . $this->_output);
         }
    }
}
```
Запускаем наш сценарий, который также «красный». Заставим его работать. Будем использовать все тот же класс с константами, что и в первом случае. 
```
class Hippopotamus_Fitness_Calculator
{

	private $game;

	private $time;

	public function setGame($game)
	{
		$this->game = $game;
	}

	public function setTime($time)
	{
		$this->time = $time;
	}

	public function getWeightReduction()
	{
		if (!isset(Hippopotamus_Fitness_Calories::COUNT_IN_GAMES[$this->game])) {
			return null;

		}

		return (Hippopotamus_Fitness_Calories::COUNT_IN_GAMES[$this->game] * $this->time) / 
			Hippopotamus_Fitness_Calories::СOUNT_FOR_BURN_FAT;
	}

}
```
Снова запускаем сценарий и видим следующий результат. 

![Images](https://ekaterinagoltsova.github.io/img/testing/2.png)

Таким образом, любой человек может описать множество сценариев:
```
Scenario: Calculation of the loss of kilograms during a game of chess within an hour
  Given calculator 
  And I have a game named "chess"
  And I have a time - 60 minutes
  When I call getWeightReduction
  Then I should get: 174 grams 

Scenario: Calculation of the loss of kilograms during a game of chess within an hour
  Given calculator 
  And I have a game named "hide_and_seek"
  And I have a time - 30 minutes
  When I call getWeightReduction
  Then I should get: 132 grams 

Scenario: Calculation of the loss of kilograms during a game of chess within an hour
  Given calculator 
  And I have a game named "catch-up"
  And I have a time - 15 minutes
  When I call getWeightReduction
  Then I should get: 216 grams
```
Лично мне больше нравится результат после TDD разработки, но как говорится, для определенных целей определенные инструменты. 

Преимущества BDD:
*   сценарии понятны не только вам, но и менеджерам и другим далеким от разработки людям
*   сценарии описывают, как конечные пользователи будут использовать системы
*   на выходе получается почти готовая отличная документация

Недостатки BDD:
*   все те же, что и у TDD ;) 

Свое изложение мне пора заканчивать, но нами остались нетронуты еще ADDT, DDT, KDT и ODT .. о которых обязательно будет отдельная статья. Всем добра!
