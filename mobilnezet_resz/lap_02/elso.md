# React Native – Route Group, Nested Layout, \_layout.jsx, Slot, Stack és Tabs

## 1. Az Expo Router szerepe

Az Expo Router egy fájlalapú navigációs rendszer React Native alkalmazásokhoz. A `app` vagy `src/app` mappában létrehozott fájlok és könyvtárak határozzák meg az alkalmazás képernyőit és útvonalait.

Például:

```text
app/
├── _layout.jsx
├── index.jsx
├── profile.jsx
└── settings.jsx
```

Ebben a struktúrában:

- `index.jsx`: a kezdőképernyő, amely a `/` útvonalhoz tartozik.
- `profile.jsx`: a `/profile` útvonalhoz tartozó képernyő.
- `settings.jsx`: a `/settings` útvonalhoz tartozó képernyő.
- `_layout.jsx`: a képernyők közös navigációs elrendezését határozza meg.

Az Expo Router automatikusan összekapcsolja a fájlokat az útvonalakkal, így nem szükséges minden képernyőhöz külön útvonal-konfigurációt létrehozni.

## 2. Route Group – útvonalcsoport

A `Route Group` olyan könyvtár, amely az összetartozó képernyőket csoportosítja anélkül, hogy a csoport neve megjelenne az URL-útvonalban.

A csoport nevét zárójelek közé kell írni.

Például:

```text
app/
├── (public)/
│   ├── login.jsx
│   └── register.jsx
├── (private)/
│   ├── dashboard.jsx
│   └── profile.jsx
└── _layout.jsx
```

A csoportnevek nem részei az útvonalnak:

- `app/(public)/login.jsx` → `/login`
- `app/(public)/register.jsx` → `/register`
- `app/(private)/dashboard.jsx` → `/dashboard`
- `app/(private)/profile.jsx` → `/profile`

### Mire használjuk?

A Route Group segítségével logikailag különválaszthatjuk például:

- a bejelentkezéshez tartozó képernyőket;
- a regisztrációs képernyőket;
- a bejelentkezés után elérhető képernyőket;
- az adminisztrációs felületet;
- az alkalmazás különböző funkcióit.

A csoportokhoz saját `_layout.jsx` fájl is tartozhat, így a csoport képernyői közös navigációs elrendezést használhatnak.

### Példa saját layouttal

```text
app/
├── (auth)/
│   ├── _layout.jsx
│   ├── login.jsx
│   └── register.jsx
└── _layout.jsx
```

Az `auth` csoporthoz tartozó képernyők külön layoutot használhatnak, például közös fejlécet vagy egységes navigációs viselkedést.

**Fontos:** a Route Group önmagában nem biztosít hozzáférés-védelmet. A bejelentkezés és a jogosultságok ellenőrzését külön kell megvalósítani.

## 3. Nested Layout – egymásba ágyazott elrendezés

A `Nested Layout` azt jelenti, hogy egy layouton belül további layoutokat használunk.

Az alkalmazásnak lehet egy általános elrendezése, amelyen belül külön elrendezést kapnak például a felhasználói, adminisztrációs vagy bejelentkezési képernyők.

### Példa mappaszerkezetre

```text
app/
├── _layout.jsx
├── index.jsx
├── (admin)/
│   ├── _layout.jsx
│   ├── dashboard.jsx
│   └── users.jsx
└── (auth)/
    ├── _layout.jsx
    ├── login.jsx
    └── register.jsx
```

A működés logikája:

1. A legfelső `_layout.jsx` az alkalmazás gyökérszintű elrendezése.
2. Az `(admin)` csoport saját layoutot használ.
3. Az `(auth)` csoport szintén saját layoutot használ.
4. A csoportokon belüli képernyők a megfelelő szülői elrendezésbe kerülnek.

A gyökérszintű layout például egy `Stack` navigátort tartalmazhat, míg az adminisztrációs layout saját navigációt vagy közös felületi elemeket határozhat meg.

**Előnye:** a közös működést és megjelenést nem kell minden képernyőn külön megírni.

## 4. \_layout.jsx – az elrendezés konfigurációs fájlja

A `_layout.jsx` egy speciális fájlnév az Expo Routerben. Az adott könyvtárhoz tartozó képernyők közös elrendezését és navigációs szerkezetét határozza meg.

Nem önálló képernyőként működik, hanem a hozzá tartozó képernyők megjelenítését szervezi.

### Egyszerű gyökérszintű layout

```jsx
// app/_layout.jsx

import { Stack } from 'expo-router';

export default function RootLayout() {
    return (
        <Stack>
            <Stack.Screen
                name="index"
                options={{ title: 'Kezdőlap' }}
            />

            <Stack.Screen
                name="profile"
                options={{ title: 'Profil' }}
            />
        </Stack>
    );
}
```

Ebben a példában:

