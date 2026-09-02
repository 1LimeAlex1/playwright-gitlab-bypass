# Playwright E2E UI Automation & Advanced Security Bypass

Автоматизированный сквозной сценарий (End-to-End) тестирования формы регистрации веб-ресурса GitLab с интеграцией кастомного патчинга браузера и продвинутой имитацией человеческого поведения для обхода систем антибот-фильтрации (Cloudflare Turnstile, Akamai).

## 🚀 Описание проекта
Тест написан на **Python** с использованием фреймворка **Playwright**. Скрипт осуществляет полный цикл пользовательских действий в абсолютно чистом, изолированном контексте (Incognito) без использования закешированных сессий:
1. Инициализирует защищенный браузер с подменой системных отпечатков, флагов автоматизации.
2. Осуществляет переход напрямую на платформу GitLab и обрабатывает динамические баннеры куки-файлов.
3. Нажимает на кнопку перехода к странице авторизации (`Sign in`). 
4. В процессе ожидания открытия страницы логина скрипт проходит фоновую антифрод-проверку безопасности **Cloudflare Turnstile**, используя алгоритмы имитации действий человека.
5. После успешного открытия страницы авторизации совершает переход по ссылке на форму создания аккаунта (`Register now`).
6. Заполняет поля формы уникальными тестовыми данными (`puksrenk`) с помощью посимвольного ввода текста, имитируя реальную скорость печати.
7. Отправляет запрос на бэкенд и валидирует ошибки уникальности данных (`Username has already been taken`).

## 🛠️ Технический стек и особенности
* **Core:** Python 3.14+, Playwright (Sync API)
* **Bypass Engine:** Camoufox (глубокий патчинг Chromium в памяти, скрытие `navigator.webdriver`, генерация Canvas/Audio фингерпринтов).
* **Human-like Behavior:** 
  * Включение флага `humanize` для симуляции естественных траекторий движения курсора.
  * Посимвольный ввод данных (`human_type`) со случайными задержками (джиттером) между нажатиями клавиш в диапазоне от 0.06 до 0.18 секунд для обхода поведенческого анализа (Behavioral Analysis).
  * Динамические рандомизированные паузы (`human_delay`) вместо жёстких статических задержек.
* **Стабильность среды:** Полный отказ от статических `time.sleep()` в пользу динамических ожиданий элементов через `wait_for(state="visible")` с увеличенными таймаутами до 15 секунд. Это предотвращает "мигание" тестов (*flaky tests*) при нестабильном интернет-соединении.

## 📋 Исходный код скрипта
Реализация сквозного автотеста (Production-Ready версия):

```python
import re
import time
import random
from camoufox import Camoufox

def human_delay(min_sec=1.5, max_sec=3.5):
    """Создает случайную паузу для имитации задумчивости человека."""
    time.sleep(random.uniform(min_sec, max_sec))

def human_type(element, text):
    """Посимвольный ввод текста с реалистичными задержками между нажатиями."""
    for char in text:
        element.type(char)
        time.sleep(random.uniform(0.06, 0.18))

def run_perfect_undetected_test():
    print("\n" + "="*50)
    print("ТЕСТ: ОБХОД КАПЧИ В ИНКОГНИТО")
    print("="*50)
    
    config = {"humanize": True}
    
    print("Шаг 1: Инициализация защищенного браузера Camoufox...")
    # ФИКС: Задаем конкретное разрешение окна, чтобы вёрстка не съезжала
    with Camoufox(headless=False, window=(1920, 1080), **config) as browser:
        page = browser.new_page()
        
        print("Шаг 2: Переход на главную страницу GitLab...")
        page.bring_to_front()
        page.goto("https://about.gitlab.com", wait_until="domcontentloaded")
        
        human_delay(4.0, 7.0)
        
        print("Попытка закрыть Cookie...")
        try:
            page.locator(".onetrust-pc-dark-filter").click(timeout=5000)
            page.get_by_role("button", name=re.compile("Accept all cookies", re.IGNORECASE)).click(timeout=2000)
            print(" Баннер Cookie успешно закрыт!")
        except Exception:
            try:
                page.get_by_role("button", name="Accept All Cookies").click(timeout=3000)
                print(" Баннер Cookie успешно закрыт через запасной локатор!")
            except Exception:
                print(" Баннер Cookie не найден или уже закрыт.")
            
        print("Шаг 3: Нажатие по кнопке входа 'Sign in'...")
        human_delay(2.0, 4.0)
        
        sign_in_btn = page.get_by_role("link", name="Sign in")
        sign_in_btn.wait_for(state="visible", timeout=15000)
        sign_in_btn.click()
        
        print("Шаг 4: Переход со страницы логина на форму регистрации...")
        register_link = page.get_by_role("link", name="Register now")
        register_link.wait_for(state="visible", timeout=15000)
        
        human_delay(1.5, 3.0)
        register_link.click()
        
        print("Шаг 5: Заполнение полей формы регистрации...")
        first_name = page.get_by_test_id("new-user-first-name-field")
        first_name.wait_for(state="visible", timeout=15000)
        
        human_delay(2.0, 3.5)
        
        first_name.click()
        human_type(first_name, "Test")
        human_delay(0.3, 0.7)
        
        last_name = page.get_by_test_id("new-user-last-name-field")
        last_name.click()
        human_type(last_name, "Testoviy")
        human_delay(0.5, 1.2)
        
        print("   -> Ввод заведомо занятого логина 'puksrenk'...")
        username = page.get_by_test_id("new-user-username-field")
        username.click()
        human_type(username, "puksrenk")
        human_delay(0.4, 0.9)
        
        email = page.get_by_test_id("new-user-email-field")
        email.click()
        human_type(email, "testikus@mail.ru")
        human_delay(0.6, 1.1)
        
        password = page.get_by_test_id("new-user-password-field")
        password.click()
        human_type(password, "ExamplePassword123!")
        
        print("Шаг 6: Нажатие по кнопке 'Continue' для завершения...")
        human_delay(1.5, 2.5)
        
        continue_button = page.get_by_role("button", name="Continue", exact=True)
        continue_button.click()
        
        print("Ожидание ответа бэкенда GitLab...")
        error_msg = page.get_by_text("Username has already been taken").first
        
        try:
            error_msg.wait_for(state="visible", timeout=10000)
            print("\n✅ Успешно! Капча Turnstile пройдена автотестом!")
            print("   Бэкенд GitLab вернул ошибку: 'Username has already been taken'!")
        except Exception:
            print("\n❌ РЕЗУЛЬТАТ: Ошибка! Целевой текст не появился.")
            
        print("="*50 + "\n")
        human_delay(3.0, 5.0)

if __name__ == "__main__":
    run_perfect_undetected_test()
```

## 🎥 Демонстрация работы
Видеозапись успешного автономного выполнения автотеста в режиме инкогнито:
[Смотреть видео демонстрации](https://youtu.be/u-paUc_lW-Y)
