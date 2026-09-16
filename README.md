# la Argentina — arreglos para 1.13.11

Arreglos para el mod [la Argentina](https://steamcommunity.com/sharedfiles/filedetails/?id=2982907360)
de Victoria 3, verificados contra el juego **1.13.11 (Matcha)** con todos los DLC.

El repo trae el mod entero con los arreglos ya aplicados y los parches sueltos, por si preferís
revisarlos uno por uno.

## Resultado medido

Partida nueva en modo observador, mundo entero, tres años de juego (1836–1839), sin BPM:

| Configuración | Líneas en `error.log` | Segundos por mes |
|---|---:|---:|
| Juego sin mods | 1.729 | 27,5 |
| la Argentina + Argentina nuevos eventos | 7.876 | — |
| **Los dos mods con estos arreglos** | **1.825** | **26,2** |

El mod pasa de agregar **6.147 líneas de error** a agregar **96**. El rendimiento es el mismo
que el del juego sin mods: no hay problema de performance.

Verificado además en el archivo de guardado de esa partida: Uruguay existe con capital en
Montevideo, Argentina tiene activas sus journal entries (Rosas, Chaco, Malvinas, Libertad de
Vientres, Patagonia, Tarija, Malones), Rosas aparece como personaje, el caciquismo se aplica
y la ley Local Caudillos funciona.

## Los problemas y sus arreglos

### 1. Tres estados del juego se eliminan pero trece archivos los siguen nombrando

El mod divide `STATE_URUGUAY`, `STATE_PATAGONIA` y `STATE_ARAUCANIA` en estados nuevos y borra
los originales, pero no toca los archivos del juego que los mencionan. Consecuencias:

- **Cuatro países quedan con capital en un estado inexistente**, Uruguay entre ellos
  (`common/country_definitions/00_countries.txt`). Es el mejor candidato a los crashes que no
  dejan rastro en los logs.
- Uruguay como país liberable apunta a un estado que no existe (`country_creation`).
- Las Provincias Unidas del Río de la Plata no se pueden formar (`country_formation`).
- Condiciones que nunca se cumplen en `scripted_triggers`, `power_bloc_names`,
  `dynamic_country_names`, `geographic_regions/06_old_strategic_regions`,
  `journal_entries/02_paraguay` y `events/brazil/gran_colombia`.
- Tres plantillas de personajes de Argentina, Chile y Uruguay con `home_region` inválida.

**Arreglo:** copias corregidas de esos trece archivos. Donde la referencia era a un solo estado
se usa el sucesor principal; donde era una lista, se expande a todos los sucesores:

| Estado eliminado | Sucesores |
|---|---|
| `STATE_URUGUAY` | Montevideo, Banda Oriental, Paysandú |
| `STATE_PATAGONIA` | Río Negro, Neuquén, Chubut, Santa Cruz |
| `STATE_ARAUCANIA` | Los Lagos, Aysén, Magallanes, Tierra del Fuego |

### 2. El propio mod nombra el Uruguay viejo

`common/history/military_formations/02_military_formations_south_america.txt` crea la formación
inicial en `STATE_URUGUAY`, que el mod ya borró.

### 3. Falta un `=` y un evento no existe

`events/malvinas_events.txt:201` dice `islas_malvinas.1 {` en vez de `islas_malvinas.1 = {`.
El evento nunca se define y la journal entry de las Malvinas lo llama igual.

### 4. Argentina pierde el caciquismo de Colossus of the South

`common/history/countries/arg - argentina.txt` es una copia anterior a la expansión y perdió
`initialise_caciquismo_effect`. Sin eso Argentina nunca recibe `caciquismo_var`, que es lo que
habilita el clientelismo electoral y el fraude.

**Arreglo:** `common/on_actions/zz_fix_caciquismo.txt` lo aplica al empezar la partida, sin tocar
el resto del historial del país.

### 5. Dos regiones geográficas borradas

La copia de `common/geographic_regions/02_geographic_regions_america.txt` elimina
`geographic_region_north_andes` y `geographic_region_southern_cone`, que el juego usa en la
journal entry de la Gran Colombia.

**Arreglo:** `common/geographic_regions/zz_fix_regiones_restauradas.txt` las restaura, adaptadas
al mapa del mod: el Cono Sur incluye Montevideo, Banda Oriental y Paysandú en lugar del Uruguay
viejo.

### 6. Un estado sin `subsistence_building`

`STATE_SANTIAGO_DEL_ESTERO` es el único estado del mod sin ese campo. Se le pone
`building_subsistence_farm`, igual que sus vecinos.

### 7. Dos ideologías propias que no las recibe nadie

El mod define `ideology_autonomist` e `ideology_pampas_expansionism`, con sus posturas ante cada
grupo de leyes, pero ningún evento, personaje ni grupo de interés las recibe: quedan escritas y
muertas.

**Arreglo:** `common/on_actions/zz_fix_ideologias_huerfanas.txt` se las da a quien corresponde por
su propio contenido. La autonomista, que apoya caudillos locales y arrendatarios, va a los
terratenientes de los países de cultura platense. La expansionista, que empuja la colonización de
frontera y la inmigración abierta, va a las fuerzas armadas de la Confederación.

### 8. Textos faltantes

Sin traducción inglesa: los rasgos de idioma `language_chono`, `language_kawesqar` y
`language_selknam`, la religión `slave` y la journal entry `random_character`. Se ven como claves
crudas en pantalla.

## El mod de eventos va aparte

Los arreglos de [Argentina nuevos eventos](https://steamcommunity.com/sharedfiles/filedetails/?id=3750562635)
y el compatch con Better Politics Mod están en su propio repo:
[argentina-nuevos-eventos-fixes](https://github.com/Juanpintoselso33/argentina-nuevos-eventos-fixes).

## Cómo usarlo

- **Para el mod completo ya corregido:** copiá `la-argentina/` sobre el mod.
- **Para revisar cambio por cambio:** están en `parches/`, en formato diff unificado contra el
  original.
- **Para usarlo como mod local:** poné `la-argentina/` en
  `Documents/Paradox Interactive/Victoria 3/mod/` con su `.mod` apuntando a esa carpeta.

## Lo que está bien y no hace falta tocar

Durante la revisión también se verificó, sin encontrar problemas: el mapa no tiene provincias
duplicadas ni inexistentes, los hubs caen dentro de su estado, los ids de estado no chocan con los
del juego, no hay referencias rotas a leyes, edificios, métodos de producción, tecnologías,
culturas ni religiones, y no hay objetos definidos dos veces dentro del mod.

Sobre Salta: que Bolivia arranque con nueve provincias adentro no es un error. El juego base hace
lo mismo con Jujuy y le da Antofagasta entera a Bolivia, y ese bloque limita con territorio
boliviano en Jujuy y Antofagasta. Es la Puna de Atacama, que Bolivia reclamó hasta 1899.
