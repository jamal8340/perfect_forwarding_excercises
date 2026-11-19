# Perfect forwarding w C++ — zadania na zajęcia

**Zestaw na ocenę 5:** 2 najprostsze + 1 konkretne + 2 praktyczne. Wszystkie zadania rozwiązuj samodzielnie na podstawie `materialy_perfect_forwarding.md`. Wymagany standard **C++17**. Nie używaj elementów C++20 ani C++23.

## 1. Które przeciążenie zostanie wywołane?

**Najprostsze.** W `zadanie1/main.cpp` napisz trzy przeciążenia funkcji `describe` przyjmujące odpowiednio `std::string&`, `const std::string&` i `std::string&&`. Niech wypisują odpowiednio `lvalue`, `const lvalue`, `rvalue` (każdy w osobnej linii).

W `main()` utwórz `std::string text = "raport"`, `const std::string fixed = "archiwum"` oraz `std::string&& temp = std::string{"roboczy"}`. Wywołaj `describe` kolejno dla: `text`, `fixed`, `std::string{"nowy"}`, `std::move(text)`, `temp`, `std::move(temp)`.

**Oczekiwane linie:** `lvalue`, `const lvalue`, `rvalue`, `rvalue`, `lvalue`, `rvalue`. Wyjaśnij w komentarzu, dlaczego samo `temp` jest lvalue, mimo że zadeklarowano je jako `std::string&&`. `describe` nie może zmieniać argumentów.

**Gdzie w materiałach?** Część 3, przykład 2 („typ referencji a kategoria jej nazwy”).

## 2. Zwykłe przekazanie, `std::move` i `std::forward`

**Najprostsze.** W `zadanie2/main.cpp` napisz dwa przeciążenia `receive(int&)` i `receive(int&&)` wypisujące odpowiednio `lvalue` i `rvalue`. Następnie napisz trzy szablony z parametrem `T&& value`:

- `pass_plain` — przekazuje do `receive` samo `value`;
- `pass_moved` — przekazuje `std::move(value)`;
- `pass_forwarded` — przekazuje `std::forward<T>(value)`.

Sprawdź każdą funkcję dla `int number = 7` oraz literału `7`. Przed każdą parą wywołań wypisz nazwę testowanej funkcji.

**Oczekiwane wyniki:** `pass_plain`: `lvalue`, `lvalue`; `pass_moved`: `rvalue`, `rvalue`; `pass_forwarded`: `lvalue`, `rvalue`.

W komentarzu wyjaśnij, dlaczego w uniwersalnym pośredniku `std::forward<T>` jest bezpieczniejszym wyborem od bezwarunkowego `std::move`.

**Gdzie w materiałach?** Część 5, przykład 4; części 4 i 6 (znaczenie `move`, dedukcja `T&&`).

## 3. Kopia czy przeniesienie?

**Konkretne.** W `zadanie3/main.cpp` stwórz strukturę `Tracked` z polem `int value` oraz konstruktorami:

- `Tracked(int)` — zapisuje wartość w polu;
- `Tracked(const Tracked&)` — kopiuje pole i wypisuje `copy`;
- `Tracked(Tracked&&) noexcept` — przepisuje wartość pola do nowego obiektu i wypisuje `move`.

To ćwiczenie ilustruje **wybór konstruktora**, a nie faktyczne przenoszenie dynamicznego zasobu. Nie musisz modyfikować pola źródła w konstruktorze przenoszącym.

Napisz `void consume(Tracked item)` sprawdzające przez `assert`, że `item.value == 42`, oraz szablon `relay(T&& value)`, który wywołuje `consume` z wykorzystaniem `std::forward<T>(value)`.

W `main()` przygotuj `Tracked source{42}` i `const Tracked fixed{42}`. Wykonaj po jednym wywołaniu, wypisując przed nim krótki nagłówek:

1. `relay(source)` — `copy`;
2. `relay(std::move(source))` — `move`;
3. `relay(std::move(fixed))` — `copy`.

Wyjaśnij w komentarzu, dlaczego `std::move(fixed)` nie wywołuje konstruktora `Tracked(Tracked&&)`.

**Gdzie w materiałach?** Część 7, przykład 5; część 4 („dlaczego `std::move` na stałym obiekcie może spowodować kopię?”).

## 4. Fabryka zgłoszeń

