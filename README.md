# poc-oficina-virtual · Fase 4

Implementació de la **Fase 4**: monitors interactius, streams de sala i bloqueig de teclat (Vue 3 + Phaser 4 + Vite).

Quan en George entra en una zona del mapa de Tiled que té `streamUrl`, apareix un **monitor en miniatura** al panell lateral. En clicar-lo s'obre un **modal gran** i Phaser **bloqueja el teclat** perquè en George no camini.

## Execució

```bash
npm install      # instal·la les dependències
npm run dev      # servidor de desenvolupament → http://localhost:5173
npm run build    # versió de producció a dist/
npm run preview  # serveix dist/ en local
```

Es mou amb **WASD** o amb les **fletxes**. Els vídeos de prova són MP4 públics, així que cal connexió a Internet.

## Estructura

```text
poc-oficina-virtual/
├── public/assets/
│   ├── maps/
│   │   ├── oficina.tmx          <-- [PAS 0] Fitxer editable de Tiled (amb 'streamUrl')
│   │   └── oficina.json         <-- [PAS 0] Exportat en JSON per a Phaser
│   ├── sprites/
│   │   └── george.png           <-- Spritesheet 48x48 (4 columnes x 4 files)
│   └── tilesets/
│       └── pixel_office.png     <-- Tileset de l'oficina
└── src/
    ├── game/
    │   └── OfficeScene.js       <-- [PAS 1 i 4] Escena Phaser (zones + bloquejarControls)
    ├── components/
    │   └── PhaserCanvas.vue     <-- [PAS 2 i 5] Pont/Proxy (defineEmits + defineExpose)
    └── App.vue                  <-- [PAS 3 i 6] Interfície Vue (Monitor mini + Modal gran)
```

### Recursos gràfics i mapa

El tutorial parteix del projecte de la Fase 3 del professor, però no n'inclou els recursos. Aquí s'han creat de nou amb el format que espera el codi:

| Fitxer | Format |
| --- | --- |
| `george.png` | 192 × 192 px, fotogrames de 48 × 48. **Columnes** = direcció (0 avall, 1 esquerra, 2 amunt, 3 dreta). **Files** = pas de l'animació. Encaixa amb `crearAnimacions()` (`[0, 4, 8, 12]` = caminar avall) i amb `frameQuiets`. |
| `pixel_office.png` | Tiles de 32 px sense marge ni espai. Els tiles sòlids tenen la propietat `collides = true`. |
| `oficina.tmx` / `oficina.json` | 25 × 19 tiles (800 × 608 px; el joc en mostra 800 × 600). Capes `Terra`, `Parets` (parets i mobles) i `Objectes`. Tileset incrustat amb el nom `pixel_office`. |

Objectes de la capa `Objectes`:

