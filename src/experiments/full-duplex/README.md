# Full-duplex / enlace integrado

`main_experimental.m` implementa o fluxo gNB→UE no downlink e UE→gNB no uplink, incluindo ACK e métricas de BER/EVM.

## Como rodar

Abra `main_experimental.m`. Primeiro use:

```matlab
pCHDownlink = 'software';
pCHUplink = 'software';
```

Para a bancada integrada, a configuração prevista é `pCHDownlink = 'adalm2000'` para VLC e `pCHUplink = 'adalm-pluto'` para RF.

## Parametrização

- Controle: `pMaxIt`, `pMaxRetry`, `pPlotFigures`.
- gNB/UE: `pSCS`, `pSizeGrid`, `pBW`, `pCyclicPrefix`, `pNCellID`, `pLayers` e `DCIModulation`.
- Simulação: `pSimArg.pSNRdB`, `pSimArg.pFrequencyOffset`, `pSimArg.pPhaseOffset`.
- PLUTO: `pPlutoArg.pGain`, `pPlutoArg.pCenterFrequency`, `pRadioIDTx`, `pRadioIDRx`.
- ADALM2000: `pM2K.pVLCCenterFrequency`, `pM2K.pDACSampleRate`, `pM2K.pADCSampleRate`.

Antes de usar hardware, atualize os IPs/IDs e confirme as conexões RF e analógicas.
