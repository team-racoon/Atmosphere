# Guía de Compilación con Docker Compose

Esta guía explica cómo compilar este fork de **Atmosphère** de forma aislada, reproducible y eficiente utilizando **Docker Compose** junto con el `Dockerfile` y `Makefile` del proyecto.

---

## 1. Requisitos Previos

1. **Docker y Docker Compose** instalados y con permisos de ejecución sin sudo (usuario en el grupo `docker`).
2. **Llaves de Nintendo Switch (`prod.keys`)**:
   - Deben estar ubicadas en tu máquina local dentro de `~/.switch/prod.keys`.
   - El archivo `compose.yaml` monta automáticamente tu carpeta `~/.switch` en modo solo lectura (`:ro`) hacia `/home/atmosphere/.switch` en el contenedor.

---

## 2. Modos de Uso y Comandos Principales

### A. Compilación Rápida para Consola (Recomendada)
Compila todo el sistema y genera el archivo ZIP listo para descomprimir en la tarjeta SD (`atmosphere-<version>.zip`), pero **omite** el empaquetado del archivo pesado de símbolos de depuración.

```bash
docker compose run --rm dist-no-debug
```

> **¿Dónde quedan los archivos?**  
> En la carpeta raíz y en `out/nintendo_nx_arm64_armv8a/release/` encontrarás `atmosphere-<version>.zip`, `fusee.bin` y los binarios correspondientes.

---

### B. Compilación Release Completa (con símbolos de depuración)
Genera el paquete para la consola y **además** el archivo `atmosphere-<version>-debug.zip` con todos los ejecutables `.elf` para depuración con GDB:

```bash
docker compose up
# O de manera equivalente:
docker compose run --rm build
```

---

### C. Limpiar y Recompilar desde Cero
Si deseas asegurarte de que ningún archivo objeto anterior interfiera y reconstruir todo el árbol de código:

```bash
docker compose run --rm rebuild
```

---

### D. Limpieza de Artefactos

* **Limpiar compilación actual (Release)**:
  ```bash
  docker compose run --rm clean
  ```

* **Limpiar todas las configuraciones (Release, Debug, Audit)**:
  ```bash
  docker compose run --rm clean-all
  ```

> [!TIP]
> **No es necesario hacer `clean` en cada compilación.** Make soporta compilación incremental: si solo modificaste uno o dos archivos de código, `make` solo recompilará lo que cambió en segundos.

---

### E. Otros Perfiles de Compilación

* **Compilar con flags de depuración activadas (`AMS_BUILD_FOR_DEBUGGING`)**:
  ```bash
  docker compose run --rm debug
  ```

* **Compilar con flags de auditoría (`AMS_BUILD_FOR_AUDITING`)**:
  ```bash
  docker compose run --rm audit
  ```

---

## 3. Control de Rendimiento y Multihilo (CPU Cores)

El archivo `compose.yaml` tiene configurada la variable `MAKEFLAGS=-j${JOBS:-6}`.

1. **Uso por defecto**:
   - Por defecto utiliza **6 hilos** concurrentes de tu procesador (50% de uso en CPUs de 12 hilos), permitiendo una compilación rápida sin congelar la computadora.

2. **Personalizar la cantidad de núcleos al vuelo**:
   Si deseas cambiar temporalmente la cantidad de núcleos para una ejecución, antepón la variable `JOBS`:
   ```bash
   # Modo ultra liviano (3 núcleos):
   JOBS=3 docker compose run --rm dist-no-debug

   # Modo máxima velocidad (ej. 10 núcleos):
   JOBS=10 docker compose run --rm dist-no-debug
   ```

---

## 4. Terminal Interactiva para Depuración

Si necesitas inspeccionar el entorno, revisar librerías de devkitPro o compilar únicamente un submódulo específico a mano:

```bash
docker compose run --rm build bash
```

Una vez dentro del contenedor puedes navegar a cualquier carpeta y compilar puntualmente:
```bash
# Ejemplo: compilar solo ams_mitm
cd stratosphere/ams_mitm
make

# Salir del contenedor (se destruye automáticamente sin dejar residuos)
exit
```

---

## 5. Resumen Rápido de Equivalencias

| Acción | Comando Docker Compose | Comando Makefile equivalente |
| :--- | :--- | :--- |
| **Release SD (sin debug zip)** | `docker compose run --rm dist-no-debug` | `make dist-no-debug-nx_release` |
| **Release completo** | `docker compose up` | `make` o `make nx_release` |
| **Clean + Build** | `docker compose run --rm rebuild` | `make clean && make` |
| **Limpiar release** | `docker compose run --rm clean` | `make clean` |
| **Limpiar todo** | `docker compose run --rm clean-all` | `make clean-all` |
| **Debug build** | `docker compose run --rm debug` | `make nx_debug` |
| **Terminal interactiva** | `docker compose run --rm build bash` | N/A |
