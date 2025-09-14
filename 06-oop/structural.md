Вот 8 наиболее часто используемых структурных паттернов в PHP с простыми примерами:

## 1. Adapter (Адаптер)
**Назначение**: Преобразует интерфейс класса в другой интерфейс

```php
// Старый класс с несовместимым интерфейсом
class OldLogger {
    public function writeToFile($message) {
        file_put_contents('log.txt', $message, FILE_APPEND);
    }
}

// Новый интерфейс, который нам нужен
interface LoggerInterface {
    public function log($message);
}

// Адаптер
class LoggerAdapter implements LoggerInterface {
    private $oldLogger;
    
    public function __construct(OldLogger $oldLogger) {
        $this->oldLogger = $oldLogger;
    }
    
    public function log($message) {
        $this->oldLogger->writeToFile($message);
    }
}

// Использование
$oldLogger = new OldLogger();
$logger = new LoggerAdapter($oldLogger);
$logger->log("New message");
```

## 2. Decorator (Декоратор)
**Назначение**: Динамически добавляет новую функциональность объекту

```php
interface Coffee {
    public function getCost();
    public function getDescription();
}

class SimpleCoffee implements Coffee {
    public function getCost() { return 10; }
    public function getDescription() { return "Simple coffee"; }
}

class MilkDecorator implements Coffee {
    private $coffee;
    
    public function __construct(Coffee $coffee) {
        $this->coffee = $coffee;
    }
    
    public function getCost() {
        return $this->coffee->getCost() + 2;
    }
    
    public function getDescription() {
        return $this->coffee->getDescription() . ", milk";
    }
}

// Использование
$coffee = new SimpleCoffee();
$coffee = new MilkDecorator($coffee);
echo $coffee->getDescription(); // Simple coffee, milk
echo $coffee->getCost(); // 12
```

## 3. Facade (Фасад)
**Назначение**: Предоставляет простой интерфейс к сложной системе

```php
// Сложные подсистемы
class Database {
    public function connect() { /* ... */ }
    public function query($sql) { /* ... */ }
}

class Cache {
    public function set($key, $value) { /* ... */ }
    public function get($key) { /* ... */ }
}

class Logger {
    public function log($message) { /* ... */ }
}

// Фасад
class ApplicationFacade {
    private $db;
    private $cache;
    private $logger;
    
    public function __construct() {
        $this->db = new Database();
        $this->cache = new Cache();
        $this->logger = new Logger();
    }
    
    public function getUserData($userId) {
        $this->logger->log("Getting user $userId");
        
        // Проверяем кэш
        if ($data = $this->cache->get("user_$userId")) {
            return $data;
        }
        
        // Если нет в кэше, ищем в БД
        $data = $this->db->query("SELECT * FROM users WHERE id = $userId");
        $this->cache->set("user_$userId", $data);
        
        return $data;
    }
}

// Использование
$app = new ApplicationFacade();
$userData = $app->getUserData(123);
```

## 4. Composite (Компоновщик)
**Назначение**: Объединяет объекты в древовидные структуры

```php
interface Employee {
    public function getSalary();
}

class Developer implements Employee {
    private $salary;
    
    public function __construct($salary) {
        $this->salary = $salary;
    }
    
    public function getSalary() {
        return $this->salary;
    }
}

class Department implements Employee {
    private $employees = [];
    
    public function addEmployee(Employee $employee) {
        $this->employees[] = $employee;
    }
    
    public function getSalary() {
        $total = 0;
        foreach ($this->employees as $employee) {
            $total += $employee->getSalary();
        }
        return $total;
    }
}

// Использование
$dev1 = new Developer(5000);
$dev2 = new Developer(6000);

$department = new Department();
$department->addEmployee($dev1);
$department->addEmployee($dev2);

echo $department->getSalary(); // 11000
```

## 5. Proxy (Заместитель)
**Назначение**: Контролирует доступ к другому объекту

