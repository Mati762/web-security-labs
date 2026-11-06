# Write-up: DOM XSS (OWASP Juice Shop ⭐)

* **Platforma:** OWASP Juice Shop (Docker localhost)
* **Kategoria:** Cross-Site Scripting (DOM-based XSS)
* **CWE:** CWE-79 (Improper Neutralization of Input During Web Page Generation)
* **Status:** Rozwiązane (manualnie)

---

### 1. Opis problemu
Pole wyszukiwarki produktów w górnym pasku aplikacji pobiera tekst wpisany przez użytkownika i wyświetla go z powrotem na stronie w nagłówku wyników wyszukiwania. Kod frontendu wstawia tę wartość bezpośrednio do drzewa DOM bez odpowiedniego oczyszczenia znaków specjalnych HTML.

---

### 2. Kroki do odtworzenia (Proof of Concept)
1. Kliknięto ikonę lupki w pasku nawigacji sklepu.
2. W pole wyszukiwania wpisano kod HTML z tagiem `<iframe>` wykorzystującym pseudo-protokół `javascript:`:
   ```html
   <iframe src="javascript:alert(`xss`)">
