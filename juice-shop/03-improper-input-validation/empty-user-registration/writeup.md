# Write-up: Empty User Registration (OWASP Juice Shop ⭐⭐)

* **Platforma:** OWASP Juice Shop (Docker localhost)
* **Kategoria:** Improper Input Validation / Ominięcie walidacji po stronie klienta
* **CWE:** CWE-20 (Improper Input Validation)
* **Status:** Rozwiązane (Burp Suite Proxy)

---

### 1. Opis problemu
Formularz rejestracji nowego użytkownika w interfejsie przeglądarki blokuje przycisk wysyłania, dopóki pola adresu e-mail oraz hasła nie zostaną wypełnione. Po stronie API endpoint `/api/Users/` przyjmuje jednak żądanie zawierające puste ciągi znaków (`""`) dla kluczy `email` oraz `password` do wstępnego przetworzenia, zamiast natychmiast odrzucić je na etapie walidacji schematu wejściowego.

---

### 2. Kroki do odtworzenia (Proof of Concept)
1. Wypełniono formularz rejestracji dowolnymi danymi testowymi, aby odblokować przycisk zatwierdzający w interfejsie.
2. W **Burp Suite** włączono przechwytywanie ruchu (**Proxy $\rightarrow$ Intercept**) i wysłano formularz.
3. Przechwycono żądanie `POST /api/Users/` i zmieniono wartości parametrów `email`, `password` oraz `passwordRepeat` na puste napisy `""` (zachowując same klucze w strukturze JSON):
   ```http
   POST /api/Users/ HTTP/1.1
   Host: localhost:3000
   Content-Type: application/json

   {
     "email": "",
     "password": "",
     "passwordRepeat": "",
     "securityQuestion": {
       "id": 11,
       "question": "Twoja ulubiona książka?"
     },
     "securityAnswer": "Wiedzmin"
   }
   ```
4. Przesłano zmodyfikowany pakiet przyciskiem **Forward**. Aplikacja zaakceptowała strukturę pustego żądania (zaliczając wektor podatności), mimo że warstwa bazy danych finalnie odrzuciła zapis rekordu odpowiedzią `400 Bad Request` (`Invalid email/password cannot be empty`).

---

### 3. Wpływ na bezpieczeństwo (Impact)
Opieranie walidacji pól wymaganych wyłącznie na mechanizmach frontendowych pozwala na wysyłanie pustych lub niekompletnych obiektów bezpośrednio do API. W aplikacjach pozbawionych dodatkowych ograniczeń na poziomie bazy danych prowadzi to do tworzenia kont bez hasła lub zapisywania uszkodzonych rekordów użytkowników.

---

### 4. Proponowana naprawa
1. **Rygorystyczna walidacja schematu na backendzie:** Endpoint `/api/Users/` powinien przed dalszym przetwarzaniem weryfikować nie tylko samą obecność i typ pól (`string`), ale także ich minimalną długość po usunięciu białych znaków (np. `trim().length > 0`) oraz poprawność składni adresu e-mail (np. przy użyciu bibliotek walidujących takich jak Joi lub Zod).