```php
interface Image {
    public function display();
}

class RealImage implements Image {
    private $filename;
    
    public function __construct($filename) {
        $this->filename = $filename;
        $this->loadFromDisk();
    }
    
    private function loadFromDisk() {
        echo "Loading image: $this->filename\n";
        // Тяжелая операция загрузки
    }
    
    public function display() {
        echo "Displaying image: $this->filename\n";
    }
}

class ImageProxy implements Image {
    private $realImage;
    private $filename;
    
    public function __construct($filename) {
        $this->filename = $filename;
    }
    
    public function display() {
        if ($this->realImage === null) {
            $this->realImage = new RealImage($this->filename);
        }
        $this->realImage->display();
    }
}

// Использование
$image = new ImageProxy("photo.jpg");
// Реальное изображение еще не загружено
$image->display(); // Загружается и отображается
```

## 6. Bridge (Мост)
**Назначение**: Разделяет абстракцию и реализацию

```php
// Реализация
interface Theme {
    public function getColor();
}

class DarkTheme implements Theme {
    public function getColor() { return "Dark Black"; }
}

class LightTheme implements Theme {
    public function getColor() { return "Off White"; }
}

// Абстракция
abstract class Page {
    protected $theme;
    
    public function __construct(Theme $theme) {
        $this->theme = $theme;
    }
    
    abstract public function getContent();
}

class AboutPage extends Page {
    public function getContent() {
        return "About page in " . $this->theme->getColor();
    }
}

class CareersPage extends Page {
    public function getContent() {
        return "Careers page in " . $this->theme->getColor();
    }
}

// Использование
$darkTheme = new DarkTheme();
$aboutPage = new AboutPage($darkTheme);
echo $aboutPage->getContent(); // About page in Dark Black
```

## 7. Flyweight (Приспособленец)
**Назначение**: Экономит память, разделяя общие данные

```php
class CharacterFlyweight {
    private $char;
    
    public function __construct($char) {
        $this->char = $char;
    }
    
    public function render($font) {
        return "Character: $this->char with font: $font";
    }
}

class FlyweightFactory {
    private $characters = [];
    
    public function getCharacter($char) {
        if (!isset($this->characters[$char])) {
            $this->characters[$char] = new CharacterFlyweight($char);
        }
        return $this->characters[$char];
    }
}

// Использование
$factory = new FlyweightFactory();
$charA = $factory->getCharacter('A');
$charB = $factory->getCharacter('B');
$charA2 = $factory->getCharacter('A'); // Возвращает существующий объект

echo $charA->render('Arial'); // Character: A with font: Arial
```

## 8. Data Mapper (Преобразователь данных)
**Назначение**: Переносит данные между объектами и БД

```php
class User {
    private $id;
    private $name;
    private $email;
    
    public function __construct($id, $name, $email) {
        $this->id = $id;
        $this->name = $name;
        $this->email = $email;
    }
    
    // Геттеры и сеттеры...
}

class UserMapper {
    private $pdo;
    
    public function __construct(PDO $pdo) {
        $this->pdo = $pdo;
    }
    
    public function findById($id) {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = :id");
        $stmt->execute(['id' => $id]);
        $data = $stmt->fetch();
        
        if (!$data) return null;
        
        return new User($data['id'], $data['name'], $data['email']);
    }
    
    public function save(User $user) {
        if ($user->getId()) {
            // Update
            $stmt = $this->pdo->prepare("UPDATE users SET name = :name, email = :email WHERE id = :id");
            $stmt->execute([
                'name' => $user->getName(),
                'email' => $user->getEmail(),
                'id' => $user->getId()
            ]);
        } else {
            // Insert
            $stmt = $this->pdo->prepare("INSERT INTO users (name, email) VALUES (:name, :email)");
            $stmt->execute([
                'name' => $user->getName(),
                'email' => $user->getEmail()
            ]);
            $user->setId($this->pdo->lastInsertId());
        }
    }
}
```

Эти структурные паттерны помогают организовать отношения между классами, делая систему более гибкой и легко расширяемой.
