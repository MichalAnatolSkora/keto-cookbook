# Dokumentacja Umiejętności Keto

Ten dokument definiuje kluczowe umiejętności kulinarne jako funkcje techniczne, aby zapewnić spójne i wysokiej jakości wykonanie w środowisku ketogenicznym.

## Umiejętności (Skills)

### 1. Smażenie Mięsa (Searing)

**Nazwa**: `skill_searing_meat`

**Opis**:
Wykonywanie obróbki cieplnej powierzchni białka w wysokiej temperaturze w celu wywołania reakcji Maillarda. Ta "funkcja" jest krytyczna w gotowaniu keto dla generowania głębi smaku (umami) bez użycia marynat czy glazur na bazie cukru. Tworzy kontrast teksturalny między skórką a wnętrzem.

**Schemat Wejściowy (Input Schema)**:
```json
{
  "surface_temp": {
    "type": "integer",
    "description": "Docelowa temperatura powierzchni patelni w stopniach Celsjusza.",
    "minimum": 200
  },
  "fat_type": {
    "type": "string",
    "enum": ["tallow", "ghee", "avocado_oil"],
    "description": "Tłuszcz o wysokim punkcie dymienia zgodny z makro keto."
  },
  "doneness": {
    "type": "string",
    "enum": ["rare", "medium-rare", "medium", "well-done"],
    "description": "Pożądany stopień wysmażenia białka."
  }
}
```

**Schemat Wyjściowy (Output Schema)**:
```json
{
  "internal_temp_c": "integer",
  "crust_quality": {
    "type": "string",
    "enum": ["golden_brown", "dark_brown", "charred", "pale"]
  },
  "success": "boolean"
}
```

**Logika Implementacji**:
1.  **Rozgrzej**: Zainicjuj patelnię do `surface_temp`. Zweryfikuj temperaturę termometrem IR lub testem kropli wody (efekt Leidenfrosta).
2.  **Osusz**: Usuń wilgoć z powierzchni białka ręcznikiem papierowym. Wilgoć hamuje reakcję Maillarda.
3.  **Przypraw**: Posól obficie powierzchnię tuż przed smażeniem.
4.  **Natłuść**: Dodaj `fat_type` na patelnię. Czekaj, aż tłuszcz zacznie lśnić (zmiana lepkości) lub lekko dymić.
5.  **Wykonaj**: Umieść białko na patelni.
    *   *Ograniczenie*: Nie przepełniaj (błąd generowania pary).
    *   *Akcja*: Dociśnij, aby zweryfikować równomierny kontakt.
    *   *Czekaj*: Nie ruszaj białka, dopóki samo nie odejdzie od powierzchni patelni.
6.  **Obróć**: Powtórz dla drugiej strony, aż temperatura wewnętrzna spełni kryteria `doneness`.
7.  **Odpocznij**: Przenieś na neutralną powierzchnię na 5-10 minut, aby rozprowadzić soki wewnętrzne.

**Obsługa Błędów (Error Handling)**:
*   `error: burning_fats`: Jeśli dym jest czarny/gryzący, temperatura > punkt dymienia. **Akcja**: Wylej tłuszcz, wytrzyj patelnię, zacznij od nowa z niższą temperaturą lub tłuszczem o wyższym punkcie dymienia.
*   `error: gray_meat`: Brak skórki. **Przyczyna**: Zbyt zimna patelnia lub zbyt mokre mięso. **Akcja**: Zdejmij mięso, osusz ponownie, zwiększ ogień, spróbuj ponownie.

---

### 2. Deglazowanie Patelni (Deglazing)

**Nazwa**: `skill_deglazing_pan`

**Opis**:
Odzyskiwanie skarmelizowanych białek (fond) przywartych do patelni po smażeniu. Ta funkcja rozpuszcza związki smakowe w medium płynnym, tworząc bazę dla sosów przyjaznych keto. Niezbędne dla efektywności zasobów i maksymalizacji smaku.

**Schemat Wejściowy (Input Schema)**:
```json
{
  "pan_state": {
    "type": "string",
    "description": "Stan patelni po smażeniu.",
    "required_pattern": "contains_fond"
  },
  "liquid_agent": {
    "type": "string",
    "enum": ["bone_broth", "dry_wine", "vinegar", "water"],
    "description": "Rozpuszczalnik dla fondu. Musi być niskowęglowodanowy."
  },
  "volume_ml": {
    "type": "integer",
    "description": "Ilość płynu do dodania.",
    "minimum": 50
  }
}
```

**Schemat Wyjściowy (Output Schema)**:
```json
{
  "sauce_consistency": {
    "type": "string",
    "enum": ["nappe", "watery", "syrup"]
  },
  "flavor_profile": "string",
  "success": "boolean"
}
```

**Logika Implementacji**:
1.  **Oceń**: Zweryfikuj `pan_state`. Sprawdź kolor fondu.
    *   *Sprawdź*: Złoty/Ciemnobrązowy = OK. Czarny = `error: burned_fond`.
2.  **Kontrola Ciepła**: Zmniejsz ogień na średnio-wysoki, jeśli patelnia nadmiernie dymi.
3.  **Wykonaj**: Wlej `liquid_agent` (`volume_ml`) na gorącą patelnię.
    *   *Oczekiwanie*: Natychmiastowe wrzenie i uwolnienie pary.
4.  **Zeskrob**: Użyj drewnianej lub silikonowej szpatułki, aby stanowczo zeskrobać dno patelni.
5.  **Rozpuść**: Kontynuuj mieszanie, aż wszystkie stałe cząstki fondu zostaną zawieszone w fazie płynnej.
6.  **Zredukuj**: Gotuj na wolnym ogniu, aż objętość płynu zmniejszy się o 50%, aby skoncentrować smak.

