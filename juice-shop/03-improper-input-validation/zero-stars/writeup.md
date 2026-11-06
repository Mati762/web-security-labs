# Write-up: Zero Stars (OWASP Juice Shop ⭐)

* **Platforma:** OWASP Juice Shop (Docker localhost)
* **Kategoria:** Improper Input Validation / Ominięcie walidacji po stronie klienta
* **CWE:** CWE-20 (Improper Input Validation)
* **Status:** Rozwiązane (Burp Suite Proxy)

---

### 1. Opis problemu
Formularz opinii klienta (*Customer Feedback*) pozwala wybrać ocenę w skali od 1 do 5 gwiazdek. Próba ustawienia wartości `0` bezpośrednio w przeglądarce blokuje przycisk wysyłania (`disabled`). Aplikacja polega jednak wyłącznie na ograniczeniach interfejsu graficznego i nie weryfikuje zakresu oceny po stronie serwera.

---

### 2. Kroki do odtworzenia (Proof of Concept)
1. Wypełniono formularz opinii w przeglądarce poprawnymi danymi (wybrano ocenę `1`, wpisano komentarz i rozwiązano captchę).
2. W **Burp Suite** w zakładce **Proxy $\rightarrow$ Intercept** włączono przechwytywanie ruchu (*Intercept is on*).
3. Kliknięto przycisk **Submit** na stronie.
4. Przechwycono żądanie `POST /api/Feedbacks/` zawierające obiekt JSON:
   ```json
   {
     "UserId": 2,
     "captchaId": 0,
     "captcha": "12",
     "comment": "Test",
     "rating": 1
   }
   ```
5. W oknie Burp Suite zmieniono wartość parametru `"rating"` z `1` na `0` i puszczono pakiet dalej przyciskiem **Forward**.
6. Serwer zaakceptował żądanie, zwracając kod `201 Created`, i zapisał opinię z oceną 0 gwiazdek w bazie danych.

---
### Alternatywna metoda (manipulacja DOM w DevTools)
Na samym początku próbowałem przesłać opinię z zerową liczbą gwiazdek za pomocą narzędzi deweloperskich przeglądarki (`Zbadaj`) modyfikując minimalną przyjmowaną wartość, po tym kroku w zakres ocen udało się wybrać zero. Problem jednak napotkałem z przyciskiem **Submit**, który po ustawieniu oceny na zero dezaktywował się. Zanim uporałem się z przyciskiem udało mi się przesłać opinię przechwytując wychodzące żądanie w Burp Suite. Jednakże, później przyjrzałem sie temu raz jeszcze i zauważyłem, że wystarczy usunąć atrybut `disabled="true"` ze znacznika `<button>` w kodzie HTML przycisku przesyłania. Po zmodyfikowaniu stanu przycisku, formularz został wysłany.

---
### 3. Wpływ na bezpieczeństwo (Impact)
Opieranie reguł biznesowych wyłącznie na walidacji frontendowej (HTML/JavaScript) pozwala każdemu użytkownikowi z narzędziem typu proxy na przesłanie dowolnych, nieprawidłowych danych bezpośrednio do API. W realnych aplikacjach podobny błąd umożliwia np. zamawianie ujemnej liczby produktów lub manipulację cenami.

---

### 4. Proponowana naprawa
1. **Walidacja po stronie serwera (Backend):** Endpoint `/api/Feedbacks/` musi bezwzględnie weryfikować przychodzące dane przed zapisem do bazy (sprawdzenie, czy `rating` jest liczbą całkowitą z przedziału `<1, 5>`). W przypadku wartości spoza zakresu serwer powinien odrzucić żądanie kodem `400 Bad Request`.