| Objecte | `tipus` | `nomSala` | `titolMonitor` | `streamUrl` |
| --- | --- | --- | --- | --- |
| Rectangle | `zona` | Sala de Reunions | *(no en té → s'usa `nomSala`)* | `https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4` |
| Rectangle | `zona` | Zona Cafè | Càmera Descans · Canal 02 | `https://media.w3.org/2010/05/sintel/trailer.mp4` |
| Punt | `spawn` | | | |

L'`Spawn` és l'últim objecte de la capa. Phaser dibuixa els objectes per ordre de creació, i així els rectangles verds de les zones queden per sota d'en George.

## Cicle 1 · Monitor en miniatura (Phaser → Vue)

1. **Pas 0 · Tiled**: cada zona té `tipus`, `nomSala` i `streamUrl`. Després de canviar-los, cal exportar de nou `oficina.json`.
2. **Pas 1 · `OfficeScene.js`**: `create()` guarda `streamUrl` a `this.zones`. `comprovarZones()` emet `canvi-zona` amb `{ nom, titolMonitor, streamUrl }`, o `null` en sortir al passadís.
3. **Pas 2 · `PhaserCanvas.vue`**: escolta `game.events.on('canvi-zona')` i ho reenvia a `App.vue` amb `emit('canviZona', dadaZona)`.
4. **Pas 3 · `App.vue`**: `<div v-if="zonaActiva?.streamUrl" class="monitor-mini">` amb un `<video autoplay loop playsinline>`.

**Punt de Control 1**

- [x] Al passadís el panell diu «Estàs al passadís», i no hi ha ni vídeo ni so.
- [x] En entrar a la Sala de Reunions apareix el monitor amb el vídeo i el so.
- [x] A la Zona Cafè el monitor canvia de canal.
- [x] En sortir al passadís, el monitor desapareix i el so s'atura.

## Cicle 2 · Modal gran i bloqueig de teclat (Vue → Phaser)

5. **Pas 4 · `OfficeScene.js`**: nou mètode `bloquejarControls(bloquejat)`. Desactiva `this.input.keyboard` i frena en George amb `setVelocity(0, 0)`.
6. **Pas 5 · `PhaserCanvas.vue`**: `defineExpose({ bloquejarControls })`.
7. **Pas 6 · `App.vue`**: `refCanvas`, `modalObert`, `obrirModalGran()` i `tancarModalGran()`, i el modal amb `@click.self` per tancar-lo clicant fora.

**Punt de Control 2**

- [x] Clicar el monitor petit obre el modal gran amb els controls del vídeo.
- [x] Amb el modal obert, WASD i les fletxes no mouen en George.
- [x] «Tancar [X]», o un clic fora del requadre, tanca el modal. El monitor petit continua i en George torna a caminar.

## Ampliació · Segon monitor amb múltiples atributs

La Zona Cafè té un quart atribut, `titolMonitor`. `OfficeScene.js` el llegeix amb `props.titolMonitor || props.nomSala` i l'envia dins de `canvi-zona`. El monitor mostra `{{ zonaActiva.titolMonitor }}`. A la Sala de Reunions, que no té `titolMonitor`, es veu el `nomSala`.

## Millores respecte al codi del tutorial

El codi segueix les diapositives línia a línia. Les úniques diferències estan marcades amb `[MILLORA]` als comentaris. Totes les proves oficials passaven també sense aquests canvis, però en provar casos límit al navegador van aparèixer tres errors:

| Problema (codi original) | Causa | Canvi |
| --- | --- | --- |
| Si tenies **D premuda** en clicar el monitor, en George **continuava caminant** amb el modal obert. En tancar-lo, caminava tot sol. | Amb `keyboard.enabled = false`, Phaser ja no processa el `keyup`: la tecla es queda amb `isDown = true` i `update()` torna a posar velocitat al fotograma següent. El `setVelocity(0, 0)` no hi pot fer res. | `this.input.keyboard.resetKeys()` en bloquejar (`OfficeScene.js`). |
| En un camp de text dins del modal (el «xat» de la diapositiva), en escriure «wasd hola» només apareixia **«hol»**. | Phaser continua fent `preventDefault()` de les tecles capturades (W, A, S, D, fletxes, espai) encara que el teclat estigui desactivat. | `disableGlobalCapture()` en bloquejar i `enableGlobalCapture()` en desbloquejar (`OfficeScene.js`). |
| Amb el modal obert sonaven **dos vídeos alhora** (el petit i el gran), desfasats. | El monitor petit no s'atura quan s'obre el modal. | `:muted="modalObert"` al `<video>` del monitor petit (`App.vue`). |

També s'ha substituït el `style.css` de la plantilla de Vite per un reset mínim. La plantilla centrava `#app` i desquadrava el `layout-oficina`.

## Nota sobre l'autoplay amb so

Els navegadors bloquegen l'autoplay amb so fins que l'usuari interactua amb la pàgina. En aquest projecte, en George només es mou amb el teclat, i prémer una tecla ja compta com a interacció. Per això, quan arriba a una sala el vídeo es pot reproduir amb so.

## Següent pas: SimplePeer

El `<video>` del monitor és el punt d'injecció. Al projecte final només caldrà canviar `:src="zonaActiva.streamUrl"` per `:srcObject="remoteStream"`, on `remoteStream` és el que arriba amb `peer.on('stream', ...)`.

## Lliurament

Comprimir la carpeta en un `.zip` **sense `node_modules`** (ni `dist`), o lliurar l'enllaç al repositori Git.
