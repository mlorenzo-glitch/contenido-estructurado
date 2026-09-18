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

### 1.1 Parte 1 — Interacción: fecha de nacimiento → signo → raza

Esta parte define el recorrido interactivo que debe seguir la aplicación:

1. Pedir la **fecha de nacimiento** al usuario (día, mes, año).
2. Determinar el **signo zodiacal** a partir del día y el mes.
3. (Opcional pero recomendado) Ajustar el **año** según la fecha del Año Nuevo chino si se quiere coherencia con la sección 3.1.
4. Tomar la **lista de razas** asociada al signo (Sección 3) y calcular la posición dentro del ciclo usando el año.
5. Mostrar la **raza resultado**, junto con sus datos (características, personalidad, fuentes) y el animal chino complementario si se desea.

Tabla de rangos para los signos (día/mes):

| Signo | Rango |
|---|---|
| Aries | 21 de marzo – 19 de abril |
| Tauro | 20 de abril – 20 de mayo |
| Géminis | 21 de mayo – 20 de junio |
| Cáncer | 21 de junio – 22 de julio |
| Leo | 23 de julio – 22 de agosto |
| Virgo | 23 de agosto – 22 de septiembre |
| Libra | 23 de septiembre – 22 de octubre |
| Escorpio | 23 de octubre – 21 de noviembre |
| Sagitario | 22 de noviembre – 21 de diciembre |
| Capricornio | 22 de diciembre – 19 de enero |
| Acuario | 20 de enero – 18 de febrero |
| Piscis | 19 de febrero – 20 de marzo |

Algoritmo (resumen):

```
entrada: fecha_nacimiento (día, mes, año)

1. signo = getSign(día, mes)                          # usar los rangos de arriba
2. año_ajustado = ajustarPorCNY(año, mes, día)        # opcional (ver Sección 3.1)
3. lista_razas = RAZAS_POR_SIGNO[signo]               # lista de la Sección 3
4. n = len(lista_razas)
5. índice = (año_ajustado - AÑO_BASE) mod n          # AÑO_BASE = 2000 (ejemplo)
6. raza_resultado = lista_razas[índice]

salida: raza_resultado, posición_ciclo = índice+1
```

Notas:
- `AÑO_BASE` es un año de referencia elegido para alinear ejemplos (en este documento usamos 2000). Cambiarlo altera los ejemplos, no la lógica.
- Si se requiere usar el Año Nuevo chino para `año_ajustado`, aplicar la regla: si la fecha de nacimiento está antes de la fecha del CNY de ese año, restar 1 al año.

Ejemplo rápido (sin ajuste CNY):
- Fecha: 14/04/1995 → Signo: Aries (21 mar–19 abr)
- Lista Aries (7 razas), AÑO_BASE=2000 → n = 7
- índice = (1995 - 2000) mod 7 = (-5) mod 7 = 2  → posición 3
- Raza resultado = la tercera raza de Aries (según Sección 3, p. ej. Savannah)

Snippet de formulario HTML + JS (ejemplo mínimo para prototipo):

```html
<!-- Formulario simple -->
<form id="birthform">
        <label>Fecha de nacimiento: <input type="date" id="bdate" required></label>
        <button type="button" id="calc">Calcular mi michi</button>
</form>
<div id="resultado"></div>

<script>
const RAZAS_POR_SIGNO = {
        'Aries': ['Bengalí','Bengalí de Pelo Largo','Savannah','Toyger','Egyptian Mau','Chausie','Serengeti'],
        /* completar con las demás listas desde la Sección 3 */
};
const AÑO_BASE = 2000;

function getSign(d){
        const m = d.getMonth()+1; // 1-12
        const day = d.getDate();
        // Rangos simplificados
        if ((m==3 && day>=21) || (m==4 && day<=19)) return 'Aries';
        if ((m==4 && day>=20) || (m==5 && day<=20)) return 'Tauro';
        if ((m==5 && day>=21) || (m==6 && day<=20)) return 'Géminis';
        if ((m==6 && day>=21) || (m==7 && day<=22)) return 'Cáncer';
        if ((m==7 && day>=23) || (m==8 && day<=22)) return 'Leo';
        if ((m==8 && day>=23) || (m==9 && day<=22)) return 'Virgo';
        if ((m==9 && day>=23) || (m==10 && day<=22)) return 'Libra';
        if ((m==10 && day>=23) || (m==11 && day<=21)) return 'Escorpio';
        if ((m==11 && day>=22) || (m==12 && day<=21)) return 'Sagitario';
        if ((m==12 && day>=22) || (m==1 && day<=19)) return 'Capricornio';
        if ((m==1 && day>=20) || (m==2 && day<=18)) return 'Acuario';
        return 'Piscis';
}

document.getElementById('calc').addEventListener('click',()=>{
        const v = document.getElementById('bdate').value;
        if(!v){ document.getElementById('resultado').innerText='Indica tu fecha'; return; }
        const d = new Date(v);
        const signo = getSign(d);
        const año = d.getFullYear();
        // Para prototipo omitimos ajuste CNY; para producción usar ajustarPorCNY
        const lista = RAZAS_POR_SIGNO[signo] || [];
        const n = lista.length || 1;
        const idx = ((año - AÑO_BASE) % n + n) % n; // módulo positivo
        const raza = lista[idx] || '—';
        document.getElementById('resultado').innerHTML = `<strong>Signo:</strong> ${signo}<br><strong>Raza:</strong> ${raza} <small>(posición ${idx+1} de ${n})</small>`;
});
</script>
```

