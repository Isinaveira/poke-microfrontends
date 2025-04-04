# Pokémon Team Builder - Arquitectura Microfrontend 🚀

## Visión General 🎯

Este proyecto implementa un "Constructor de Equipos Pokémon" utilizando una arquitectura de microfrontends moderna. El objetivo principal es servir como un proyecto de aprendizaje práctico para tecnologías como React, Vite, Module Federation, Redux, TypeScript y Docker. La aplicación permite a los usuarios explorar Pokémon, construir un equipo temporal (máximo 6), nombrarlo, guardarlo usando `localStorage` y consultar los equipos previamente guardados.

## Tecnologías Principales 🛠️

* **Lenguaje:** ![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
* **Librería UI:** ![React](https://img.shields.io/badge/-React-61DAFB?style=flat-square&logo=react&logoColor=black)
* **Build Tool / Dev Server:** ![Vite](https://img.shields.io/badge/-Vite-646CFF?style=flat-square&logo=vite&logoColor=white)
* **Estado Global:** ![Redux](https://img.shields.io/badge/-Redux-764ABC?style=flat-square&logo=redux&logoColor=white) (con Redux Toolkit)
* **Module Federation:** 🧩 `vite-plugin-federation` (@originjs)
* **Routing:** 🗺️ React Router DOM
* **Persistencia:** 💾 `localStorage` del navegador
* **Contenerización:** ![Docker](https://img.shields.io/badge/-Docker-2496ED?style=flat-square&logo=docker&logoColor=white) & Docker Compose
* **API Externa:** ![PokeAPI](https://img.shields.io/badge/-PokeAPI-EF5350?style=flat-square)

## Componentes de la Arquitectura (Servicios Docker 🐳)

La aplicación se divide en varios servicios independientes, cada uno corriendo en su propio contenedor Docker y construido con Vite + React + TS.

---

### 1. 🏠 `poke-app-shell` (Aplicación Contenedora)

* **Rol:** Orquestador principal. Punto de entrada de la aplicación.
* **Responsabilidades:**
    * **Layout Principal:** Define la estructura visual base (cabecera, área de contenido).
    * **Enrutamiento 🗺️:** Gestiona las rutas (`/`, `/my-teams`) con `react-router-dom`.
    * **Estado Global ![Redux](https://img.shields.io/badge/-Redux-764ABC?style=flat-square&logo=redux&logoColor=white):** Aloja la **única store global** de Redux Toolkit. Define slices para `selectedPokemon`, `currentTeam`, `savedTeams`.
    * **Persistencia 💾:** Carga/Guarda `savedTeams` en `localStorage`.
    * **Carga de MFEs 🧩:** Usa `vite-plugin-federation`, `React.lazy` y `<Suspense>` para cargar componentes remotos.
    * **Coordinación:** Provee la store (`<Provider>`), despacha acciones a Redux basadas en callbacks de los MFEs, y selecciona datos del estado para pasarlos como props a los MFEs.
* **Puerto Docker (Ej):** `3000`

---

### 2. 🧩 `poke-mfe-pokedex` (Microfrontend: Lista Pokedex)

* **Rol:** Mostrar la lista de Pokémon disponibles.
* **Responsabilidades:**
    * Obtener datos de la ![PokeAPI](https://img.shields.io/badge/-PokeAPI-EF5350?style=flat-square).
    * Renderizar la lista.
    * Notificar al Shell (vía prop callback `onPokemonSelect`) cuando un Pokémon es seleccionado.
    * Exponer `PokedexList` vía Module Federation.
* **Puerto Docker (Ej):** `3001`

---

### 3. 🧩 `poke-mfe-detail` (Microfrontend: Detalle Pokémon)

* **Rol:** Mostrar información detallada de un Pokémon específico.
* **Responsabilidades:**
    * Recibir `pokemonId` como prop.
    * Obtener detalles de la ![PokeAPI](https://img.shields.io/badge/-PokeAPI-EF5350?style=flat-square).
    * Renderizar detalles.
    * Recibir `teamInfo` (estado del equipo actual) como prop.
    * Mostrar y gestionar el botón "Añadir al Equipo Actual".
    * Notificar al Shell (vía prop callback `onAddToTeam`) para añadir el Pokémon.
    * Exponer `PokemonDetailView` vía Module Federation.
* **Puerto Docker (Ej):** `3002`

---

### 4. 🧩 `poke-mfe-team` (Microfrontend: Constructor Equipo / "Carrito")

* **Rol:** Gestionar la creación del equipo Pokémon *actual*.
* **Responsabilidades:**
    * Recibir y mostrar `currentTeam` como prop (leído de Redux por el Shell).
    * Permitir eliminar Pokémon del `currentTeam` (notificando al Shell).
    * Gestionar el `teamName` (estado local del MFE).
    * Validar y notificar al Shell (vía prop callback `onSaveTeam`) para guardar el equipo.
    * Exponer `CurrentTeamBuilder` vía Module Federation.
* **Puerto Docker (Ej):** `3003`

---

### 5. 🧩 `poke-mfe-saved-teams` (Microfrontend: Lista Equipos Guardados)

* **Rol:** Mostrar los equipos guardados por el usuario.
* **Responsabilidades:**
    * Recibir `savedTeams` como prop (leído de Redux / `localStorage` por el Shell).
    * Renderizar la lista de equipos guardados.
    * (Opcional) Permitir eliminar equipos guardados (notificando al Shell).
    * Exponer `SavedTeamsList` vía Module Federation.
* **Puerto Docker (Ej):** `3004`

---

## Flujo de Datos y Estado (Resumen con Redux ![Redux](https://img.shields.io/badge/-Redux-764ABC?style=flat-square&logo=redux&logoColor=white))

1.  **Interacción en MFE:** El usuario interactúa (ej. clic en PokedexList).
2.  **Callback al Shell:** El MFE invoca una función callback pasada como prop.
3.  **Dispatch en Shell:** El handler del callback en el Shell despacha una acción a la store Redux.
4.  **Actualización en Redux:** El reducer correspondiente actualiza el estado global en la store del Shell.
5.  **Persistencia (si aplica):** Un `useEffect` en el Shell detecta cambios en `savedTeams` y lo guarda en `localStorage` 💾.
6.  **Selección en Shell:** El Shell (y potencialmente otros MFEs) usa `useAppSelector` para leer el estado actualizado.
7.  **Props a MFEs:** El Shell pasa los datos necesarios del estado actualizado como props a los MFEs hijos para que se re-rendericen.

## Dockerización ![Docker](https://img.shields.io/badge/-Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

* Cada servicio (Shell + 4 MFEs) tiene su `Dockerfile`.
* `docker-compose.yml` orquesta el inicio y la red de todos los contenedores.
* La comunicación entre el Shell y los MFEs para Module Federation se realiza a través de la red Docker, usando los nombres de servicio definidos en `docker-compose.yml`.

---