**Obsługa Błędów (Error Handling)**:
*   `error: burned_fond`: Fond jest czarny i pachnie węglem. **Akcja**: Przerwij. Nie deglazuj. Gorzki smak jest nie do naprawienia. Wyczyść patelnię i zacznij sos oddzielnie.
*   `error: weak_flavor`: Za dużo płynu lub niewystarczająca redukcja. **Akcja**: Kontynuuj proces redukcji.

---

### 3. Emulgowanie Sosu (Emulsifying)

**Nazwa**: `skill_emulsifying_sauce`

**Opis**:
Wymuszanie stabilnej zawiesiny koloidalnej dwóch niemieszających się cieczy (tłuszczu i wody/kwasu). W kuchni keto jest to podstawowa metoda tworzenia bogatych, kremowych sosów (Holenderski, Beurre Blanc, Majonez) bez zagęstników skrobiowych.

**Schemat Wejściowy (Input Schema)**:
```json
{
  "dispersed_phase": {
    "type": "string",
    "description": "Składnik tłuszczowy (np. roztopione masło, olej)."
  },
  "continuous_phase": {
    "type": "string",
    "description": "Składnik wodny/kwaśny (np. żółtko jaja, redukcja octu)."
  },
  "method": {
    "type": "string",
    "enum": ["whisk", "blender", "immersion_blender"],
    "default": "whisk"
  },
  "temperature_c": {
    "type": "integer",
    "description": "Krytyczna temperatura dla stabilności.",
    "maximum": 60
  }
}
```

**Schemat Wyjściowy (Output Schema)**:
```json
{
  "emulsion_stability": {
    "type": "string",
    "enum": ["stable", "broken", "separated"]
  },
  "viscosity": "string",
  "success": "boolean"
}
```

**Logika Implementacji**:
1.  **Konfiguracja Bazy**: Umieść `continuous_phase` w naczyniu do mieszania.
2.  **Inicjuj Mieszanie**: Rozpocznij `method` (trzepanie/blendowanie) na bazie.
3.  **Włączanie**: POWOLI dodawaj `dispersed_phase`.
    *   *Tempo*: Początkowo kropla po kropli, potem cienkim strumieniem.
    *   *Logika*: `agitation_rate` musi przekraczać `addition_rate`, aby rozbić tłuszcz na mikrony.
4.  **Monitoruj**: Obserwuj teksturę. Powinna zgęstnieć i stać się nieprzezroczysta.
5.  **Finalizuj**: Przypraw, gdy emulsja jest stabilna.

**Obsługa Błędów (Error Handling)**:
*   `error: broken_sauce`: Audyt wykazuje plamy oleju oddzielone od płynu. **Akcja (Naprawa)**:
    1.  Weź czystą miskę.
    2.  Dodaj 1 łyżeczkę ciepłej wody (lub świeże żółtko).
    3.  Powoli wtrzep *zepsuty* sos do nowej bazy kropla po kropli.
*   `error: scrambled_eggs`: Temperatura przekroczyła 65°C. **Akcja**: Nieodwracalny błąd krytyczny. Wyrzuć i zacznij od nowa.

---

### 4. Dokumentowanie Przepisu (Documenting Recipe)

**Nazwa**: `skill_documenting_recipe`

**Opis**:
Standaryzowana procedura dodawania nowego przepisu do repozytorium. Ta funkcja zapewnia, że każdy algorytm kulinarny jest dostępny zarówno dla osób mówiących po angielsku, jak i po polsku, wymuszając dwujęzyczne parametry wejściowe.

**Schemat Wejściowy (Input Schema)**:
```json
{
  "title": {
    "en": "string (Tytuł Angielski)",
    "pl": "string (Tytuł Polski)"
  },
  "description": {
    "en": "string (Opis Angielski)",
    "pl": "string (Opis Polski)"
  },
  "ingredients": [
    {
      "item": "string",
      "quantity": "number",
      "unit": "string",
      "translation_pl": "string (Polska nazwa składnika i jednostki)"
    }
  ],
  "steps": [
    {
      "step_number": "integer",
      "instruction_en": "string",
      "instruction_pl": "string"
    }
  ],
  "macros_per_serving": {
    "net_carbs_g": "integer",
    "fat_g": "integer",
    "protein_g": "integer",
    "calories_kcal": "integer"
  }
}
```

**Schemat Wyjściowy (Output Schema)**:
```json
{
  "file_path": "string",
  "validation_status": {
    "type": "string",
    "enum": ["valid", "invalid_macros", "missing_translation"]
  },
  "commit_hash": "string"
}
```

**Logika Implementacji**:
1.  **Inicjalizuj**: plik `recipes/{category}/{recipe_name}_EN.md` ORAZ `recipes/{category}/{recipe_name}_PL.md`.
2.  **Nawigacja**: Dodaj link `[← Wróć do spisu treści](../../README_PL.md)` (lub EN) na górze pliku.
3.  **Waliduj**: Upewnij się, że wszystkie pola tekstowe mają wartości zarówno `_en`, jak i `_pl`.
4.  **Formatuj**: Wyrenderuj przepis do DWÓCH oddzielnych plików (jeden Angielski, jeden Polski).
4.  **Weryfikuj Makro**: dokładne obliczenie Węglowodanów Netto (< 10g/porcję).

**Obsługa Błędów (Error Handling)**:
*   `error: missing_translation`: Jeśli pole jest podane tylko w jednym języku. **Akcja**: Odrzuć commit. Zażądaj tłumaczenia.
*   `error: high_carb_alert`: Jeśli `net_carbs_g` > 20. **Akcja**: Oznacz jako "Low Carb" zamiast "Keto" lub odrzuć.
