# 5G NR OFDM Test

Implementação e bancada experimental em MATLAB para geração, transmissão e recepção de formas de onda OFDM 5G NR. O projeto cobre enlaces de rádio frequência (RF) com ADALM-PLUTO e enlaces de comunicação por luz visível (VLC) com ADALM2000, além de uma simulação exclusivamente em software.

## Estrutura

- `newradio/`: geração, mapeamento e decodificação dos canais físicos 5G NR (SSB, PDCCH, PDSCH, PUCCH, PUSCH e SRS).
- `adalm-pluto/`: inicialização e reinicialização dos rádios ADALM-PLUTO.
- `adalm2000/`: inicialização e configuração dos instrumentos ADALM2000 via libm2k.
- `filters/`: filtros e configurações usadas pelo ADALM-PLUTO.
- `etc/`: utilitários de métricas e de visualização.
- `experiments/`: pontos de entrada dos experimentos descritos abaixo.

## Requisitos

- MATLAB com **5G Toolbox**, **Communications Toolbox** e **Signal Processing Toolbox**.
- Para RF: **Communications Toolbox Support Package for ADALM-PLUTO Radio** e um ou dois ADALM-PLUTO configurados por USB ou IP.
- Para VLC: suporte MATLAB para **libm2k** e dois ADALM2000 (um para transmissão e outro para recepção), com o enlace óptico conectado aos canais analógicos configurados no código.
- Para execução apenas em software, os dispositivos físicos não são necessários.

Abra o MATLAB a partir desta pasta (`src`) ou execute `cd` para ela. Cada script de experimento adiciona os caminhos auxiliares relativos de que precisa.

## Elementos de canal

Os scripts usam a variável de canal indicada abaixo. Ajuste-a antes de executar o experimento.

| Valor | Elemento | Uso |
| --- | --- | --- |
| `software` | Simulação MATLAB | Aplica canal AWGN, atraso aleatório e desvios de fase/frequência; é o ponto de partida recomendado. |
| `adalm-pluto` | ADALM-PLUTO | Enlace RF baseband via `sdrtx`/`sdrrx`. Configure IDs/IPs, frequência central, ganho e filtros no script. |
| `adalm2000` | ADALM2000 | Enlace VLC em banda passante. O sinal OFDM é reamostrado, modulado em QAM analógica, transmitido pelo DAC e capturado pelo ADC via libm2k. |
| `b2b` | Loopback em software | Disponível nos scripts half-duplex para validar a cadeia de conversão para banda passante sem hardware. |

Nos experimentos integrados, o **downlink** pode usar `software` ou `adalm2000`, enquanto o **uplink** pode usar `software` ou `adalm-pluto`.

## Principais experimentos

### Half-duplex

Os scripts estão em `experiments/half-duplex/`.

- `main_downlink.m`: gera SSB, PDCCH e PDSCH; transmite, sincroniza e decodifica o downlink.
- `main_uplink.m`: gera SRS, PUCCH e PUSCH; transmite, sincroniza e decodifica o uplink.

Para rodar, abra o script desejado, ajuste `pChannel` para `software`, `adalm-pluto`, `adalm2000` ou `b2b`, revise os parâmetros de canal e execute-o. Para a primeira execução, use `software` e depois altere `pSNRdB`, `pFrequencyOffset` e `pPhaseOffset` conforme necessário.

### Full-duplex / enlace integrado

Em `experiments/full-duplex/main_experimental.m`, o gNB envia o downlink e, após a recepção, a UE retorna ACK e uplink. Configure `pCHDownlink` e `pCHUplink` no início do arquivo e execute o script.

Configuração típica de bancada:

- downlink: `pCHDownlink = 'adalm2000'` para VLC;
- uplink: `pCHUplink = 'adalm-pluto'` para RF.

Antes da execução com hardware, ajuste os IPs dos PLUTOs (`pRadioIDTx`, `pRadioIDRx`) e dos ADALM2000 (`pM2KTxIP`, `pM2KRxIP`), a frequência central RF, os ganhos e as taxas de amostragem. Verifique também as conexões físicas e a calibração dos ADALM2000.

### Resposta em frequência

Em `experiments/freqResponse/`:

- `mainPluto.m` e `mainPLuto2Antenas.m`: varredura de resposta em frequência com ADALM-PLUTO;
- `mainM2k.m`: varredura de resposta em frequência do enlace ADALM2000/VLC;
- `plutoFreqTestCorrigido*.m`: variantes de medição e correção para o PLUTO;
- `ofdm3D.m`: visualização do sinal OFDM.

Escolha o script que corresponde à bancada, revise frequência inicial/final, passo, taxa de amostragem, identificadores do dispositivo e execute-o. Esses experimentos podem salvar arquivos de medição no próprio diretório; valide o local de saída antes de rodar.

### VLC Test Bed

Os arquivos `experiments/VLCTestBed/main_start.m` e `main_experimental.m` automatizam varreduras de modulação/codificação, corrente de polarização e distância do enlace VLC. Em `main_start.m`, configure `DCItypes`, `Idc`, `distance`, `downlinkCH` e `uplinkCH`, então execute `main_start.m`.

Os resultados originalmente produzidos em `experiments/VLCTestBed/Results2/` são intencionalmente ignorados pelo Git para manter o repositório focado no código-fonte.

## Publicação e dados ignorados

O arquivo `.gitignore` na raiz exclui `UNUSED Code/`, o arquivo ZIP, a imagem de condição de teste e `experiments/VLCTestBed/Results2/`. Ele também exclui figuras MATLAB (`.fig`) e a forma de onda gerada `filters/waveform_pb.mat`, que são artefatos binários de grande porte; o código-fonte e as configurações de filtro necessárias permanecem versionados.

## Observações de segurança operacional

Comece sempre em `software` para conferir a configuração. Ao migrar para hardware, reduza os ganhos iniciais, confira frequência e taxas de amostragem compatíveis e confirme que os canais analógicos/RF estão conectados à bancada correta antes de transmitir.