- a `Stack` egy veremalapú navigációs rendszert hoz létre;
- a `Stack.Screen` egy képernyőt konfigurál;
- a `name` a fájlalapú útvonalat azonosítja;
- az `options` a képernyő beállításait tartalmazza;
- a `title` a navigációs fejlécben megjelenő címet állítja be.

Ha az `index.jsx` és a `profile.jsx` fájl létezik, a navigátor ezeket a képernyőket kezeli.

### A layout és a képernyő közötti különbség

A képernyő a tényleges tartalmat jeleníti meg, a layout pedig meghatározza, hogy ez a tartalom milyen navigációs környezetben jelenjen meg.

Például:

- `profile.jsx`: megjeleníti a felhasználó profilját;
- `_layout.jsx`: meghatározza, hogy a profil milyen navigátorban és milyen fejlécbeállításokkal jelenjen meg.

## 5. Slot – a gyermekképernyők helye

A `Slot` egy olyan komponens, amely a szülői layoutba illeszti be az aktuális útvonalhoz tartozó gyermekképernyőt.

Akkor hasznos, amikor közös elrendezést szeretnénk biztosítani, de nem akarunk külön `Stack` vagy `Tabs` navigátort létrehozni.

### Példa

```jsx
// app/_layout.jsx

import { Slot } from 'expo-router';
import { View, Text, StyleSheet } from 'react-native';

export default function RootLayout() {
    return (
        <View style={styles.container}>
            <Text style={styles.header}>Saját alkalmazásom</Text>

            <Slot />
        </View>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 20,
    },
    header: {
        fontSize: 22,
        fontWeight: 'bold',
        marginBottom: 20,
    },
});
```

A `Slot` helyére kerül az aktuális útvonalhoz tartozó képernyő tartalma.

Ha például a `profile.jsx` útvonalat nyitjuk meg, akkor a profilképernyő a közös fejléc alatti területen jelenik meg.

### Mikor használjuk?

A `Slot` jó választás, ha:

- közös fejlécet vagy láblécet szeretnénk;
- közös háttérrel vagy konténerrel jelenítenénk meg a képernyőket;
- egyszerű, közös layoutot készítünk;
- a navigációs viselkedést nem akarjuk a layoutban `Stack` vagy `Tabs` segítségével meghatározni.

**Fontos:** a `Slot` nem navigációs menü, és önmagában nem biztosít verem- vagy lapfülnavigációt.

## 6. Stack – veremalapú navigáció

A `Stack` olyan navigációs elrendezés, amely a képernyőket egy veremhez hasonlóan kezeli.

Amikor egy új képernyőre lépünk, az általában az aktuális képernyő fölé kerül. A visszalépéskor visszatérhetünk az előző képernyőre.

### Példa

```jsx
// app/_layout.jsx

import { Stack } from 'expo-router';

export default function RootLayout() {
    return (
        <Stack>
            <Stack.Screen
                name="index"
                options={{ title: 'Kezdőlap' }}
            />

            <Stack.Screen
                name="details"
                options={{ title: 'Részletek' }}
            />
        </Stack>
    );
}
```

A hozzá tartozó képernyők:

```jsx
// app/index.jsx

import { View, Text, Button } from 'react-native';
import { router } from 'expo-router';

export default function HomeScreen() {
    return (
        <View>
            <Text>Kezdőlap</Text>

            <Button
                title="Részletek megnyitása"
                onPress={() => router.push('/details')}
            />
        </View>
    );
}
```

```jsx
// app/details.jsx

import { View, Text, Button } from 'react-native';
import { router } from 'expo-router';

export default function DetailsScreen() {
    return (
        <View>
            <Text>Részletek képernyő</Text>

            <Button
                title="Vissza"
                onPress={() => router.back()}
            />
        </View>
    );
}
```

A `router.push("/details")` megnyitja a részletek képernyőt, a `router.back()` pedig visszalép az előző navigációs állapotra, ha van ilyen.

### Mikor használjuk?

A Stack megfelelő például:

- listaelem részleteinek megjelenítésére;
- profil- és beállítási képernyők közötti navigációra;
- több lépésből álló űrlapokhoz;
- olyan felületekhez, ahol fontos a visszalépés.

A Stack fejlécének megjelenése és a képernyőváltás animációja beállításokkal módosítható.

## 7. Tabs – lapfüles navigáció

A `Tabs` olyan navigációs elrendezés, amelyben a felhasználó több fő képernyő között válthat, jellemzően a képernyő alján megjelenő lapfülek segítségével.

A lapfüles navigáció gyakori például közösségi alkalmazásokban, ahol külön lap tartozik a kezdőlaphoz, a kereséshez és a profilhoz.

### Példa mappaszerkezetre

```text
app/
├── _layout.jsx
└── (tabs)/
    ├── _layout.jsx
    ├── index.jsx
    ├── search.jsx
    └── profile.jsx
```

A `(tabs)` könyvtár egy útvonalcsoport. A neve nem jelenik meg az útvonalakban.

### A lapfülek konfigurálása

