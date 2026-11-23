# Write-up: Mass Dispel (OWASP Juice Shop ⭐)

* **Platforma:** OWASP Juice Shop (Docker localhost)
* **Kategoria:** Unhandled Client-Side Issues / UI Shortcuts
* **CWE:** CWE-639 (Authorization Bypass Through User-Controlled Key)
* **Status:** Rozwiązane (manualnie / skrót klawiszowy)

---

### 1. Opis problemu
Aplikacja wyświetla powiadomienia (snackbary) w prawym górnym rogu po każdym osiągnięciu lub sukcesie. Przy nagromadzeniu wielu komunikatów ręczne zamykanie każdego pojedynczego okna jest uciążliwe. Interfejs zawiera ukrytą logikę frontendową (event listener w Angularze), która po wykryciu wciśniętego klawisza `Shift` przy zamykaniu pojedynczego powiadomienia wykonuje funkcję czyszczącą całą tablicę komunikatów.

---

### 2. Kroki do odtworzenia (Proof of Concept)
1. Wywołano pojawienie się na ekranie przynajmniej dwóch powiadomień snackbar.
2. Przytrzymano klawisz **`Shift`** na klawiaturze.
3. Kliknięto ikonę zamknięcia `X` na dowolnym zielonym powiadomieniu.
4. Przeglądarka przekazała flagę `event.shiftKey = true` do event listenera, co spowodowało hurtowe wyczyszczenie całej tablicy aktywnych powiadomień z interfejsu użytkownika i zaliczenie wyzwania.

---

### 3. Wpływ na bezpieczeństwo (Impact)
Wykorzystanie ukrytych skrótów interfejsu (`Shift` + klik) nie stanowi bezpośredniego zagrożenia dla bezpieczeństwa, ale ilustruje obecność nieudokumentowanej logiki po stronie klienta (Client-Side Logic), która w bardziej zaawansowanych aplikacjach może prowadzić do ominięcia kroków w procesach biznesowych.

---

### 4. Proponowana naprawa
1. **Jawny interfejs użytkownika:** Dodanie widocznego przycisku „Zamknij wszystkie” (*Dismiss All*) w interfejsie zamiast polegania na nieudokumentowanych kombinacjach klawiszy.
