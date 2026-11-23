# Write-up: Missing Encoding (OWASP Juice Shop ⭐)

* **Platforma:** OWASP Juice Shop (Docker localhost)
* **Kategoria:** Security Misconfiguration / Lack of URL Encoding
* **CWE:** CWE-172 (Encoding Error)
* **Status:** Rozwiązane (manualnie / DevTools)

---

### 1. Opis problemu
W sekcji *Photo Wall* jeden z obrazków nie ładuje się na stronie i wyświetla domyślną ikonę błędu. Wstępna analiza w DevTools wykazała, że nazwa pliku w atrybucie `src` zawiera znak `#` (`wytłumaczenie#zamieszanie.jpg`). Przeglądarka interpretuje znak `#` w adresie URL jako kotwicę (fragment identifier) i obcina nazwę pliku przed wysłaniem żądania do serwera, co skutkuje odpowiedzią `404 Not Found`.

---

### 2. Kroki do odtworzenia (Proof of Concept)
1. Przejrzano sekcję *Photo Wall* w aplikacji i zlokalizowano nieładujący się obrazek.
2. Użyto opcji `Zbadaj` (Inspect Element), aby odczytać surową ścieżkę z atrybutu `src`:
   `assets/public/images/uploads/wytłumaczenie#zamieszanie.jpg`
3. Zastąpiono niezakodowany znak `#` jego odpowiednikiem w kodowaniu procentowym URL (`%23`):
   `assets/public/images/uploads/wytłumaczenie%23zamieszanie.jpg`
4. Wklejono pełny adres `http://localhost:3000/assets/public/images/uploads/wytłumaczenie%23zamieszanie.jpg` do paska adresu przeglądarki, co spowodowało poprawne pobranie obrazu i zaliczenie wyzwania.

---

### 3. Wpływ na bezpieczeństwo (Impact)
Brak automatycznego kodowania znaków specjalnych w ścieżkach zasobów generowanych przez aplikację prowadzi do błędów renderowania treści (DoS na poziomie interfejsu) oraz niepoprawnej obsługi zapytań HTTP przez serwer webowy.

---

### 4. Proponowana naprawa
1. **Automatyczne kodowanie URL (URL Encoding):** Wszystkie dynamicznie generowane ścieżki do plików statycznych powinny być przepuszczane przez funkcję kodującą znaki specjalne (np. `encodeURIComponent()` w JavaScript) przed wstawieniem ich do drzewa DOM.
2. **Standardy nazewnictwa plików:** Unikanie znaków specjalnych (takich jak `#`, `?`, `&`, spacje) w nazwach plików zapisywanych na serwerze.
