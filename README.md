# chess-motor-web

Suite de estudio y análisis de ajedrez: un solo programa para jugar, analizar, explorar aperturas y enfrentar motores entre sí. Corre en el navegador y como aplicación de escritorio (Tauri), con motores UCI locales — sin depender de servicios externos.

> Nació como interfaz para **KallpaModulo**, un motor propio en C++, y creció hasta cubrir el flujo completo de estudio al estilo de En Croissant o ChessBase.

## Qué hace

| Pestaña      | Para qué sirve                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------------ |
| **Jugar**    | Partida contra un motor con nivel y tiempo configurables.                                        |
| **Análisis** | Análisis infinito con barra de evaluación, gráfica de la partida y árbol de variantes navegable. |
| **Match**    | Enfrenta dos motores en una serie de partidas con control de tiempo — para comparar fuerza.      |
| **Motores**  | Registro de motores UCI disponibles, con detección de binarios y opciones por motor.             |
| **Bases**    | Explorador de aperturas y consulta de tablas de finales (tablebase).                             |
| **Estudios** | Partidas y posiciones guardadas para repasar.                                                    |
| **Archivos** | Importación y manejo de PGN.                                                                     |

## Arquitectura

```
Next.js 14 (React + TypeScript + Tailwind)
        │  WebSocket
        ▼
server.js ── engine-bridge ──► proceso UCI (stdin/stdout)
                   │
                   └─ stockfish-bridge  (UCI estándar)
                      match.js          (motor vs motor)
                      mega.js           (base de partidas)
```

El frontend nunca habla con los motores directamente: manda un `id` de motor por WebSocket y el servidor resuelve la ruta del binario. Así el mismo código sirve en web y en escritorio, y las rutas absolutas no salen del servidor.

`src-tauri/` empaqueta todo como app nativa con Tauri 2, con Stockfish incluido como _sidecar_.

## Motores soportados

Cualquier motor que hable **UCI**. Vienen registrados:

- **Stockfish** — por defecto
- **Leela Chess Zero (lc0)** — red neuronal
- **Ethereal**
- **KallpaModulo** — motor propio en C++, con protocolo JSON y modo UCI
- **KallpaModulo HCE / ML** — los dos evaluadores del motor propio (heurístico vs. modelo tabular), registrados por separado para poder compararlos en el **Match**

Agregar uno más es añadir una entrada en `server/engines.js`.

## Puesta en marcha

Requisitos: Node 18+ y al menos un binario UCI (por ejemplo [Stockfish](https://stockfishchess.org/download/)).

```bash
git clone https://github.com/JuanMCanchala/chess-motor-web
cd chess-motor-web
npm install
cp .env.example .env        # indica la ruta a tu binario de Stockfish
npm run dev                 # http://localhost:3000
```

Variables relevantes (`.env`):

| Variable                               | Qué es                                       |
| -------------------------------------- | -------------------------------------------- |
| `ENGINE_KIND`                          | `stockfish` (por defecto) o `kallpa`         |
| `STOCKFISH_PATH`                       | ruta al binario UCI                          |
| `STOCKFISH_THREADS` / `STOCKFISH_HASH` | hilos y tabla hash; vacío = automático       |
| `ENGINE_PATH`                          | ruta a KallpaModulo, si usas el motor propio |
| `PORT`                                 | puerto del servidor (3000)                   |

### Escritorio

```bash
npm run tauri:dev      # ventana nativa
npm run tauri:build    # instalador
```

## Stack

Next.js 14 · React 18 · TypeScript · Tailwind · Zustand · chess.js · react-chessboard · `ws` · Tauri 2 (Rust)

## Contexto

Este repo es la **interfaz**. El motor en C++ y el trabajo de investigación viven aparte:

- `KallpaModulo` — motor de ajedrez clásico en C++ (alfa-beta, evaluación heurística)
- `chess-motor-tabular` — evaluador de aprendizaje automático tabular integrado en el mismo motor, para una comparación controlada HCE vs. ML (trabajo de grado, Pontificia Universidad Javeriana Cali)

## Licencia

MIT
