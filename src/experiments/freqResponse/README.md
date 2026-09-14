# Resposta em frequência

Mede e visualiza a resposta em frequência das bancadas RF e VLC.

- `mainPluto.m`: varredura com ADALM-PLUTO.
- `mainPLuto2Antenas.m`: varredura PLUTO com duas antenas.
- `mainM2k.m`: varredura ADALM2000/VLC.
- `plutoFreqTest*.m`: variantes de teste e correção para PLUTO.
- `ofdm3D.m`: visualização OFDM.

## Como rodar

Abra o script correspondente à bancada. Ajuste frequência inicial/final, `fpass` (passo), taxa de amostragem e identificação dos dispositivos antes de executar.

Para o PLUTO, confira frequência central, ganho e IDs/IPs. Para o ADALM2000, confira `pM2K.pDACSampleRate`, `pM2K.pADCSampleRate`, faixas analógicas e a calibração DAC/ADC.
