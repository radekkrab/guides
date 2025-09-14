Вот 8 наиболее часто используемых порождающих паттернов в PHP с простыми примерами:

## 1. Singleton (Одиночка)
**Назначение**: Гарантирует, что класс имеет только один экземпляр

```php
class DatabaseConnection
{
    private static $instance = null;
    
    private function __construct() {}
    
    public static function getInstance()
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    // Предотвращаем клонирование
    private function __clone() {}
}

// Использование
$db = DatabaseConnection::getInstance();
```

## 2. Factory Method (Фабричный метод)
**Назначение**: Создание объектов без указания конкретного класса

```php
interface Logger
{
    public function log($message);
}

class FileLogger implements Logger
{
    public function log($message) {
        file_put_contents('log.txt', $message, FILE_APPEND);
    }
}

class LoggerFactory
{
    public static function createLogger($type = 'file')
    {
        switch ($type) {
            case 'file': return new FileLogger();
            default: throw new Exception("Unknown logger type");
        }
    }
}

// Использование
$logger = LoggerFactory::createLogger('file');
$logger->log("Test message");
```

## 3. Abstract Factory (Абстрактная фабрика)
**Назначение**: Создание семейств связанных объектов

```php
interface Button
{
    public function render();
}

interface Checkbox
{
    public function render();
}

class WindowsButton implements Button
{
    public function render() { return "Windows Button"; }
}

class WindowsCheckbox implements Checkbox
{
    public function render() { return "Windows Checkbox"; }
}

interface GUIFactory
{
    public function createButton(): Button;
    public function createCheckbox(): Checkbox;
}

class WindowsFactory implements GUIFactory
{
    public function createButton(): Button { return new WindowsButton(); }
    public function createCheckbox(): Checkbox { return new WindowsCheckbox(); }
}
```

## 4. Builder (Строитель)
**Назначение**: Пошаговое создание сложных объектов

```php
class QueryBuilder
{
    private $select = [];
    private $from;
    private $where = [];
    
    public function select(array $fields)
    {
        $this->select = $fields;
        return $this;
    }
    
    public function from($table)
    {
        $this->from = $table;
        return $this;
    }
    
    public function where($condition)
    {
        $this->where[] = $condition;
        return $this;
    }
    
    public function build()
    {
        return "SELECT " . implode(', ', $this->select) . 
               " FROM " . $this->from . 
               " WHERE " . implode(' AND ', $this->where);
    }
}

// Использование
$query = (new QueryBuilder())
    ->select(['id', 'name'])
    ->from('users')
    ->where('age > 18')
    ->build();
```

## 5. Prototype (Прототип)
**Назначение**: Создание объектов через клонирование

```php
class UserProfile
{
    public $name;
    public $email;
    
    public function __clone()
    {
        // Глубокая копия при необходимости
    }
}

$prototype = new UserProfile();
$prototype->name = "Default User";
$prototype->email = "default@example.com";

// Клонирование
$user1 = clone $prototype;
$user1->name = "John Doe";

$user2 = clone $prototype;
$user2->name = "Jane Smith";
```

## 6. Simple Factory (Простая фабрика)
**Назначение**: Упрощенная версия фабрики для создания объектов

```php
class PaymentProcessorFactory
{
    public static function create($type)
    {
        switch ($type) {
            case 'credit_card': return new CreditCardProcessor();
            case 'paypal': return new PayPalProcessor();
            case 'stripe': return new StripeProcessor();
            default: throw new Exception("Unknown payment type");
        }
    }
}

// Использование
$processor = PaymentProcessorFactory::create('paypal');
$processor->processPayment(100);
```

## 7. Object Pool (Пул объектов)
**Назначение**: Переиспользование объектов вместо создания новых

```php
class DatabaseConnectionPool
{
    private $pool = [];
    private $maxSize;
    
    public function __construct($maxSize = 10)
    {
        $this->maxSize = $maxSize;
    }
    
    public function getConnection()
    {
        if (!empty($this->pool)) {
            return array_pop($this->pool);
        }
        
        if (count($this->pool) < $this->maxSize) {
            return new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');
        }
        
        throw new Exception("Pool exhausted");
    }
    
    public function releaseConnection($connection)
    {
        if (count($this->pool) < $this->maxSize) {
            $this->pool[] = $connection;
        }
    }
}
```

## 8. Lazy Initialization (Ленивая инициализация)
**Назначение**: Отложенное создание объекта до момента его использования

```php
class HeavyService
{
    private $service = null;
    
    public function getService()
    {
        if ($this->service === null) {
            $this->service = $this->createHeavyService();
        }
        return $this->service;
    }
    
    private function createHeavyService()
    {
        // Тяжелая операция инициализации
        sleep(2);
        return new ExpensiveService();
    }
}

// Использование
$heavy = new HeavyService();
// Объект создается только здесь:
$service = $heavy->getService();
```

Эти паттерны помогают сделать код более гибким, поддерживаемым и тестируемым, решая распространенные проблемы создания объектов в PHP-приложениях.
