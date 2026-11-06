# Web Security Labs & Write-ups

Repozytorium zawierające moje notatki i raporty z nauki bezpieczeństwa aplikacji webowych, analizy podatności według standardu OWASP oraz pracy z narzędziami sieciowymi.

### Środowisko testowe
* **System:** Kali Linux (maszyna wirtualna VirtualBox)
* **Narzędzia:** Burp Suite Community Edition, DevTools
* **Poligony:** OWASP Juice Shop (uruchamiany lokalnie w Dockerze)

---

### Rozwiązane wyzwania (OWASP Juice Shop)

| Poziom | Nazwa zadania | Kategoria OWASP | Metoda | Raport |
| :---: | :--- | :--- | :---: | :---: |
| ⭐ | Error Handling | Security Misconfiguration | Test manualny | [Czytaj write-up](juice-shop/01-security-misconfiguration/error-handling/writeup.md) |
| ⭐ | DOM XSS | Cross-Site Scripting (XSS) | Test manualny | [Czytaj write-up](juice-shop/02-xss/dom-xss/writeup.md) |
| ⭐ | Bonus Payload | Cross-Site Scripting / HTML Injection | Test manualny | [Czytaj write-up](juice-shop/02-xss/bonus-payload/writeup.md) |
| ⭐ | Zero Stars | Improper Input Validation | Burp Suite Proxy | [Czytaj write-up](juice-shop/03-improper-input-validation/zero-stars/writeup.md) |
