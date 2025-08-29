# Настройка рабочего пространства
## Руководства
*   [Настройка SSH](create-ssh.md)
## Установка Chocolatey (если ещё не установлен) PowerShell от имени администратора
```bash
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
## Включение необходимых компонентов Windows для WSL
```bash
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux -NoRestart
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -NoRestart
```
## Установка WSL и Ubuntu через Chocolatey
## Установка Docker (нужен для WSL2)
## Основные программы для PHP-разработки
```bash
choco install -y php --version=8.2.6
choco install -y wsl2 docker-desktop git composer mysql postman 
choco install -y mysql.workbench
choco install -y vscode nodejs
```
## Установку Ubuntu 22.04 лучше произвести через магазин microsoft
## Далее устанавливаем шторм Phpstorm

## После установки выполнить:
```bash
wsl --set-default-version 2"
wsl --set-version Ubuntu 2"
```
## Запустите Ubuntu из меню Пуск для завершения установки

## Редактировать php.ini
notepad C:\tools\php\php.ini
