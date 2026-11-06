# Write-up: Bonus Payload / Iframe Injection (OWASP Juice Shop ⭐)

* **Platforma:** OWASP Juice Shop (Docker localhost)
* **Kategoria:** Cross-Site Scripting / HTML Injection
* **CWE:** CWE-79 (Improper Neutralization of Input During Web Page Generation)
* **Status:** Rozwiązane (manualnie)

---

### 1. Opis problemu
Podatność wykorzystuje ten sam brak walidacji i kodowania w wyszukiwarce produktów co w zadaniu *DOM XSS*. Tym razem zamiast wywołania lokalnej funkcji JavaScript, wstrzyknięto znacznik `<iframe>` ładujący zewnętrzny zasób z innego serwera.

---

### 2. Kroki do odtworzenia (Proof of Concept)
1. Otwarto pasek wyszukiwarki w sklepie.
2. Wklejono kod HTML osadzający zewnętrzny odtwarzacz z serwisu SoundCloud:
   ```html
   <iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="[https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true](https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true)"></iframe>
   ```
3. Strona wyrenderowała ramkę w miejscu nagłówka wyników wyszukiwania i załadowała zewnętrzną zawartość.

---

### 3. Wpływ na bezpieczeństwo (Impact)
Możliwość osadzenia dowolnej zewnętrznej ramki `<iframe>` pozwala atakującemu na:
* Podmianę wyglądu strony (defacement) poprzez zakrycie właściwej treści sklepu.
* Osadzenie fałszywego panelu płatności lub logowania hostowanego na serwerze atakującego pod prawdziwym adresem URL sklepu.

---

### 4. Proponowana naprawa
1. **Kodowanie danych wejściowych** we frontendzie przed wyświetleniem ich w drzewie DOM.
2. **Content Security Policy (CSP):** Wdrożenie nagłówka HTTP `Content-Security-Policy` z dyrektywą `frame-src 'self'`, która blokuje ładowanie ramek z niezaufanych domen zewnętrznych.
