# VLC Test Bed

Automatiza experimentos do enlace 5G NR sobre VLC: downlink pelo ADALM2000 e uplink pelo ADALM-PLUTO ou por simulação.

- `main_start.m`: configura e inicia a varredura.
- `main_experimental.m`: executa uma combinação de parâmetros.
- `ofdm3D.m`: visualização OFDM.

## Como rodar

Em `main_start.m`, configure `DCItypes`, `Idc`, `distance`, `downlinkCH` e `uplinkCH`; depois execute `main_start.m`.

Para teste sem equipamentos, use `downlinkCH = 'software'` e `uplinkCH = 'software'`. Para a bancada física, use `downlinkCH = 'adalm2000'` e `uplinkCH = 'adalm-pluto'`, revise os IPs de cada equipamento e confirme a calibração do ADALM2000.

Os resultados são propositalmente ignorados pelo Git.
