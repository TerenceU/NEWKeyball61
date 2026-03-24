# NEWKeyball61

Configurazione ZMK per una `Keyball61` split con due `nice_nano_v2`.

## Come funziona

Questa configurazione e' una split **BLE**, non una split cablata tra le due meta':

- `keyball61_right` e' il lato **central**.
- `keyball61_left` e' il lato **peripheral**.
- Solo il lato destro parla con il PC/telefono.
- Il lato sinistro non deve comparire come tastiera BLE autonoma e non deve essere collegato via USB per l'uso normale.

Nel repo questo e' definito qui:

- `build.yaml` costruisce firmware separati per `keyball61_left` e `keyball61_right`
- `config/boards/shields/keyball61/Kconfig.defconfig` imposta `keyball61_right` come `CONFIG_ZMK_SPLIT_ROLE_CENTRAL`

## Cosa c'era di sbagliato nel repo

Ho corretto una fonte di confusione reale nella configurazione layer:

- in `config/keyball61.keymap` mancava il layer `NUM`
- le costanti layer erano sfasate rispetto all'ordine reale dei layer
- gli overlay usavano numeri grezzi per `automouse`, `scroll` e `snipe`

Il comportamento runtime non e' stato cambiato intenzionalmente: ora i layer sono espressi con nomi coerenti (`NUM`, `SYM`, `FUN`, `MOUSE`, `SCROLL`, `SNIPE`) invece di numeri ambigui.

Sul branch `niceview` c'era anche un problema di porting del display:

- il lato destro era buildato con `nice_view` ma senza `nice_view_adapter`
- il base shield continuava a forzare `zephyr,display = &oled`
- gli overlay sinistro e destro lasciavano ancora attivo l'`ssd1306` I2C originale

Con `nice_view`, questo creava sovrapposizioni di pin con il display vecchio e poteva causare instabilita', disconnessioni o comportamenti simili a sleep del lato periferico.

## Perche' la sinistra puo' sembrare "non connessa"

Per il sintomo che hai descritto ("si accende ma non si collega alla destra"), le cause piu' probabili sono:

1. pairing interno tra `central` e `peripheral` da resettare
2. firmware flashato sulla meta' sbagliata
3. configurazione precedente salvata nei controller

Questo repo include gia' il build `settings_reset` in `build.yaml`, proprio per resettare il bonding.

## Procedura consigliata di recovery

1. Flasha `settings_reset` su **entrambi** i controller.
2. Poi flasha il firmware normale corretto su ciascuna meta':
   - `keyball61_left` sulla sinistra
   - `keyball61_right` sulla destra
3. Riavvia le due meta' quasi contemporaneamente.
4. Se avevi gia' associato la tastiera al PC/telefono, fai anche `forget device` lato host e riaccoppia il lato destro.

Questa procedura segue il troubleshooting ufficiale ZMK per le split BLE.

## Nota importante

Nel port ci sono ancora pin GPIO da ricontrollare contro lo schema hardware del PCB, perche' alcuni pin usati da matrix e I2C sembrano sovrapporsi. Non li ho cambiati in automatico per non rischiare di rompere un wiring reale del tuo PCB senza avere lo schema davanti.
