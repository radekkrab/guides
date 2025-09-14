Вот 8 наиболее часто используемых поведенческих паттернов в PHP с простыми примерами:

## 1. Strategy (Стратегия)
**Назначение**: Определяет семейство алгоритмов и делает их взаимозаменяемыми

```php
interface PaymentStrategy {
    public function pay($amount);
}

class CreditCardPayment implements PaymentStrategy {
    public function pay($amount) {
        echo "Paying $amount via Credit Card\n";
    }
}

class PayPalPayment implements PaymentStrategy {
    public function pay($amount) {
        echo "Paying $amount via PayPal\n";
    }
}

class PaymentContext {
    private $strategy;
    
    public function setStrategy(PaymentStrategy $strategy) {
        $this->strategy = $strategy;
    }
    
    public function executePayment($amount) {
        $this->strategy->pay($amount);
    }
}

// Использование
$payment = new PaymentContext();
$payment->setStrategy(new CreditCardPayment());
$payment->executePayment(100);

$payment->setStrategy(new PayPalPayment());
$payment->executePayment(200);
```

## 2. Observer (Наблюдатель)
**Назначение**: Создает механизм подписки для уведомления объектов об изменениях

```php
interface Observer {
    public function update($data);
}

interface Subject {
    public function attach(Observer $observer);
    public function detach(Observer $observer);
    public function notify();
}

class NewsPublisher implements Subject {
    private $observers = [];
    private $news;
    
    public function attach(Observer $observer) {
        $this->observers[] = $observer;
    }
    
    public function detach(Observer $observer) {
        $key = array_search($observer, $this->observers);
        if ($key !== false) {
            unset($this->observers[$key]);
        }
    }
    
    public function notify() {
        foreach ($this->observers as $observer) {
            $observer->update($this->news);
        }
    }
    
    public function setNews($news) {
        $this->news = $news;
        $this->notify();
    }
}

class EmailSubscriber implements Observer {
    public function update($data) {
        echo "Email: New news - $data\n";
    }
}

class SMSSubscriber implements Observer {
    public function update($data) {
        echo "SMS: Breaking news - $data\n";
    }
}

// Использование
$publisher = new NewsPublisher();
$publisher->attach(new EmailSubscriber());
$publisher->attach(new SMSSubscriber());

$publisher->setNews("PHP 8.4 released!");
```

## 3. Command (Команда)
**Назначение**: Инкапсулирует запрос как объект

```php
interface Command {
    public function execute();
}

class Light {
    public function turnOn() { echo "Light is ON\n"; }
    public function turnOff() { echo "Light is OFF\n"; }
}

class LightOnCommand implements Command {
    private $light;
    
    public function __construct(Light $light) {
        $this->light = $light;
    }
    
    public function execute() {
        $this->light->turnOn();
    }
}

class LightOffCommand implements Command {
    private $light;
    
    public function __construct(Light $light) {
        $this->light = $light;
    }
    
    public function execute() {
        $this->light->turnOff();
    }
}

class RemoteControl {
    private $command;
    
    public function setCommand(Command $command) {
        $this->command = $command;
    }
    
    public function pressButton() {
        $this->command->execute();
    }
}

// Использование
$light = new Light();
$remote = new RemoteControl();

$remote->setCommand(new LightOnCommand($light));
$remote->pressButton();

$remote->setCommand(new LightOffCommand($light));
$remote->pressButton();
```

## 4. Iterator (Итератор)
**Назначение**: Предоставляет способ последовательного доступа к элементам коллекции

```php
class BookCollection implements Iterator {
    private $books = [];
    private $position = 0;
    
    public function addBook($book) {
        $this->books[] = $book;
    }
    
    public function current() {
        return $this->books[$this->position];
    }
    
    public function key() {
        return $this->position;
    }
    
    public function next() {
        $this->position++;
    }
    
    public function rewind() {
        $this->position = 0;
    }
    
    public function valid() {
        return isset($this->books[$this->position]);
    }
}

// Использование
$collection = new BookCollection();
$collection->addBook("PHP Basics");
$collection->addBook("Design Patterns");
$collection->addBook("Clean Code");

foreach ($collection as $book) {
    echo $book . "\n";
}
```

## 5. Template Method (Шаблонный метод)
**Назначение**: Определяет скелет алгоритма, позволяя подклассам переопределять шаги

