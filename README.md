# Playwright E2E UI Automation & Security Bypass

Автоматизированный сквозной сценарий (End-to-End) тестирования формы регистрации веб-ресурса GitLab с интеграцией кастомного патчинга браузера для обхода систем антибот-фильтрации.

## 🚀 Описание проекта
Тест написан на **Python** с использованием фреймворка **Playwright**. Скрипт осуществляет полный цикл пользовательских действий в абсолютно чистом, изолированном контексте (Incognito):
1. Инициализирует защищенный браузер с подменой системных отпечатков.
2. Проходит главную промо-страницу в обход триггеров **Cloudflare Turnstile**.
3. Автоматически обрабатывает динамические баннеры локализации (Cookie Consent) на английском языке.
4. Осуществляет переход по цепочке ссылок до формы создания аккаунта.
5. Заполняет поля формы валидными тестовыми данными и отправляет запрос на бэкенд, валидируя ошибки уникальности логина (`Username has already been taken`).

## 🛠️ Технический стек и особенности
* **Core:** Python 3.14+, Playwright (Sync API)
* **Bypass Engine:** Camoufox (глубокий патчинг Chromium в памяти, вырезание флагов автоматизации `navigator.webdriver`, генерация Canvas/Audio фингерпринтов)
* **Селекторы:** Тестирование через семантические роли (`get_by_role`) и уникальные инспекционные маркеры (`get_by_test_id`)
* **Логирование:** Кастомное пошаговое логирование выполнения сценария в CLI с цветовой индикацией статусов (`✅ Success` / `❌ Error`)

## 📋 Исходный код скрипта
Реализация сквозного автотеста со встроенными механизмами маскировки среды:

```python
import re
import time
from camoufox import Camoufox

def run_perfect_undetected_test():
    print("\n" + "="*50)
    print("🦊 ТЕСТ: ОБХОД КАПЧИ В ИНКОГНИТО")
    print("="*50)
    
    print("Шаг 1: Инициализация защищенного браузера Camoufox...")
    with Camoufox(headless=False) as browser:
        page = browser.new_page()
        
        print("Шаг 2: Переход на главную страницу GitLab...")
        page.bring_to_front()
        page.goto("https://about.gitlab.com/")
        
        print("10 секунд ожидания для прогрузки главной страницы")
        time.sleep(10)
        
        print("Попытка закрыть Cookie...")
        try:
            page.locator(".onetrust-pc-dark-filter").click(timeout=3000)
            page.get_by_role("button", name=re.compile("Accept all cookies", re.IGNORECASE)).click(timeout=3000)
            print(" Баннер Cookie успешно закрыт!")
        except Exception:
            try:
                page.get_by_role("button", name="Accept All Cookies").click(timeout=3000)
                print(" Баннер Cookie успешно закрыт через запасной локатор!")
            except Exception:
                print(" Баннер Cookie не найден или уже закрыт.")
            
        print("Шаг 3: Нажатие по кнопке входа 'Sign in'...")
        page.get_by_role("link", name="Sign in").click()
        
        print("Ожидание капчи перед переходом на страницу авторизации...")
        time.sleep(7)
        
        print("Шаг 4: Переход со страницы логина на форму регистрации...")
        page.get_by_role("link", name="Register now").click()
        time.sleep(5)
        
        print("Шаг 5: Заполнение полей формы регистрации...")
        page.get_by_test_id("new-user-first-name-field").fill("Test")
        page.get_by_test_id("new-user-last-name-field").fill("Testoviy")
        
        print("   -> Ввод заведомо занятого логина 'puksrenk'...")
        page.get_by_test_id("new-user-username-field").fill("puksrenk")
        
        page.get_by_test_id("new-user-email-field").fill("testikus@mail.ru")
        page.get_by_test_id("new-user-password-field").fill("ExamplePassword123!")
        
        print("Шаг 6: Нажатие по кнопке 'Continue' для завершения...")
        continue_button = page.get_by_role("button", name="Continue", exact=True)
        continue_button.click()
        
        print("Ожидание ответа бэкенда GitLab...")
        time.sleep(5)
        
        error_msg = page.get_by_text("Username has already been taken").first
        
        if error_msg.is_visible():
            print("\n✅ Успешно! Капча Turnstile пройдена автотестом в инкогнито !")
            print("   Робот прошёл сквозной сценарий, и бэкенд выдал ошибку: 'Username has already been taken'!")
        else:
            print("\n❌ РЕЗУЛЬТАТ: Ошибка! Целевой текст о занятом логине не найден.")
            
        print("="*50 + "\n")
        time.sleep(5)

if __name__ == "__main__":
    run_perfect_undetected_test()
```

## 🎥 Демонстрация работы
Видеозапись успешного автономного выполнения автотеста в режиме инкогнито (без использования закешированных профилей браузера и куков):
[Смотреть видео демонстрации] https://drive.google.com/file/d/1MmvBy9xTOt6puTR8ywKVvnMN7BvVjffy/view?usp=sharing