Dónde mejorar para producción:
- Implementar `ajustarPorCNY(año, mes, día)` usando la tabla de Sección 3.1 o una API de fechas de Año Nuevo chino.
- Completar `RAZAS_POR_SIGNO` con todas las listas de la Sección 3 (los slugs en Sección 4 pueden mapearse aquí).
- Añadir validaciones, internacionalización y enlaces directos a las fichas de raza.

Esta sección deja documentado el camino solicitado: pedir fecha → ligar a signo → usar año para seleccionar la raza dentro del ciclo del signo, y ofrece un ejemplo ejecutable mínimo para prototipado rápido.

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

## 3.2 Ciclos por signo (mapa de posiciones → años)

Para poder vincular un **año de nacimiento** con una raza dentro del ciclo de cada signo, proponemos el siguiente método de trabajo (editable según requerimientos del producto):

- Elegir un **año de referencia** por signo (por simplicidad aquí usamos el año 2000 como posición 1 para todos los signos en los ejemplos). 
- Cada ciclo tiene longitud igual al número de razas listadas para ese signo (6 o 7). Los años se repiten cada N años, donde N = longitud del ciclo del signo.
- Las columnas "Años ejemplo" contienen años de ejemplo separados por la longitud del ciclo; cada año enlaza a la subsección del signo correspondiente en este documento.

<a id="aries"></a>
### ♈ Aries — Ciclo (7 razas)
| Pos | Raza |
|---:|---|
| 1 | Bengalí |
| 2 | Bengalí de Pelo Largo |
| 3 | Savannah |
| 4 | Toyger |
| 5 | Egyptian Mau |
| 6 | Chausie |
| 7 | Serengeti |