```php
abstract class DataProcessor {
    // Шаблонный метод
    final public function process() {
        $this->readData();
        $this->processData();
        $this->saveData();
    }
    
    abstract protected function readData();
    abstract protected function processData();
    
    protected function saveData() {
        echo "Saving data to database...\n";
    }
}

class CSVProcessor extends DataProcessor {
    protected function readData() {
        echo "Reading data from CSV file...\n";
    }
    
    protected function processData() {
        echo "Processing CSV data...\n";
    }
}

class XMLProcessor extends DataProcessor {
    protected function readData() {
        echo "Reading data from XML file...\n";
    }
    
    protected function processData() {
        echo "Processing XML data...\n";
    }
}

// Использование
$csvProcessor = new CSVProcessor();
$csvProcessor->process();

$xmlProcessor = new XMLProcessor();
$xmlProcessor->process();
```

## 6. State (Состояние)
**Назначение**: Позволяет объекту изменять поведение при изменении состояния

```php
interface OrderState {
    public function proceedToNext(OrderContext $context);
    public function toString();
}

class NewOrderState implements OrderState {
    public function proceedToNext(OrderContext $context) {
        echo "Order is being processed...\n";
        $context->setState(new ProcessingState());
    }
    
    public function toString() { return "New"; }
}

class ProcessingState implements OrderState {
    public function proceedToNext(OrderContext $context) {
        echo "Order is being shipped...\n";
        $context->setState(new ShippedState());
    }
    
    public function toString() { return "Processing"; }
}

class ShippedState implements OrderState {
    public function proceedToNext(OrderContext $context) {
        echo "Order is delivered!\n";
        $context->setState(new DeliveredState());
    }
    
    public function toString() { return "Shipped"; }
}

class DeliveredState implements OrderState {
    public function proceedToNext(OrderContext $context) {
        echo "Order already delivered\n";
    }
    
    public function toString() { return "Delivered"; }
}

class OrderContext {
    private $state;
    
    public function __construct() {
        $this->state = new NewOrderState();
    }
    
    public function setState(OrderState $state) {
        $this->state = $state;
    }
    
    public function proceed() {
        $this->state->proceedToNext($this);
    }
    
    public function getState() {
        return $this->state->toString();
    }
}

// Использование
$order = new OrderContext();
echo $order->getState() . "\n"; // New

$order->proceed();
echo $order->getState() . "\n"; // Processing

$order->proceed();
echo $order->getState() . "\n"; // Shipped
```

## 7. Chain of Responsibility (Цепочка обязанностей)
**Назначение**: Передает запрос по цепочке обработчиков

```php
abstract class Logger {
    const INFO = 1;
    const DEBUG = 2;
    const ERROR = 3;
    
    protected $level;
    protected $nextLogger;
    
    public function setNextLogger(Logger $nextLogger) {
        $this->nextLogger = $nextLogger;
    }
    
    public function logMessage($level, $message) {
        if ($this->level <= $level) {
            $this->write($message);
        }
        
        if ($this->nextLogger !== null) {
            $this->nextLogger->logMessage($level, $message);
        }
    }
    
    abstract protected function write($message);
}

class ConsoleLogger extends Logger {
    public function __construct($level) {
        $this->level = $level;
    }
    
    protected function write($message) {
        echo "Console: $message\n";
    }
}

class FileLogger extends Logger {
    public function __construct($level) {
        $this->level = $level;
    }
    
    protected function write($message) {
        echo "File: $message\n";
    }
}

class ErrorLogger extends Logger {
    public function __construct($level) {
        $this->level = $level;
    }
    
    protected function write($message) {
        echo "Error: $message\n";
    }
}

// Использование
$consoleLogger = new ConsoleLogger(Logger::INFO);
$fileLogger = new FileLogger(Logger::DEBUG);
$errorLogger = new ErrorLogger(Logger::ERROR);

$consoleLogger->setNextLogger($fileLogger);
$fileLogger->setNextLogger($errorLogger);

$consoleLogger->logMessage(Logger::INFO, "This is an information");
$consoleLogger->logMessage(Logger::ERROR, "This is an error");
```

## 8. Mediator (Посредник)
**Назначение**: Уменьшает связанность классов, организуя их взаимодействие через посредника

```php
interface Mediator {
    public function sendMessage($message, User $user);
}

class ChatRoom implements Mediator {
    public function sendMessage($message, User $user) {
        $time = date('H:i');
        $sender = $user->getName();
        echo "[$time] $sender: $message\n";
    }
}

class User {
    private $name;
    private $mediator;
    
    public function __construct($name, Mediator $mediator) {
        $this->name = $name;
        $this->mediator = $mediator;
    }
    
    public function getName() {
        return $this->name;
    }
    
    public function send($message) {
        $this->mediator->sendMessage($message, $this);
    }
}

// Использование
$chatRoom = new ChatRoom();

$user1 = new User("Alice", $chatRoom);
$user2 = new User("Bob", $chatRoom);

$user1->send("Hi Bob!");
$user2->send("Hello Alice!");
```

Эти поведенческие паттерны помогают организовать взаимодействие между объектами, делая систему более гибкой и легко расширяемой.