**Praktyczne.** W `zadanie4/main.cpp` zdefiniuj strukturę `Ticket` z polami `std::string title` i `std::unique_ptr<int> priority`. Jej konstruktor powinien przyjmować oba argumenty **przez wartość** i przenosić je do pól.

Napisz szablon fabryki:

```cpp
template<class T, class... Args>
std::unique_ptr<T> make_record(Args&&... args);
```

Utwórz w niej obiekt przez `std::make_unique<T>`, przekazując argumenty z użyciem `std::forward<Args>(args)...`.

W `main()` przygotuj `std::string title = "Blad logowania"` i `auto priority = std::make_unique<int>(3)`, a następnie utwórz `auto ticket = make_record<Ticket>(title, std::move(priority))`. Sprawdź przez `assert`, że `title` się nie zmienił, `priority` jest pusty, a utworzony `Ticket` przechowuje poprawny tytuł i priorytet.

**Część druga (obowiązkowa).** Napisz funkcję:

```cpp
template<class... Args>
void add_ticket(std::vector<Ticket>& tickets, Args&&... args);
```

Funkcja ma konstruować zgłoszenie bezpośrednio w wektorze przez `emplace_back(std::forward<Args>(args)...)`. Dodaj do wektora dwa zgłoszenia: pierwsze z nazwanego `std::string` i `std::unique_ptr<int>` przekazanego przez `std::move`, drugie z tymczasowego `std::string` i nowego `std::unique_ptr<int>`. Sprawdź przez `assert` rozmiar wektora, wartości zgłoszeń, zachowanie oryginalnego napisu oraz przeniesienie własności wskaźnika.

**Wyjaśnij w komentarzu:** dlaczego nazwany `unique_ptr` wymaga `std::move` i czym różni się konstrukcja przez `emplace_back` od dodania wcześniej utworzonego obiektu.

**Gdzie w materiałach?** Część 8, przykłady 6 i 6a; część 4, przykład 3 (`unique_ptr`).

## 5. Wrapper z prostym logowaniem

**Praktyczne.** W `zadanie5/main.cpp` napisz uniwersalną funkcję:

```cpp
template<class F, class... Args>
decltype(auto) logged_call(F&& function, Args&&... args);
```

Funkcja powinna wypisać `start`, a następnie wywołać przekazaną operację przez `std::invoke`, przekazując `function` oraz **wszystkie argumenty** przez `std::forward`. Zwróć wynik operacji **bezpośrednio**, zachowując typ referencyjny, jeżeli funkcja docelowa zwraca referencję. W tym zadaniu **nie trzeba wypisywać `end`**; nie trzeba także obsługiwać wyjątków ani operacji zwracających `void`.

W `main()` sprawdź:

1. `int counter = 4` i lambda `[](int& x, int amount) -> int& { x += amount; return x; }`. Wywołaj `int& result = logged_call(increment, counter, 3)`, sprawdź `counter == 7`, a po `result = 20` sprawdź `counter == 20` (zachowana referencja).
2. Wywołanie lambdy mnożącej dwie liczby `6` i `7` — wynik `42`.
3. **Samodzielne zastosowanie:** utwórz strukturę `Counter` z polem `int value = 0` i metodą `int& add(int amount)`, która zwiększa `value` i zwraca referencję do tego pola. Wywołaj ją przez `logged_call(&Counter::add, counterObject, 5)`, gdzie `counterObject` jest obiektem `Counter`. Sprawdź wynik i udowodnij, że zwrócona referencja pozwala zmienić `counterObject.value`.

Każde wywołanie `logged_call` ma wypisać dokładnie jeden wiersz `start`. W komentarzu wyjaśnij, dlaczego zastosowano `decltype(auto)` zamiast `auto`.

**Gdzie w materiałach?** Część 9, przykład „Najpierw argumenty i wynik” (`invokeSimple`) oraz opis `std::invoke` i wskaźników do metod. Przykład 7 jest opcjonalny.

---

## Organizacja

- Jedno zadanie = jeden program (`zadanieN/main.cpp`). Pliki startowe są szkieletami, nie gotowymi odpowiedziami.
- Możesz korzystać z materiałów i biblioteki standardowej; rozwiązania mają być możliwe do napisania bez AI.
- Korzystaj z `assert` w zadaniach 3–5.
- Nie używaj `requires`, `std::forward_like`, RAII do logowania ani `if constexpr`, bo nie są potrzebne do tych zadań.
- Zgodność: **C++17**, bez dodatkowych bibliotek zewnętrznych.
