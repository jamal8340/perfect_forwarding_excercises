# Perfect forwarding w C++ — zadania na zajęcia

Zestaw na **ocenę 5**: **2 najprostsze + 1 konkretne + 2 praktyczne**, zgodnie z kryteriami.

- [Treść pięciu zadań](zadania.md)
- `zadanie1/` — przeciążenia lvalue, const lvalue i rvalue
- `zadanie2/` — zwykłe przekazanie, move i forward
- `zadanie3/` — kopiowanie i przenoszenie
- `zadanie4/` — fabryka `Ticket` i `emplace_back` w `std::vector`
- `zadanie5/` — wrapper z `std::invoke`, zachowaniem referencji i wywołaniem metody

Zadania bazują wyłącznie na `materialy_perfect_forwarding.md` (sekcje 3–9). W treści podano, do której części materiału warto zajrzeć. 

## Wymagania

- Kompilator obsługujący **C++17**, np. GCC 7+, Clang 5+ lub Visual Studio 2017 (z odpowiednim wsparciem C++17).
- CMake 3.10+ lub kompilacja ręczna.
- Bez dodatkowych bibliotek.

## Uruchamianie (przykład: zadanie 1)

```sh
cmake -S zadanie1 -B zadanie1/build
cmake --build zadanie1/build
```

GCC / Clang bez CMake:

```sh
g++ -std=c++17 -Wall -Wextra -pedantic zadanie1/main.cpp -o program
./program
```

Na Windows lokalizacja pliku wykonywalnego zależy od generatora CMake, np. `zadanie1\build\Debug\program.exe`.


POWODZENIA!
