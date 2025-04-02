# Proyecto PokeApp - Arquitectura de Microfrontends

Este proyecto es una aplicación que utiliza una arquitectura de microfrontends (MFEs) para gestionar la visualización y manipulación de datos de Pokémon. La aplicación está diseñada para aprender y practicar la integración de microfrontends con **Module Federation** de Webpack y **Docker** para contenerización.

## Descripción General de la Arquitectura

La aplicación se estructura en tres microfrontends principales, cada uno responsable de una parte de la funcionalidad y con comunicación específica entre ellos a través de eventos y propiedades (props). El contenedor principal, **poke-app-shell**, orquesta los diferentes microfrontends y gestiona el estado compartido.

### Estructura del Proyecto

- **poke-app-shell** (Contenedor/Shell)
- **poke-mfe-pokedex** (Microfrontend 1: Pokedex)
- **poke-mfe-detail** (Microfrontend 2: PokemonDetail)
- **poke-mfe-team** (Microfrontend 3: PokemonTeam)

## 1. poke-app-shell (Contenedor/Shell)

### Responsabilidad
- Orquestar la aplicación cargando y gestionando los microfrontends (MFEs).
- Renderizar el layout principal (dividido en dos columnas: una para la Pokedex y otra para el detalle de un Pokémon).
- Gestionar el estado compartido entre los MFEs:
  - **selectedPokemonId**: ID del Pokémon seleccionado en la Pokedex.
  - **pokemonTeam**: Lista de los Pokémon añadidos al equipo (máximo 6).

### Flujo de Datos
- **Estado compartido**:
  - Cuando un usuario selecciona un Pokémon en la Pokedex, se actualiza `selectedPokemonId`.
  - El valor de `selectedPokemonId` se pasa como prop al microfrontend **poke-mfe-detail**.
  - Si un Pokémon es añadido al equipo desde **poke-mfe-detail**, se actualiza el estado `pokemonTeam`.
  - El estado `pokemonTeam` se pasa como prop al microfrontend **poke-mfe-team**.

### Comunicación con los Microfrontends
- **poke-mfe-pokedex**: Recibe un evento de selección de Pokémon y actualiza `selectedPokemonId`.
- **poke-mfe-detail**: Actualiza el equipo de Pokémon a través de un botón "Añadir al equipo" y envía el nuevo estado a **poke-app-shell**.
- **poke-mfe-team**: Muestra el equipo de Pokémon y permite eliminar Pokémon, comunicándose con **poke-app-shell**.

### Carga de MFEs
- Utiliza **Module Federation** para cargar dinámicamente los MFEs (Pokedex, PokemonDetail, y PokemonTeam).

### Visualización del Equipo
- El equipo de Pokémon puede mostrarse en:
  - Una ruta separada `/team`.
  - Un modal que se abre desde el **poke-app-shell**.
  - Una tercera columna si se cambia el layout.

## 2. poke-mfe-pokedex (Microfrontend 1)

### Responsabilidad
- Obtener la lista de Pokémon desde la **PokeAPI** y mostrarla, por ejemplo, en una cuadrícula de Pokémon.

### Flujo de Datos
- Cuando un usuario hace clic en un Pokémon, se invoca una función pasada como prop por **poke-app-shell** para notificar el **ID** del Pokémon seleccionado.

### Exposición
- Expone el componente principal: `<PokedexList />`.

## 3. poke-mfe-detail (Microfrontend 2)

### Responsabilidad
- Recibir un **pokemonId** como prop.
- Obtener los detalles de ese Pokémon desde la **PokeAPI** y mostrar la información correspondiente.
- Mostrar un botón **"Añadir al equipo"** que permita agregar el Pokémon al equipo.

### Flujo de Datos
- Recibe `pokemonId` y `teamInfo` (con la información de si el equipo está lleno o si el Pokémon ya está en el equipo) como props desde **poke-app-shell**.
- Al hacer clic en el botón "Añadir al equipo", se llama a una función de **poke-app-shell** para agregar el Pokémon al equipo.

### Exposición
- Expone el componente principal: `<PokemonDetailView />`.

## 4. poke-mfe-team (Microfrontend 3)

### Responsabilidad
- Recibir la lista de **pokemonTeam** como prop desde **poke-app-shell** y mostrar los Pokémon que forman parte del equipo.
- Si se permite, también podría tener la capacidad de eliminar Pokémon del equipo.

### Flujo de Datos
- Recibe `pokemonTeam` como prop.
- Si la lógica de eliminación está implementada, puede llamar a una función de **poke-app-shell** para notificar qué Pokémon debe eliminar del equipo.

### Exposición
- Expone el componente principal: `<PokemonTeamDisplay />`.

## Tecnologías y Herramientas

- **React**: Framework para construir los microfrontends.
- **Module Federation**: Para la carga dinámica de microfrontends en tiempo de ejecución.
- **Webpack**: Para la configuración de la distribución y la carga de los microfrontends.
- **Docker**: Para la contenerización de cada microfrontend y la aplicación principal.

## Contenerización con Docker

El proyecto está diseñado para ser contenerizado usando Docker. Cada microfrontend será un contenedor independiente, lo que permite la escalabilidad y la modularidad. El contenedor principal **poke-app-shell** orquestará los MFEs, mientras que los microfrontends se cargarán de manera dinámica usando **Module Federation**.

## Flujo de Trabajo y Desarrollo

1. **Desarrollar cada microfrontend** de manera independiente.
2. **Configurar Docker** para contenerizar cada microfrontend y el shell.
3. **Utilizar Module Federation** para integrar los microfrontends en el contenedor principal.
4. **Desarrollar la comunicación** entre los microfrontends a través de props y callbacks.
5. **Desplegar y ejecutar la aplicación** utilizando contenedores Docker para una experiencia aislada y escalable.

## Conclusión

Esta arquitectura de microfrontends permite construir aplicaciones modulares, escalables y fáciles de mantener. Al usar tecnologías como **Module Federation** y **Docker**, se optimiza la carga y el despliegue de los diferentes componentes de la aplicación, facilitando el desarrollo y la colaboración entre equipos. ¡Esperamos que este enfoque te ayude a aprender y mejorar tus habilidades en la construcción de aplicaciones modernas y escalables!

This is a basic proyect exploring microfrontend architecture. 
