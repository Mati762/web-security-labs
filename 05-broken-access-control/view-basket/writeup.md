# Write-up: View Basket (OWASP Juice Shop ⭐⭐)

* **Platforma:** OWASP Juice Shop (Docker localhost)
* **Kategoria:** Broken Access Control / IDOR (Insecure Direct Object Reference)
* **CWE:** CWE-639 (Authorization Bypass Through User-Controlled Key)
* **Status:** Rozwiązane (Burp Suite Proxy / Modyfikacja API)

---

### 1. Opis problemu
Aplikacja weryfikuje tożsamość użytkownika przy dostępie do koszyka zakupowego wyłącznie na podstawie parametru identyfikatora koszyka (`BasketId`) przekazywanego w ścieżce adresu URL. Warstwa backendowa nie weryfikuje, czy identyfikator zawarty w żądaniu odpowiada powiązanemu z nim tokenowi sesyjnemu obecnie zalogowanego użytkownika, co pozwala na podgląd zawartości koszyków innych osób.

---

### 2. Kroki do odtworzenia (Proof of Concept)
1. Zalogowano się na konto użytkownika i przejdź do widoku koszyka w aplikacji.
2. Za pomocą Burp Suite przechwycono żądanie typu GET wysyłane do REST API aplikacji: `GET /rest/basket/2 HTTP/1.1`
3. Zmodyfikowano parametr identyfikatora koszyka w adresie URI z `2` na `1`: `GET /rest/basket/1 HTTP/1.1`
4. Przesłano zmodyfikowane żądanie do serwera. Backend zwrócił obiekt JSON zawierający towary znajdujące się w koszyku przypisanym do konta o identyfikatorze `1` (koszyk administratora), co zaliczyło wyzwanie.

---

### 3. Wpływ na bezpieczeństwo (Impact)
Brak kontroli dostępu na poziomie obiektu (BOLA / IDOR) umożliwia zalogowanym użytkownikom odczytywanie prywatnych danych innych klientów sklepu (zawartości ich koszyków, adresów czy historii wybranych produktów), a w skrajnych przypadkach może prowadzić do modyfikacji lub czyszczenia cudzych zamówień.

---

### 4. Proponowana naprawa
1. **Weryfikacja kontekstu sesji po stronie backendu:** Endpoint `/rest/basket/{id}` powinien pobierać identyfikator użytkownika bezpośrednio z zezwolonego i zweryfikowanego tokenu sesyjnego (np. z JWT), a nie z parametru jawnie przekazywanego przez klienta w adresie URL.
2. **Kontrola uprawnień (RBAC / ABAC):** Serwer przed zwróceniem zasobu musi sprawdzić warunek: `if (basket.UserId !== currentUser.id) return 403 Forbidden`.
