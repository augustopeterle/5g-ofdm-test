# Half-duplex

Executa os sentidos do enlace 5G NR separadamente.

- `main_downlink.m`: SSB, PDCCH e PDSCH; sincronização e decodificação de downlink.
- `main_uplink.m`: SRS, PUCCH e PUSCH; sincronização e decodificação de uplink.
- `main_downlink_branch1.m`: variante de downlink.

## Como rodar

No MATLAB, entre nesta pasta, abra um dos scripts principais e execute-o. Para validar sem equipamentos, defina `pChannel = 'software'`.

## Parâmetros

- NR: `pSCS`, `pSizeGrid`, `pLayers`, `pCyclicPrefix`, `pNCellID`.
- Dados: `pDigitalModulation`, `pModulationLevel`, `pCodeRate`.
- Simulação: `pSNRdB`, `pFrequencyOffset`, `pPhaseOffset`.
- PLUTO: `pGain`, `pCenterFrequency`, `pRadioIDTx`, `pRadioIDRx`.
- ADALM2000: `pVLCCenterFrequency`, `pDACSampleRate`, `pADCSampleRate`.

Use `adalm-pluto` para RF, `adalm2000` para VLC e `b2b` para a validação de banda passante sem hardware.
