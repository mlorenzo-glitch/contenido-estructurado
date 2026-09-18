# Gatos como Horóscopo — Documento de Información Estructurada

> Este documento combina la **arquitectura de navegación** (flujo de 10 secciones)
> con el **catálogo completo de datos** (razas, personalidad, patrones de pelaje).
> Está organizado para poder convertirse directamente en JSON / HTML.
> Los campos marcados como `[PENDIENTE]` requieren una decisión antes de publicarse.

---

## 0. Flujo general del sistema

```
GATOS COMO HORÓSCOPO
        │
        ▼
FECHA DE NACIMIENTO + SEXO
        │
        ▼
CICLO DE RAZAS (por año)  ──────────►  TU RAZA DE NACIMIENTO
        │
        ▼
TU SIGNO (por fecha)  ──────────────►  GATO SEGÚN TU SIGNO
        │
        ▼
TEST DE PERSONALIDAD
        │
        ▼
CLASIFICACIÓN POPULAR DE GATO (color/patrón)  ──►  ¿QUÉ GATO ERES?
        │
        ▼
MEDIA NARANJA GATUNA (cruce: raza nacimiento + signo + clasificación popular)
        │
        ▼
FICHA DE RAZA COMPATIBLE + FUENTES (TICA / Hill's / Royal Canin / Purina)
```

**Regla de diseño (heredada del análisis previo):** el sexo se conserva como
dato de entrada por experiencia de usuario, pero **no** altera ningún resultado.
No existe base confiable para vincular sexo de la persona con raza de gato.

---

## 1. SECCIÓN 01 — Carta astral (datos de entrada)

**Inputs del usuario:**
| Campo | Tipo | Notas |
|---|---|---|
| Fecha de nacimiento | Día / Mes / Año | Determina Signo (por día/mes) y Raza de nacimiento (por año) |
| Sexo | Femenino / Masculino / Otro / Prefiero no decirlo | Solo personalización, no altera resultados |

**Output de esta sección:**
- Nombre de la raza (por ciclo anual)
- Tipo de pelaje
- Características principales
- Personalidad
- Signo zodiacal correspondiente
- Breve explicación de por qué esa raza forma parte del ciclo

---

## 2. Ciclo de razas por año de nacimiento

Sistema inspirado en el zodiaco chino: cada año del calendario corresponde a
una raza dentro de un ciclo que se repite. **Es un sistema creativo del
proyecto, no una tradición real.**

**Decisión resuelta (reemplaza la propuesta anterior de un ciclo único de 12):**
el ciclo **no** es uno solo compartido por todos los signos. Cada signo tiene
su **propio** ciclo, formado por las razas que ya se le asignaron en la
Sección 3 (6 o 7 razas según el signo). El "año de nacimiento" indica en qué
posición de *ese* ciclo cae la persona.

Flujo correcto:
1. La fecha de nacimiento da el **signo** (Sección 3).
2. El signo determina **qué lista de razas** se usa como ciclo.
3. El año de nacimiento (ajustado al año chino — ver Sección 3.1) determina
   la **posición dentro del ciclo de ese signo**, es decir, la raza final.

El desarrollo completo de este sistema — incluyendo cómo vincular el año de
nacimiento a un año calendario real usando el Año Nuevo chino, y el animal
chino correspondiente como dato complementario — está en la **Sección 3.1**.

---

## 3. Signo zodiacal → Razas asociadas (segunda capa de personalidad)

No sustituye la raza de nacimiento; es un sistema paralelo. Cada signo
tiene varias razas afines (no solo una), lo que permite variedad de
combinaciones.

| Signo | Razas asociadas |
|---|---|
| ♈ Aries | Bengalí, Bengalí de Pelo Largo, Savannah, Toyger, Egyptian Mau, Chausie, Serengeti |
| ♉ Tauro | Persa, Himalayo, Exótico de Pelo Corto, Ragdoll, Chartreux, American Wirehair |
| ♊ Géminis | Siamés, Oriental de Pelo Corto, Oriental de Pelo Largo, Balinés, Tailandés, Tonkinés, Burmés |
| ♋ Cáncer | Munchkin, Munchkin Pelo Largo, Minuet, Minuet Pelo Largo, Minuet Talls, Snowshoe, Nebelung |
| ♌ Leo | Maine Coon, Maine Coon Polydactyl, Noruego de Bosque, Siberiano, Highlander, British Longhair, Turco Angora |
| ♍ Virgo | Abisinio, Ocicat, Singapura, Korat, Habana, Lykoi |
| ♎ Libra | Británico de Pelo Corto, Escocés Recto, Escocés Plegado, Birmilla, Burmilla, Burmilla Pelo Largo |
| ♏ Escorpio | Bombay, Cymric, LaPerm, LaPerm Pelo Corto, Tennessee Rex, Pixiebob, Pixiebob Pelo Largo |
| ♐ Sagitario | Devon Rex, Cornish Rex, American Bobtail, American Bobtail Shorthair, Kurilian Bobtail, Kurilian Bobtail Pelo Largo |
| ♑ Capricornio | Ruso Azul, American Shorthair, Manx, Manx Tailed, Peterbald, Donskoy |
| ♒ Acuario | Turco Van, American Curl, American Curl Pelo Largo, Sphynx, Selkirk Rex, Selkirk Rex Pelo Largo |
| ♓ Piscis | Khaomanee, Cherubim, Japanese Bobtail, Japanese Bobtail Pelo Largo, Australian Mist, Toybob |

---

---

## 3.1 Ciclo de razas vinculado a años reales (Año Nuevo Chino)

