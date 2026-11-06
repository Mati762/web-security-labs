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
   ```
3. Aplikacja przekierowała pod adres:
   ```text
   http://localhost:3000/#/search?q=%3Ciframe%20src%3D%22javascript:alert(%60xss%60)%22%3E
   ```
4. Przeglądarka zamiast potraktować wpis jako zwykły tekst, zinterpretowała znacznik i wykonała kod JavaScript, wyświetlając okienko `alert`.

---

### 3. Wpływ na bezpieczeństwo (Impact)
Podatność pozwala na wykonanie dowolnego kodu JavaScript w kontekście sesji ofiary po kliknięciu przez nią spreparowanego linku. Atakujący może w ten sposób:
* Wykraść token sesyjny (np. z `localStorage`) i przejąć konto użytkownika lub administratora.
* Wyświetlić fałszywy formularz logowania wewnątrz zaufanej domeny sklepu (phishing).
* Wykonywać nieautoryzowane akcje w tle w imieniu zalogowanego użytkownika.

---

### 4. Proponowana naprawa
1. **Sanityzacja i kodowanie danych wyjściowych:** Przed wstawieniem ciągu znaków z parametrów URL do drzewa DOM należy zamienić znaki specjalne HTML (np. `<`, `>`, `"`, `'`) na bezpieczne encje HTML.
2. **Unikanie niebezpiecznych metod DOM:** Zamiast bezpośredniego wstrzykiwania surowego HTML (np. przez `innerHTML`), należy używać bezpiecznych metod przypisywania tekstu (np. `textContent`).
