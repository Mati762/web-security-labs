# Write-up: Error Handling (OWASP Juice Shop ⭐)

* **Platforma:** OWASP Juice Shop (Docker localhost)
* **Kategoria:** Security Misconfiguration / Błędy obsługi wyjątków
* **CWE:** CWE-209 (Generation of Error Message Containing Sensitive Information)
* **Status:** Rozwiązane (manualnie)

---

### 1. Opis problemu
Endpoint `/rest/basket/:id` przyjmuje identyfikator koszyka użytkownika. Przy poprawnym korzystaniu z aplikacji parametr ten jest liczbą całkowitą. Celem było sprawdzenie, jak backend zachowa się w momencie przekazania wartości nieliczbowej, której programista mógł nie przewidzieć.

---

### 2. Kroki do odtworzenia (Proof of Concept)
1. Uruchomiono aplikację OWASP Juice Shop w kontenerze Dockera.
2. W pasku adresu przeglądarki wysłano zapytanie GET z ciągiem tekstowym `undefined` zamiast numeru koszyka:
   ```text
   http://localhost:3000/rest/basket/undefined