Esta sección conecta los ciclos de la **Sección 3** (razas por signo) con años
calendario reales, usando el sistema del Año Nuevo chino como referencia para
obtener el animal correspondiente a cada año. **El animal chino no determina la
raza** — es una capa informativa adicional que enriquece cada resultado (ej. "eres
un Ragdoll, del ciclo de Tauro, año del Buey").

### 3.1.1 Por qué el Año Nuevo chino (y no el 1 de enero)

El calendario chino es lunar, así que el Año Nuevo no cae el 1 de enero: se mueve
cada año entre el **21 de enero y el 20 de febrero**. Esto importa para el sistema:
si alguien nace **antes** de la fecha del Año Nuevo chino de su año de nacimiento,
su "año chino" real es el anterior, no el que marca su acta de nacimiento.

> Ejemplo: una persona nacida el 15 de enero de 2024 nació *antes* del Año Nuevo
> chino de 2024 (10 de febrero). Su año chino sigue siendo 2023 (Conejo), no 2024 (Dragón).

**Regla de cálculo:**
```
si (mes_nacimiento, día_nacimiento) < (mes_CNY, día_CNY) del año de nacimiento:
    año_chino = año_nacimiento - 1
sino:
    año_chino = año_nacimiento
```

### 3.1.2 Fechas del Año Nuevo chino (referencia 2020–2031)

| Año | Animal | Fecha de inicio (Año Nuevo chino) |
|---|---|---|
| 2020 | Rata | 25 de enero |
| 2021 | Buey | 12 de febrero |
| 2022 | Tigre | 1 de febrero |
| 2023 | Conejo | 22 de enero |
| 2024 | Dragón | 10 de febrero |
| 2025 | Serpiente | 29 de enero |
| 2026 | Caballo | 17 de febrero |
| 2027 | Cabra | 6 de febrero |
| 2028 | Mono | 26 de enero |
| 2029 | Gallo | 13 de febrero |
| 2030 | Perro | 3 de febrero |
| 2031 | Cerdo | 23 de enero |

**Fórmula general (para cualquier año, una vez ajustado por la regla 11.1):**
```
animal = LISTA_ANIMALES[ (año_chino - 2020) mod 12 ]
LISTA_ANIMALES = [Rata, Buey, Tigre, Conejo, Dragón, Serpiente,
                  Caballo, Cabra, Mono, Gallo, Perro, Cerdo]
```
(2020 = Rata se usa como año ancla; el resultado del módulo da la posición en la lista.)

### 3.1.3 Ciclos por signo con año real y animal chino

Cada ciclo de signo (Sección 3) se ancla también en **2020** como Año 1, para que
sea consistente con la fórmula anterior. Como cada signo tiene un ciclo de 6 o 7
razas (no 12), la raza y el animal chino **se desincronizan** en cada vuelta del
ciclo — igual que en el sistema chino real, donde el animal (ciclo de 12) y el
elemento (ciclo de 5) tampoco coinciden siempre, dando 60 combinaciones distintas
antes de repetirse. Aquí se muestra la primera vuelta completa de cada signo (y el
inicio de la segunda, para ver cómo cambia la combinación raza + animal).

**Aries** (ciclo de 7 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Bengalí | 2020 | Rata |
| 2 | Bengalí de Pelo Largo | 2021 | Buey |
| 3 | Savannah | 2022 | Tigre |
| 4 | Toyger | 2023 | Conejo |
| 5 | Egyptian Mau | 2024 | Dragón |
| 6 | Chausie | 2025 | Serpiente |
| 7 | Serengeti | 2026 | Caballo |
| 1 (2ª vuelta) | Bengalí | 2027 | Cabra |
| 2 (2ª vuelta) | Bengalí de Pelo Largo | 2028 | Mono |

**Tauro** (ciclo de 6 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Persa | 2020 | Rata |
| 2 | Himalayo | 2021 | Buey |
| 3 | Exótico de Pelo Corto | 2022 | Tigre |
| 4 | Ragdoll | 2023 | Conejo |
| 5 | Chartreux | 2024 | Dragón |
| 6 | American Wirehair | 2025 | Serpiente |
| 1 (2ª vuelta) | Persa | 2026 | Caballo |
| 2 (2ª vuelta) | Himalayo | 2027 | Cabra |

**Géminis** (ciclo de 7 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Siamés | 2020 | Rata |
| 2 | Oriental de Pelo Corto | 2021 | Buey |
| 3 | Oriental de Pelo Largo | 2022 | Tigre |
| 4 | Balinés | 2023 | Conejo |
| 5 | Tailandés | 2024 | Dragón |
| 6 | Tonkinés | 2025 | Serpiente |
| 7 | Burmés | 2026 | Caballo |
| 1 (2ª vuelta) | Siamés | 2027 | Cabra |
| 2 (2ª vuelta) | Oriental de Pelo Corto | 2028 | Mono |

**Cáncer** (ciclo de 7 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Munchkin | 2020 | Rata |
| 2 | Munchkin Pelo Largo | 2021 | Buey |
| 3 | Minuet | 2022 | Tigre |
| 4 | Minuet Pelo Largo | 2023 | Conejo |
| 5 | Minuet Talls | 2024 | Dragón |
| 6 | Snowshoe | 2025 | Serpiente |
| 7 | Nebelung | 2026 | Caballo |
| 1 (2ª vuelta) | Munchkin | 2027 | Cabra |
| 2 (2ª vuelta) | Munchkin Pelo Largo | 2028 | Mono |

**Leo** (ciclo de 7 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Maine Coon | 2020 | Rata |
| 2 | Maine Coon Polydactyl | 2021 | Buey |
| 3 | Noruego de Bosque | 2022 | Tigre |
| 4 | Siberiano | 2023 | Conejo |
| 5 | Highlander | 2024 | Dragón |
| 6 | British Longhair | 2025 | Serpiente |
| 7 | Turco Angora | 2026 | Caballo |
| 1 (2ª vuelta) | Maine Coon | 2027 | Cabra |
| 2 (2ª vuelta) | Maine Coon Polydactyl | 2028 | Mono |

**Virgo** (ciclo de 6 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Abisinio | 2020 | Rata |
| 2 | Ocicat | 2021 | Buey |
| 3 | Singapura | 2022 | Tigre |
| 4 | Korat | 2023 | Conejo |
| 5 | Habana | 2024 | Dragón |
| 6 | Lykoi | 2025 | Serpiente |
| 1 (2ª vuelta) | Abisinio | 2026 | Caballo |
| 2 (2ª vuelta) | Ocicat | 2027 | Cabra |

**Libra** (ciclo de 6 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Británico de Pelo Corto | 2020 | Rata |
| 2 | Escocés Recto | 2021 | Buey |
| 3 | Escocés Plegado | 2022 | Tigre |
| 4 | Birmilla | 2023 | Conejo |
| 5 | Burmilla | 2024 | Dragón |
| 6 | Burmilla Pelo Largo | 2025 | Serpiente |
| 1 (2ª vuelta) | Británico de Pelo Corto | 2026 | Caballo |
| 2 (2ª vuelta) | Escocés Recto | 2027 | Cabra |

**Escorpio** (ciclo de 7 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Bombay | 2020 | Rata |
| 2 | Cymric | 2021 | Buey |
| 3 | LaPerm | 2022 | Tigre |
| 4 | LaPerm Pelo Corto | 2023 | Conejo |
| 5 | Tennessee Rex | 2024 | Dragón |
| 6 | Pixiebob | 2025 | Serpiente |
| 7 | Pixiebob Pelo Largo | 2026 | Caballo |
| 1 (2ª vuelta) | Bombay | 2027 | Cabra |
| 2 (2ª vuelta) | Cymric | 2028 | Mono |

**Sagitario** (ciclo de 6 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Devon Rex | 2020 | Rata |
| 2 | Cornish Rex | 2021 | Buey |
| 3 | American Bobtail | 2022 | Tigre |
| 4 | American Bobtail Shorthair | 2023 | Conejo |
| 5 | Kurilian Bobtail | 2024 | Dragón |
| 6 | Kurilian Bobtail Pelo Largo | 2025 | Serpiente |
| 1 (2ª vuelta) | Devon Rex | 2026 | Caballo |
| 2 (2ª vuelta) | Cornish Rex | 2027 | Cabra |

**Capricornio** (ciclo de 6 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Ruso Azul | 2020 | Rata |
| 2 | American Shorthair | 2021 | Buey |
| 3 | Manx | 2022 | Tigre |
| 4 | Manx Tailed | 2023 | Conejo |
| 5 | Peterbald | 2024 | Dragón |
| 6 | Donskoy | 2025 | Serpiente |
| 1 (2ª vuelta) | Ruso Azul | 2026 | Caballo |
| 2 (2ª vuelta) | American Shorthair | 2027 | Cabra |

**Acuario** (ciclo de 6 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Turco Van | 2020 | Rata |
| 2 | American Curl | 2021 | Buey |
| 3 | American Curl Pelo Largo | 2022 | Tigre |
| 4 | Sphynx | 2023 | Conejo |
| 5 | Selkirk Rex | 2024 | Dragón |
| 6 | Selkirk Rex Pelo Largo | 2025 | Serpiente |
| 1 (2ª vuelta) | Turco Van | 2026 | Caballo |
| 2 (2ª vuelta) | American Curl | 2027 | Cabra |

**Piscis** (ciclo de 6 razas)

| Año del ciclo | Raza | Año real | Animal chino |
|---|---|---|---|
| 1 | Khaomanee | 2020 | Rata |
| 2 | Cherubim | 2021 | Buey |
| 3 | Japanese Bobtail | 2022 | Tigre |
| 4 | Japanese Bobtail Pelo Largo | 2023 | Conejo |
| 5 | Australian Mist | 2024 | Dragón |
| 6 | Toybob | 2025 | Serpiente |
| 1 (2ª vuelta) | Khaomanee | 2026 | Caballo |
| 2 (2ª vuelta) | Cherubim | 2027 | Cabra |

### 3.1.4 Algoritmo resumido para la web

```
entrada: fecha_nacimiento (día, mes, año), signo (ya calculado por fecha)

1. año_chino = ajustar(año_nacimiento) según regla 11.1 (comparar con fecha CNY de ese año)
2. lista_razas = RAZAS_POR_SIGNO[signo]      # de la Sección 3
3. n = len(lista_razas)
4. posición_raza = (año_chino - 2020) mod n
5. raza_resultado = lista_razas[posición_raza]
6. posición_animal = (año_chino - 2020) mod 12
7. animal_resultado = LISTA_ANIMALES[posición_animal]

salida: raza_resultado (raza de nacimiento) + animal_resultado (dato complementario)
```

**Nota:** las fechas exactas del Año Nuevo chino para años fuera de la tabla 11.2
(anteriores a 2020 o posteriores a 2031) deben consultarse en una fuente actualizada
antes de programar el calendario completo, ya que no siguen una fórmula fija de día
del mes (dependen del calendario lunar).

---

## 4. Catálogo completo de razas (base de datos maestra)

Cada raza incluye: **id** (slug para uso en código), **nombre**,
**categoría**, y **personalidad** (texto base ya redactado, para usar tal
cual o resumir en la ficha final).

### 4.1 Razas "clásicas" de pelo corto

| id | Nombre | Personalidad |
|---|---|---|
| abisinio | Abisinio | Activo, curioso, inteligente, necesita estimulación mental y física constante, ágil trepador |
| american-shorthair | American Shorthair | Equilibrado, independiente, buen cazador, se adapta bien a cambios, poco exigente |
| american-wirehair | American Wirehair | Tranquilo, cariñoso, adaptable, de temperamento fácil, poco demandante |
| britanico-pelo-corto | Británico de Pelo Corto | Tranquilo, afable, independiente sin ser distante, temperamento muy estable |
| chartreux | Chartreux | Callado, independiente, paciente, lealtad silenciosa, comportamiento casi disciplinado |
| egyptian-mau | Egyptian Mau (Mau Egipcio) | Veloz, ágil, leal a su familia, desconfiado con extraños, gran instinto cazador |
| ruso-azul | Ruso Azul | Tímido con desconocidos, extremadamente apegado a su rutina, reservado pero fiel |
| korat | Korat | Sensible, observador, muy apegado a su humano, no tolera bien los cambios bruscos |
| habana | Habana (Havana Brown) | Curioso, sociable, usa mucho sus patas para "tocar" e investigar, afectuoso |
| singapura | Singapura | Juguetón, extrovertido, muy activo pese a su tamaño pequeño, sociable con la familia |

### 4.2 Razas orientales / vocales

| id | Nombre | Personalidad |
|---|---|---|
| siames | Siamés | Platicador, le gusta la compañía, muy inteligente, exige atención, se apega fuertemente a su humano |
| oriental-pelo-corto | Oriental de Pelo Corto | Extremadamente activo, curioso, vocal, se estresa fácil si le falta estimulación |
| oriental-pelo-largo | Oriental de Pelo Largo | Igual de sociable que su versión de pelo corto, cariñoso, necesita interacción constante |
| balines | Balinés | Elegante, vocal (aunque más suave que el Siamés), sociable, muy unido a su familia |
| tailandes | Tailandés (Thai) | Cariñoso, comunicativo, inteligente, forma vínculos muy fuertes con una persona en particular |
| tonkines | Tonkinés | Energético, sociable, mezcla el carácter vocal del Siamés con la calidez del Burmés |
| burmes | Burmés | Cariñoso, sigue a su humano por toda la casa, juguetón, no le gusta estar solo |
| birmilla | Birmilla | Dulce, curioso, más tranquilo que otras orientales, sociable pero sin ser demandante |

### 4.3 Razas grandes / "gentle giants"

| id | Nombre | Personalidad |
|---|---|---|
| maine-coon | Maine Coon | Sociable, juguetón, gran tamaño y presencia, le encanta la atención pero también es independiente |
| maine-coon-polydactyl | Maine Coon Polydactyl | Mismo temperamento gentil del Maine Coon, suele ser aún más hábil con las "patas extra" |
| noruego-bosque | Noruego de Bosque | Resistente, aventurero, buen trepador, independiente pero afectuoso con su círculo cercano |
| siberiano | Siberiano | Leal, juguetón, atlético, buen saltador, se lleva bien con niños y otras mascotas |
| ragdoll | Ragdoll | Dócil, cariñoso, se relaja completamente al cargarlo, muy dependiente emocionalmente de su humano |
| chausie | Chausie | Enérgico, atlético, necesita mucho espacio y ejercicio, vínculo intenso con su familia |
| highlander | Highlander | Juguetón, curioso, sociable, conserva un aire "salvaje" pero es de trato fácil |

### 4.4 Razas exóticas con patrón salvaje

| id | Nombre | Personalidad |
|---|---|---|
| bengali | Bengalí | Hiperactivo, atlético, territorial, le encanta trepar y cazar, necesita estimulación constante |
| bengali-pelo-largo | Bengalí de Pelo Largo | Mismo temperamento activo y curioso, ligeramente más tranquilo en apariencia |
| savannah | Savannah | Muy activo, curioso, atlético, necesita mucho espacio, forma vínculos intensos y casi "de perro" |
| ocicat | Ocicat | Sociable, inteligente, entrenable, le gusta interactuar y jugar en equipo con su familia |
| toyger | Toyger | Juguetón, amigable, curioso, disfruta la atención pero no es exigente |
| serengeti | Serengeti | Alerta, vocal, atlético, muy activo y necesita estímulos constantes |

*(Egyptian Mau ya está catalogado en 4.1; aquí solo se referencia por su patrón salvaje.)*

### 4.5 Razas de pelo largo / semi-largo

| id | Nombre | Personalidad |
|---|---|---|
| persa | Persa | Tranquilo, afectuoso a su manera, sedentario, necesita cuidados constantes y ambientes calmados |
| himalayo | Himalayo | Tranquilo, algo exigente en cuidados, cariñoso pero selectivo, sensible a cambios en su entorno |
| exotico-pelo-corto | Exótico de Pelo Corto | Cariñoso, tranquilo, algo dependiente, disfruta la compañía constante de su humano |
| british-longhair | British Longhair | Sereno, afectuoso, independiente, de temperamento muy estable como su versión de pelo corto |
| turco-angora | Turco Angora | Inteligente, activo, curioso, muy vocal y apegado a su humano favorito |
| turco-van | Turco Van | Atípico, activo, le encanta el agua (raro en gatos), independiente y de energía impredecible |
| nebelung | Nebelung | Tímido con extraños, muy leal y cariñoso con su familia cercana, tranquilo |
| cymric | Cymric | Juguetón, sociable, versión de pelo largo del Manx, buen saltador pese a la ausencia de cola |

### 4.6 Razas sin pelo o con pelaje atípico

| id | Nombre | Personalidad |
|---|---|---|
| sphynx | Sphynx | Extremadamente afectuoso, exigente de atención, intenso en su vínculo, nada indiferente |
| donskoy | Donskoy | Cariñoso, inteligente, sociable, busca contacto físico constante con su humano |
| peterbald | Peterbald | Curioso, sociable, activo, similar en personalidad al Oriental por su ascendencia |
| lykoi | Lykoi | Curioso, cauteloso al inicio pero cariñoso una vez que confía, conserva instintos algo "salvajes" |

### 4.7 Razas de pelo rizado (Rex)

| id | Nombre | Personalidad |
|---|---|---|
| devon-rex | Devon Rex | Travieso, juguetón, extrovertido, energía casi de duende y curiosidad sin límites |
| cornish-rex | Cornish Rex | Hiperactivo, atlético, muy vinculado a su humano, casi obsesivo por explorar cada rincón |
| selkirk-rex | Selkirk Rex | Tranquilo, paciente, tolerante, buen carácter con niños y otras mascotas |
| selkirk-rex-pelo-largo | Selkirk Rex Pelo Largo | Mismo temperamento calmado, algo más independiente en apariencia |
| laperm | LaPerm | Cariñoso, curioso, sociable, le gusta "ayudar" en las actividades de la casa |
| laperm-pelo-corto | LaPerm Pelo Corto | Mismo carácter afectuoso y curioso que su versión de pelo largo |
| tennessee-rex | Tennessee Rex | Aún poco documentada, pero reportada como sociable y de trato fácil |

### 4.8 Razas de orejas/cola distintiva

| id | Nombre | Personalidad |
|---|---|---|
| escoces-plegado | Escocés Plegado (Scottish Fold) | Tranquilo, adaptable, poco exigente, carácter dulce y algo reservado |
| escoces-recto | Escocés Recto (Scottish Straight) | Mismo temperamento tranquilo y dulce del Fold, sin la mutación de orejas |
| american-curl | American Curl | Afectuoso, juguetón, mantiene comportamiento de gatito toda su vida, sociable |
| american-curl-pelo-largo | American Curl Pelo Largo | Mismo carácter juguetón y afectuoso |
| manx | Manx | Leal, juguetón, buen saltador, se apega fuertemente a su familia |
| manx-tailed | Manx Tailed | Mismo temperamento leal del Manx, con cola presente |
| japanese-bobtail | Japanese Bobtail | Activo, vocal, sociable, considerado de "buena suerte" en la cultura japonesa |
| japanese-bobtail-pelo-largo | Japanese Bobtail Pelo Largo | Mismo carácter activo y sociable |
| american-bobtail | American Bobtail | Inteligente, juguetón, muy adaptable, buen compañero de niños |
| american-bobtail-shorthair | American Bobtail Shorthair | Mismo temperamento adaptable y juguetón |
| kurilian-bobtail | Kurilian Bobtail | Independiente, buen cazador (incluso de pesca), leal a su familia |
| kurilian-bobtail-pelo-largo | Kurilian Bobtail Pelo Largo | Mismo carácter independiente y cazador |
| pixiebob | Pixiebob | De comportamiento casi "canino", leal, entrenable, sigue a su humano por la casa |
| pixiebob-pelo-largo | Pixiebob Pelo Largo | Mismo temperamento leal y entrenable |
| toybob | Toybob | Sociable, cariñoso, de tamaño pequeño pero personalidad grande |

### 4.9 Razas de patas cortas

| id | Nombre | Personalidad |
|---|---|---|
| munchkin | Munchkin | Juguetón, curioso, ágil pese a sus patas cortas, sociable y activo |
| munchkin-pelo-largo | Munchkin Pelo Largo | Mismo carácter juguetón y curioso |
| minuet | Minuet | Dulce, sociable, tranquilo, buen compañero para casas con niños |
| minuet-pelo-largo | Minuet Pelo Largo | Mismo temperamento dulce y sociable |
| minuet-talls | Minuet Talls | Igual de sociable, con patas de longitud estándar |

### 4.10 Otras razas notables

| id | Nombre | Personalidad |
|---|---|---|
| bombay | Bombay | Sigiloso, afectuoso con su elegido, algo posesivo, disfruta la atención en sus propios términos |
| burmilla | Burmilla | (Ver arriba) dulce, curioso, tranquilo |
| burmilla-pelo-largo | Burmilla Pelo Largo | Mismo temperamento dulce |
| australian-mist | Australian Mist | Sociable, tolerante, buen carácter familiar, se adapta bien a la vida en interiores |
| khaomanee | Khaomanee | Alerta, curioso, muy vocal, considerado de "buena suerte" en Tailandia |
| cherubim | Cherubim | Raza en desarrollo, reportada como dócil y afectuosa |
| snowshoe | Snowshoe | Sociable, vocal (herencia siamesa), cariñoso, le gusta seguir a su humano por la casa |

**Total: 77 razas catalogadas**, agrupadas en 10 categorías morfológicas/temperamentales.

---

## 5. Clasificación popular por color/patrón de pelaje (Pt.4)

Esta capa se usa para el **Test de personalidad → resultado "¿Qué gato eres?"**
Debe presentarse siempre como **cultura popular / estereotipo**, nunca como
ciencia establecida.

| id | Categoría | Descripción (para mostrarse con disclaimer de "creencia popular") |
|---|---|---|
| naranja | 🧡 Gato naranja | Estudios y encuestas de dueños de gatos sugieren que los felinos naranjas suelen ser sociables, cariñosos y extrovertidos. Se cree que esto podría estar relacionado con la transmisión del gen del pelaje naranja, que es más común en machos (aproximadamente el 80%). Muchos dueños los describen como juguetones y amigables. |
| blanco | 🤍 Gato blanco | Los gatos con pelaje blanco pueden parecer más reservados o tranquilos, pero también pueden ser más sensibles a su entorno. Algunos pueden tener predisposición a problemas auditivos o de visión, lo que podría influir en su comportamiento más cauteloso. |
| negro | 🖤 Gato negro | Durante siglos, los gatos negros han sido víctimas de mitos y supersticiones. Sin embargo, los estudios han demostrado que suelen tener personalidades equilibradas y adaptables. Suelen ser tranquilos, seguros de sí mismos y afectuosos con sus dueños. |
| tabby | 🐅 Gato Tabby | El pelaje atigrado es uno de los más comunes y se encuentra en varias combinaciones de colores. Los gatos atigrados suelen ser enérgicos, curiosos y con un fuerte instinto cazador. Suelen disfrutar de la exploración y del juego activo. |
| gris | 🩶 Gato gris | Los gatos grises a menudo son descritos como inteligentes y equilibrados. Pueden mostrarse inicialmente reservados, pero una vez que ganan confianza con su Cat Lover, suelen ser muy leales y cariñosos. |
| bicolor | ⚫⚪ Gato bicolor (blanco y negro / gris y blanco) | Los gatos con combinaciones de colores suelen mostrar personalidades intermedias entre los colores que poseen. En general, suelen ser juguetones, sociables y con una actitud equilibrada entre independencia y cariño. |
| tricolor | 🐈‍⬛🧡🤍 Gato tricolor o carey | Los gatos de tres colores (calicó o carey) tienen una fuerte personalidad. Muchas personas los describen como independientes y enérgicos, con un carácter aventurero y a veces temperamental. Curiosamente, la mayoría de los gatos de tres colores son hembras debido a la genética de los cromosomas sexuales. |

**Nota de UX:** cada tarjeta de resultado debe incluir un texto tipo:
*"Esto no significa que los gatos [color] realmente tengan esta
personalidad — es una asociación popular que forma parte del lenguaje
cultural alrededor de los gatos."*

---

## 6. Test de personalidad — estructura de preguntas

Preguntas base (formato multiple choice, mapear cada opción A/B/C/D a un
color/patrón de la Sección 5):

1. **Cuando llegas a un lugar nuevo...**
   A. Quiero explorar todo. B. Busco inmediatamente a alguien con quien
   convivir. C. Primero observo. D. Busco un lugar tranquilo.

2. **Tu plan ideal es...**
   A. Una aventura. B. Reunirme con amigos. C. Hacer algo que me interese.
   D. Quedarme tranquilamente en casa.

3. **Cuando alguien invade tu espacio...**
   A. Me alejo y sigo con lo mío. B. Intento mantener la convivencia.
   C. Observo antes de reaccionar. D. Necesito recuperar mi espacio.

`[PENDIENTE]`: faltan definir el resto de preguntas necesarias y el mapeo
exacto de combinaciones de respuestas → los 7 resultados posibles
(naranja / blanco / negro / tabby / gris / bicolor / tricolor).

---

## 7. Media naranja gatuna (lógica de cruce)

**Inputs:** Raza de signo (Secciones 3 y 3.1) + Clasificación popular / patrón
(Sección 5)

**Output:** una raza "compatible" de la Sección 4, distinta a la raza de
signo, que se presenta como la "media naranja gatuna" del usuario — es decir,
la raza que mejor **convivería** con la raza que ya le tocó, no una raza
elegida por afinidad con la personalidad humana del usuario.

**Decisión resuelta:** el algoritmo de cruce completo (clasificación de las
77 razas en 4 perfiles de temperamento, compatibilidad entre perfiles, y el
pseudocódigo de combinación signo + patrón) está desarrollado en la
**Sección 11 — Tu gato ideal**.

---

## 8. Plantilla de ficha de raza (estructura idéntica para las 77 razas)

```json
{
  "id": "ragdoll",
  "nombre": "Ragdoll",
  "clasificacion": {
    "pelaje": "Semilargo",
    "caracteristica_fisica": "Ojos azules, cuerpo grande y musculoso",
    "patron_popular": "Colorpoint"
  },
  "tamano": "Grande",
  "apariencia": "",
  "nivel_actividad": "Bajo-moderado",
  "personalidad": {
    "rasgos_principales": ["dócil", "cariñoso", "dependiente emocionalmente"],
    "nivel_sociabilidad": "Alto",
    "relacion_humanos": "Se relaja completamente al cargarlo, busca contacto físico constante",
    "necesidades_estimulacion": ""
  },
  "signo_compatible": "Tauro",
  "categoria_madre": "Razas grandes / gentle giants",
  "fuentes": {
    "tica": "https://tica.org/breed/ragdoll/",
    "hills": "[PENDIENTE — URL específica de la raza]",
    "royal_canin": "[PENDIENTE — URL específica de la raza]",
    "purina": "[PENDIENTE — URL específica de la raza]"
  }
}
```

**Campos ya cubiertos por este documento:** id, nombre, personalidad
(rasgos principales), categoría madre, signo(s) asociado(s).

**Campos que faltan por completar para cada una de las 77 razas antes de
publicar:** tamaño exacto, tipo de pelaje detallado, apariencia física,
nivel de actividad, nivel de sociabilidad, necesidades de estimulación, y
**enlaces específicos por raza** de TICA / Hill's / Royal Canin / Purina.

---

## 9. Fuentes generales (a nivel de sitio, no por raza)

| Fuente | URL general |
|---|---|
| TICA — Browse All Breeds | https://tica.org/ticas-breeds/browse-all-breeds/ |
| Hill's Pet — Cat Care Center | https://www.hillspet.com/pet-care-center?species=cat&category=breed |
| Royal Canin MX — Razas de gatos | https://www.royalcanin.com/mx/cats/breeds |
| Purina UK — Cat breeds | https://www.purina.co.uk/find-a-pet/cat-breeds |

Cada ficha individual (Sección 8) debe reemplazar estas URLs generales por
el enlace específico de esa raza cuando esté disponible.

---

## 10. Resumen de pendientes para desarrollo

| # | Pendiente | Bloquea |
|---|---|---|
| 1 | Definir tamaño del ciclo anual y confirmar lista de 12 (o N) razas | Sección 01 / 02 |
| 2 | Redactar preguntas completas del test + tabla de mapeo respuesta→resultado | Sección 04 |
| 3 | Definir algoritmo de cruce para "media naranja gatuna" | Sección 07 |
| 4 | Completar campos físicos (tamaño, pelaje detallado, apariencia, actividad) por raza | Sección 08 |
| 5 | Recolectar enlaces específicos por raza en TICA/Hill's/Royal Canin/Purina | Sección 08/09 |

---

*Documento generado combinando: (a) arquitectura de navegación del análisis
de estructura del sitio, y (b) catálogo de razas y patrones ya redactado
por el usuario. No se inventaron ni sustituyeron razas o descripciones de
personalidad — todo el contenido de las Secciones 4 y 5 proviene
directamente del documento original del usuario.*

---

## 11. Tu gato ideal — compañero felino según signo + patrón

Esta sección responde a la pregunta final del recorrido: dado el resultado de tu
**signo** (Secciones 3 y 3.1) y el resultado de tu **patrón/clasificación popular**
(Sección 5), ¿qué raza sería tu compañero felino ideal? **Importante: esto no busca
una raza que combine con la personalidad de una persona, sino una raza de gato que
conviviría bien con la raza de gato que ya te tocó** — es compatibilidad entre gatos,
no entre un gato y un humano.

### 11.1 Principio de compatibilidad entre razas
Guías de crianza y comportamiento felino (consultadas en fuentes especializadas en
convivencia multi-gato) coinciden en un principio general: **la compatibilidad entre
razas depende más del nivel de energía y de la necesidad de interacción que de
cualquier otro factor**. Dos gatos muy activos suelen llevarse mejor entre sí que un
gato muy activo con uno plácido que prefiere la soledad; dos gatos independientes
toleran bien la distancia mutua sin conflicto; y los gatos que buscan cercanía
constante (ya sea vocal o físicamente) tienden a reforzarse entre ellos. Bajo ese
principio se construyó la clasificación siguiente.

### 11.2 Los 4 perfiles de temperamento

| Perfil | Descripción | Compatible con |
|---|---|---|
| **A — Activo / Enérgico** | Necesita juego y estímulo constante, alto nivel de actividad física | Otro perfil A |
| **B — Sociable / Vocal** | Extrovertido, busca interacción constante (con humanos u otros gatos) | Perfil B o D |
| **C — Tranquilo / Independiente** | Bajo nivel de exigencia, cómodo con la distancia y la calma | Otro perfil C |
| **D — Cariñoso / Dependiente** | Necesita cercanía y contacto físico/emocional constante | Perfil D o B |

*(Los perfiles B y D se combinan bien entre sí porque ambos giran en torno a la
necesidad de cercanía — uno la expresa de forma más vocal/social, el otro de forma
más física/afectiva.)*

### 11.3 Clasificación de las 77 razas por perfil de temperamento

*(Clasificación derivada de las características ya documentadas en la Sección 4,
agrupadas según el principio de la Sección 11.1 — no son datos nuevos, es una
re-organización de la personalidad ya descrita por raza.)*

**Razas "clásicas" de pelo corto**

| Raza | Perfil |
|---|---|
| Abisinio | A — Activo / Enérgico |
| American Shorthair | C — Tranquilo / Independiente |
| American Wirehair | C — Tranquilo / Independiente |
| Británico de Pelo Corto | C — Tranquilo / Independiente |
| Chartreux | C — Tranquilo / Independiente |
| Egyptian Mau (Mau Egipcio) | A — Activo / Enérgico |
| Ruso Azul | C — Tranquilo / Independiente |
| Korat | D — Cariñoso / Dependiente |
| Habana (Havana Brown) | B — Sociable / Vocal |
| Singapura | A — Activo / Enérgico |

**Razas orientales / vocales**

| Raza | Perfil |
|---|---|
| Siamés | B — Sociable / Vocal |
| Oriental de Pelo Corto | B — Sociable / Vocal |
| Oriental de Pelo Largo | B — Sociable / Vocal |
| Balinés | B — Sociable / Vocal |
| Tailandés (Thai) | D — Cariñoso / Dependiente |
| Tonkinés | B — Sociable / Vocal |
| Burmés | D — Cariñoso / Dependiente |
| Birmilla | D — Cariñoso / Dependiente |

**Razas grandes / "gentle giants"**

| Raza | Perfil |
|---|---|
| Maine Coon | B — Sociable / Vocal |
| Maine Coon Polydactyl | B — Sociable / Vocal |
| Noruego de Bosque | A — Activo / Enérgico |
| Siberiano | B — Sociable / Vocal |
| Ragdoll | D — Cariñoso / Dependiente |
| Chausie | A — Activo / Enérgico |
| Highlander | A — Activo / Enérgico |

**Razas exóticas con patrón salvaje**

| Raza | Perfil |
|---|---|
| Bengalí | A — Activo / Enérgico |
| Bengalí de Pelo Largo | A — Activo / Enérgico |
| Savannah | A — Activo / Enérgico |
| Ocicat | B — Sociable / Vocal |
| Toyger | B — Sociable / Vocal |
| Serengeti | A — Activo / Enérgico |

**Razas de pelo largo / semi-largo**

| Raza | Perfil |
|---|---|
| Persa | C — Tranquilo / Independiente |
| Himalayo | C — Tranquilo / Independiente |
| Exótico de Pelo Corto | D — Cariñoso / Dependiente |
| British Longhair | C — Tranquilo / Independiente |
| Turco Angora | B — Sociable / Vocal |
| Turco Van | A — Activo / Enérgico |
| Nebelung | D — Cariñoso / Dependiente |
| Cymric | A — Activo / Enérgico |

**Razas sin pelo o con pelaje atípico**

| Raza | Perfil |
|---|---|
| Sphynx | D — Cariñoso / Dependiente |
| Donskoy | D — Cariñoso / Dependiente |
| Peterbald | B — Sociable / Vocal |
| Lykoi | C — Tranquilo / Independiente |

**Razas de pelo rizado (Rex)**

| Raza | Perfil |
|---|---|
| Devon Rex | A — Activo / Enérgico |
| Cornish Rex | A — Activo / Enérgico |
| Selkirk Rex | C — Tranquilo / Independiente |
| Selkirk Rex Pelo Largo | C — Tranquilo / Independiente |
| LaPerm | D — Cariñoso / Dependiente |
| LaPerm Pelo Corto | D — Cariñoso / Dependiente |
| Tennessee Rex | B — Sociable / Vocal |

**Razas de orejas/cola distintiva**

| Raza | Perfil |
|---|---|
| Escocés Plegado (Scottish Fold) | C — Tranquilo / Independiente |
| Escocés Recto (Scottish Straight) | C — Tranquilo / Independiente |
| American Curl | D — Cariñoso / Dependiente |
| American Curl Pelo Largo | D — Cariñoso / Dependiente |
| Manx | B — Sociable / Vocal |
| Manx Tailed | B — Sociable / Vocal |
| Japanese Bobtail | B — Sociable / Vocal |
| Japanese Bobtail Pelo Largo | B — Sociable / Vocal |
| American Bobtail | B — Sociable / Vocal |
| American Bobtail Shorthair | B — Sociable / Vocal |
| Kurilian Bobtail | C — Tranquilo / Independiente |
| Kurilian Bobtail Pelo Largo | C — Tranquilo / Independiente |
| Pixiebob | D — Cariñoso / Dependiente |
| Pixiebob Pelo Largo | D — Cariñoso / Dependiente |
| Toybob | D — Cariñoso / Dependiente |

**Razas de patas cortas**

| Raza | Perfil |
|---|---|
| Munchkin | A — Activo / Enérgico |
| Munchkin Pelo Largo | A — Activo / Enérgico |
| Minuet | D — Cariñoso / Dependiente |
| Minuet Pelo Largo | D — Cariñoso / Dependiente |
| Minuet Talls | D — Cariñoso / Dependiente |

**Otras razas notables**

| Raza | Perfil |
|---|---|
| Bombay | D — Cariñoso / Dependiente |
| Burmilla | C — Tranquilo / Independiente |
| Burmilla Pelo Largo | C — Tranquilo / Independiente |
| Australian Mist | B — Sociable / Vocal |
| Khaomanee | B — Sociable / Vocal |
| Cherubim | D — Cariñoso / Dependiente |
| Snowshoe | B — Sociable / Vocal |

### 11.4 Perfil y compañero ideal por patrón/clasificación popular

Cada resultado del test de personalidad (Sección 5) tiene un perfil de temperamento
dominante. El compañero ideal se elige entre razas del mismo perfil (o del perfil
compatible, según la tabla 11.2).

**Gato naranja** — Perfil dominante: **B (Sociable / Vocal)**
*sociable, cariñoso, extrovertido y juguetón — busca constantemente interacción.*
Compañero ideal: razas de perfil B/D. Ejemplos: Korat, Habana (Havana Brown), Siamés, entre otras del mismo perfil (ver tabla 11.3).

**Gato blanco** — Perfil dominante: **C (Tranquilo / Independiente)**
*reservado, tranquilo y sensible a su entorno — valora la calma más que la compañía constante.*
Compañero ideal: razas de perfil C. Ejemplos: American Shorthair, American Wirehair, Británico de Pelo Corto, entre otras del mismo perfil (ver tabla 11.3).

**Gato negro** — Perfil dominante: **C (Tranquilo / Independiente)**
*equilibrado y adaptable, tranquilo y seguro de sí mismo — cómodo tanto solo como acompañado.*
Compañero ideal: razas de perfil C. Ejemplos: American Shorthair, American Wirehair, Británico de Pelo Corto, entre otras del mismo perfil (ver tabla 11.3).

**Gato Tabby** — Perfil dominante: **A (Activo / Enérgico)**
*enérgico, curioso y con fuerte instinto cazador — necesita estímulo y movimiento constante.*
Compañero ideal: razas de perfil A. Ejemplos: Abisinio, Egyptian Mau (Mau Egipcio), Singapura, entre otras del mismo perfil (ver tabla 11.3).

**Gato gris** — Perfil dominante: **D (Cariñoso / Dependiente)**
*inicialmente reservado, pero profundamente leal y cariñoso una vez que confía.*
Compañero ideal: razas de perfil D/B. Ejemplos: Korat, Habana (Havana Brown), Siamés, entre otras del mismo perfil (ver tabla 11.3).

**Gato bicolor** — Perfil dominante: **B (Sociable / Vocal)**
*juguetón, sociable, con un equilibrio entre independencia y cariño.*
Compañero ideal: razas de perfil B/D. Ejemplos: Korat, Habana (Havana Brown), Siamés, entre otras del mismo perfil (ver tabla 11.3).

**Gato tricolor o carey** — Perfil dominante: **A (Activo / Enérgico)**
*independiente pero enérgico, con un carácter aventurero y a veces temperamental.*
Compañero ideal: razas de perfil A. Ejemplos: Abisinio, Egyptian Mau (Mau Egipcio), Singapura, entre otras del mismo perfil (ver tabla 11.3).

### 11.5 Cómo se combina con el signo (algoritmo)

```
entrada: raza_de_signo (Sección 3.1), patrón_resultado (Sección 5)

1. perfil_signo   = CLUSTER[raza_de_signo]        # tabla 11.3
2. perfil_patron  = CLUSTER_PATRON[patrón_resultado]  # tabla 11.4
3. si perfil_signo == perfil_patron:
       perfil_objetivo = perfil_signo   # ambos coinciden, resultado reforzado
   si no:
       perfil_objetivo = perfil_patron  # el patrón (test de personalidad) tiene prioridad,
                                        # porque refleja cómo la persona se describe a sí misma,
                                        # no solo su fecha de nacimiento
4. compañero_ideal = raza aleatoria (o la más afín) dentro de COMPATIBLE[perfil_objetivo],
                     excluyendo la raza_de_signo para evitar repetir el mismo resultado

salida: compañero_ideal
```

**Nota:** cuando el perfil del signo y el del patrón coinciden, se puede mostrar un
mensaje reforzado (ej. "tu gato de nacimiento y tu personalidad apuntan al mismo tipo
de energía — tu media naranja gatuna comparte ese mismo ritmo"). Cuando difieren, el
mensaje puede explicar el contraste (ej. "tu gato de nacimiento es tranquilo, pero tu
personalidad de test es más activa — tu compañero ideal necesita seguirte el paso a ti,
no a tu gato de nacimiento").