Años ejemplo (cada 7 años): [2000](#aries), [2007](#aries), [2014](#aries), [2021](#aries), [2028](#aries)

<a id="tauro"></a>
### ♉ Tauro — Ciclo (6 razas)
| Pos | Raza |
|---:|---|
| 1 | Persa |
| 2 | Himalayo |
| 3 | Exótico de Pelo Corto |
| 4 | Ragdoll |
| 5 | Chartreux |
| 6 | American Wirehair |

Años ejemplo (cada 6 años): [2000](#tauro), [2006](#tauro), [2012](#tauro), [2018](#tauro), [2024](#tauro)

<a id="geminis"></a>
### ♊ Géminis — Ciclo (7 razas)
| Pos | Raza |
|---:|---|
| 1 | Siamés |
| 2 | Oriental de Pelo Corto |
| 3 | Oriental de Pelo Largo |
| 4 | Balinés |
| 5 | Tailandés (Thai) |
| 6 | Tonkinés |
| 7 | Burmés |

Años ejemplo (cada 7 años): [2000](#geminis), [2007](#geminis), [2014](#geminis), [2021](#geminis), [2028](#geminis)

<a id="cancer"></a>
### ♋ Cáncer — Ciclo (7 razas)
| Pos | Raza |
|---:|---|
| 1 | Munchkin |
| 2 | Munchkin Pelo Largo |
| 3 | Minuet |
| 4 | Minuet Pelo Largo |
| 5 | Minuet Talls |
| 6 | Snowshoe |
| 7 | Nebelung |

Años ejemplo (cada 7 años): [2000](#cancer), [2007](#cancer), [2014](#cancer), [2021](#cancer), [2028](#cancer)

<a id="leo"></a>
### ♌ Leo — Ciclo (7 razas)
| Pos | Raza |
|---:|---|
| 1 | Maine Coon |
| 2 | Maine Coon Polydactyl |
| 3 | Noruego de Bosque |
| 4 | Siberiano |
| 5 | Highlander |
| 6 | British Longhair |
| 7 | Turco Angora |

Años ejemplo (cada 7 años): [2000](#leo), [2007](#leo), [2014](#leo), [2021](#leo), [2028](#leo)

<a id="virgo"></a>
### ♍ Virgo — Ciclo (6 razas)
| Pos | Raza |
|---:|---|
| 1 | Abisinio |
| 2 | Ocicat |
| 3 | Singapura |
| 4 | Korat |
| 5 | Habana |
| 6 | Lykoi |

Años ejemplo (cada 6 años): [2000](#virgo), [2006](#virgo), [2012](#virgo), [2018](#virgo), [2024](#virgo)

<a id="libra"></a>
### ♎ Libra — Ciclo (6 razas)
| Pos | Raza |
|---:|---|
| 1 | Británico de Pelo Corto |
| 2 | Escocés Recto |
| 3 | Escocés Plegado |
| 4 | Birmilla |
| 5 | Burmilla |
| 6 | Burmilla Pelo Largo |

Años ejemplo (cada 6 años): [2000](#libra), [2006](#libra), [2012](#libra), [2018](#libra), [2024](#libra)

<a id="escorpio"></a>
### ♏ Escorpio — Ciclo (7 razas)
| Pos | Raza |
|---:|---|
| 1 | Bombay |
| 2 | Cymric |
| 3 | LaPerm |
| 4 | LaPerm Pelo Corto |
| 5 | Tennessee Rex |
| 6 | Pixiebob |
| 7 | Pixiebob Pelo Largo |

Años ejemplo (cada 7 años): [2000](#escorpio), [2007](#escorpio), [2014](#escorpio), [2021](#escorpio), [2028](#escorpio)

<a id="sagitario"></a>
### ♐ Sagitario — Ciclo (6 razas)
| Pos | Raza |
|---:|---|
| 1 | Devon Rex |
| 2 | Cornish Rex |
| 3 | American Bobtail |
| 4 | American Bobtail Shorthair |
| 5 | Kurilian Bobtail |
| 6 | Kurilian Bobtail Pelo Largo |

Años ejemplo (cada 6 años): [2000](#sagitario), [2006](#sagitario), [2012](#sagitario), [2018](#sagitario), [2024](#sagitario)

<a id="capricornio"></a>
### ♑ Capricornio — Ciclo (6 razas)
| Pos | Raza |
|---:|---|
| 1 | Ruso Azul |
| 2 | American Shorthair |
| 3 | Manx |
| 4 | Manx Tailed |
| 5 | Peterbald |
| 6 | Donskoy |

Años ejemplo (cada 6 años): [2000](#capricornio), [2006](#capricornio), [2012](#capricornio), [2018](#capricornio), [2024](#capricornio)

<a id="acuario"></a>
### ♒ Acuario — Ciclo (6 razas)
| Pos | Raza |
|---:|---|
| 1 | Turco Van |
| 2 | American Curl |
| 3 | American Curl Pelo Largo |
| 4 | Sphynx |
| 5 | Selkirk Rex |
| 6 | Selkirk Rex Pelo Largo |

Años ejemplo (cada 6 años): [2000](#acuario), [2006](#acuario), [2012](#acuario), [2018](#acuario), [2024](#acuario)

<a id="piscis"></a>
### ♓ Piscis — Ciclo (6 razas)
| Pos | Raza |
|---:|---|
| 1 | Khaomanee |
| 2 | Cherubim |
| 3 | Japanese Bobtail |
| 4 | Japanese Bobtail Pelo Largo |
| 5 | Australian Mist |
| 6 | Toybob |

Años ejemplo (cada 6 years): [2000](#piscis), [2006](#piscis), [2012](#piscis), [2018](#piscis), [2024](#piscis)

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
| burmilla | Burmilla | Dulce, curioso, tranquilo |
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

**Inputs:** Raza de nacimiento (Sección 2) + Signo (Sección 3) + Clasificación
popular (Sección 5)

**Output:** una raza "compatible" de la Sección 4, distinta a la raza de
nacimiento, que se presenta como la "media naranja gatuna" del usuario.

`[PENDIENTE]`: falta definir la tabla/algoritmo de cruce exacto (p. ej. una
matriz de compatibilidad por clasificación popular × elemento del signo,
o un sistema de puntos). Este documento deja la base de datos lista para
que esa lógica se construya sobre ella.

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
