# Analiza i predykcja cukrzycy na podstawie danych klinicznych

## Wstęp i cel projektu

Głównym celem projektu jest opracowanie modelu uczenia maszynowego służącego do wczesnej diagnostyki cukrzycy. Problem został zdefiniowany jako **klasyfikacja binarna**.

W procesie modelowania przyjęto priorytet maksymalizacji metryki **Recall (czułość)**. W diagnostyce medycznej kluczowe jest zminimalizowanie błędu II rodzaju (False Negatives) – model powinien z najwyższą możliwą skutecznością identyfikować osoby chore, nawet kosztem większej liczby wyników fałszywie dodatnich, które mogą zostać zweryfikowane w toku dalszych badań.

## Charakterystyka danych

Wykorzystany zbiór danych (`diabetes.csv`) zawiera 403 rekordy oraz 19 atrybutów klinicznych i demograficznych.

- **Atrybut decyzyjny (`Diagnosis`):** Zmienna kategoryczna utworzona na podstawie poziomu hemoglobiny glikowanej (`glyhb`). Przyjęto medyczny próg diagnostyczny: `glyhb >= 6.5` oznacza cukrzycę (1), wynik niższy oznacza brak choroby (0).
- **Atrybuty wejściowe:**
  - **Parametry fizjologiczne:** Cholesterol całkowity (`chol`), glukoza (`stab.glu`), HDL (`hdl`), ciśnienie tętnicze (skurczowe i rozkurczowe).
  - **Dane demograficzne i antropometryczne:** Wiek, płeć, wzrost, waga, obwód talii oraz bioder.
- **Struktura klas:** W zbiorze występuje istotna dysproporcja – stosunek osób zdrowych do chorych wynosi około 5:1.

## Pipeline przetwarzania danych

### 1. Eksploracyjna analiza danych (EDA)

- Stwierdzono silną korelację dodatnią między poziomem glukozy stabilizowanej (`stab.glu`) a diagnozą.
- Zidentyfikowano braki danych w obszarze pomiarów ciśnienia oraz parametrów fizycznych pacjentów.
- Analiza rozkładów wykazała silną skośność prawostronną dla zmiennych `chol` oraz `stab.glu`.

### 2. Przygotowanie danych (Preprocessing)

- **Imputacja braków:** Zastosowano medianę dla zmiennych numerycznych oraz modę dla kategorycznych. W przypadku ciśnienia tętniczego, braki w drugim pomiarze uzupełniono wartościami z pomiaru pierwszego.
- **Inżynieria cech (Feature Engineering):**
  - **WHR (Waist-to-Hip Ratio):** Utworzono wskaźnik stosunku obwodu talii do bioder.
  - **Uśrednianie ciśnienia:** Wyliczono średnie wartości ciśnienia skurczowego i rozkurczowego z dwóch serii pomiarowych.
  - **Transformacja logarytmiczna:** Zastosowano funkcję `log1p` dla zmiennych `chol` i `stab.glu` w celu zbliżenia ich rozkładów do normalnego.
- **Selekcja cech:** Usunięto identyfikatory (`id`), lokalizację oraz zmienne pierwotne, które zostały zastąpione nowymi wskaźnikami. Usunięto kolumnę `glyhb`, aby zapobiec wyciekowi danych (target leakage).

## Modelowanie i optymalizacja

Przetestowano trzy algorytmy klasyfikacyjne:

1.  **Regresja Logistyczna:** Zastosowano parametr `class_weight='balanced'`, aby zrównoważyć wpływ mniejszościowej klasy osób chorych.
2.  **Las Losowy (Random Forest):** Model zespołowy odporny na nieliniowe zależności.
3.  **K-Najbliższych Sąsiadów (KNN):** Model oparty na podobieństwie geometrycznym pacjentów.

Dla każdego modelu przeprowadzono optymalizację hiperparametrów za pomocą **GridSearchCV** z 5-krotną walidacją krzyżową, kierując się maksymalizacją czułości (Recall).

## Wyniki i ewaluacja

Ostateczna weryfikacja modeli na zbiorze testowym wykazała przewagę Regresji Logistycznej w kontekście postawionego celu medycznego.

| Model                    | Recall (Klasa 1) | Accuracy | Precision (Klasa 1) |
| :----------------------- | :--------------- | :------- | :------------------ |
| **Regresja Logistyczna** | **1.00**         | 0.87     | 0.57                |
| **Random Forest**        | 0.69             | 0.91     | 0.75                |
| **KNN**                  | 0.31             | 0.85     | 0.57                |

Model Regresji Logistycznej poprawnie zidentyfikował wszystkich chorych pacjentów w zbiorze testowym (Recall = 1.00), co czyni go skutecznym narzędziem do wstępnego screeningu.

## Wnioski

- Najważniejszymi czynnikami wpływającymi na ryzyko cukrzycy w badanym zbiorze są: poziom glukozy, wiek oraz wskaźnik WHR.
- Mimo niewielkiej liczby rekordów, zastosowane techniki preprocessingu i balansowania klas pozwoliły na uzyskanie stabilnego modelu o wysokiej czułości.
- Opracowane rozwiązanie może stanowić wsparcie dla personelu medycznego w procesie wstępnej selekcji pacjentów wymagających pogłębionej diagnostyki.