```jsx
// app/_layout.jsx

import { Stack } from 'expo-router';

export default function RootLayout() {
    return <Stack screenOptions={{ headerShown: false }} />;
}
```

```jsx
// app/(tabs)/_layout.jsx

import { Tabs } from 'expo-router';

export default function TabsLayout() {
    return (
        <Tabs>
            <Tabs.Screen
                name="index"
                options={{ title: 'Kezdőlap' }}
            />

            <Tabs.Screen
                name="search"
                options={{ title: 'Keresés' }}
            />

            <Tabs.Screen
                name="profile"
                options={{ title: 'Profil' }}
            />
        </Tabs>
    );
}
```

A három képernyő külön lapfülként jelenik meg.

A megfelelő fájlok egyszerű képernyőkomponenseket exportálnak:

```jsx
// app/(tabs)/index.jsx

import { View, Text } from 'react-native';

export default function HomeScreen() {
    return (
        <View>
            <Text>Kezdőlap</Text>
        </View>
    );
}
```

```jsx
// app/(tabs)/search.jsx

import { View, Text } from 'react-native';

export default function SearchScreen() {
    return (
        <View>
            <Text>Keresés</Text>
        </View>
    );
}
```

```jsx
// app/(tabs)/profile.jsx

import { View, Text } from 'react-native';

export default function ProfileScreen() {
    return (
        <View>
            <Text>Profil</Text>
        </View>
    );
}
```

### Mikor használjuk?

A Tabs jó választás, ha:

- több fő funkció között kell gyorsan váltani;
- a navigációs lehetőségeket folyamatosan láthatóvá szeretnénk tenni;
- mobilalkalmazásban alsó navigációs sávot szeretnénk kialakítani.

A lapfülekhez ikonokat, címeket és egyéb megjelenítési beállításokat is rendelhetünk.

Például az `options` objektumban használható a `tabBarIcon` tulajdonság ikon megjelenítésére.

## 8. A fogalmak összehasonlítása

| Fogalom         | Feladata                                                                        |
| --------------- | ------------------------------------------------------------------------------- |
| `Route Group`   | Képernyők logikai csoportosítása az útvonal nevének módosítása nélkül.          |
| `Nested Layout` | Egymásba ágyazott elrendezések kialakítása.                                     |
| `_layout.jsx`   | Az adott könyvtár képernyőinek közös elrendezését és navigációját konfigurálja. |
| `Slot`          | Az aktuális gyermekképernyőt illeszti a layoutba.                               |
| `Stack`         | Veremalapú navigációt biztosít a képernyők között.                              |
| `Tabs`          | Lapfüles navigációt biztosít a fő képernyők között.                             |

## 9. Gyakorlati példa – többféle navigáció együtt

Egy összetettebb alkalmazásban a fogalmak együtt is használhatók:

```text
app/
├── _layout.jsx
├── (auth)/
│   ├── _layout.jsx
│   ├── login.jsx
│   └── register.jsx
└── (main)/
    ├── _layout.jsx
    ├── (tabs)/
    │   ├── _layout.jsx
    │   ├── index.jsx
    │   ├── search.jsx
    │   └── profile.jsx
    └── details/
        └── [id].jsx
```

Ebben a struktúrában:

1. A gyökérszintű `_layout.jsx` az alkalmazás navigációjának kiindulópontja.
2. Az `(auth)` csoport a bejelentkezéshez és a regisztrációhoz tartozik.
3. A `(main)` csoport a fő alkalmazási felületet foglalja össze.
4. A `(tabs)` csoporton belüli képernyők lapfüles navigációt használnak.
5. A `details/[id].jsx` dinamikus útvonalon érhető el, például `/details/42` formában.
6. A Nested Layout lehetővé teszi, hogy a különböző szintek eltérő navigációs beállításokat kapjanak.

A konkrét navigációs viselkedés a layoutokban beállított navigátoroktól és az útvonalstruktúrától függ.

## 10. Összefoglalás

- A **Route Group** a fájlokat rendszerezi, és lehetővé teszi a különböző képernyőcsoportok elkülönítését.
- A **Nested Layout** több szintű, egymásba ágyazott elrendezést jelent.
- A **`_layout.jsx`** meghatározza az adott könyvtár képernyőinek közös navigációs vagy megjelenítési környezetét.
- A **`Slot`** a gyermekképernyő tartalmának helyét jelöli a layoutban.
- A **`Stack`** egymásra épülő képernyők közötti navigációra alkalmas.
- A **`Tabs`** a fő képernyők közötti lapfüles váltást valósítja meg.

A legfontosabb gyakorlati szabály: a `Slot`, a `Stack` és a `Tabs` nem egymás szinonimái. Mindegyik más feladatot lát el, ezért azt válasszuk, amelyik az adott felület navigációs igényének megfelel.

A témához következő lépésként egy rövid, futtatható Expo Router projektet is készíthetünk, amely mind a hat fogalmat együtt mutatja be: Teljes projektpélda JavaScriptben
