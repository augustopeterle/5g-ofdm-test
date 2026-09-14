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

## Parametrização

Os parâmetros ficam no início de cada script de experimento. A sequência recomendada é configurar primeiro o canal (`software`), validar a forma de onda e só então alterar os parâmetros de hardware.

| Grupo | Parâmetros principais | Onde configurar | Finalidade |
| --- | --- | --- | --- |
| Forma de onda NR | `pSCS`, `pSizeGrid`, `pLayers`, `pCyclicPrefix`, `pNCellID` | scripts half-duplex e estruturas `gNBConfig`/`UEConfig` | Espaçamento entre subportadoras (kHz), quantidade de RBs, camadas, prefixo cíclico e identidade da célula. |
| Modulação e codificação | `pDigitalModulation`, `pModulationLevel`, `pCodeRate` ou `DCIModulation` | scripts half-duplex; `main_experimental.m` | Define QPSK/16QAM/64QAM e a taxa de código. Nos experimentos integrados, `DCIModulation` é convertido pela função `getMCSLookupTable`. |
| Canal simulado | `pChannel`, `pSNRdB`, `pFrequencyOffset`, `pPhaseOffset` | `main_downlink.m` e `main_uplink.m` | Seleciona o elemento de canal e controla ruído AWGN, erro de frequência e erro de fase. |
| RF / ADALM-PLUTO | `pGain`, `pCenterFrequency`, `pRadioIDTx`, `pRadioIDRx` | scripts half-duplex e full-duplex | Define ganho de transmissão, frequência RF e os identificadores USB/IP de TX e RX. |
| VLC / ADALM2000 | `pVLCCenterFrequency`, `pDACSampleRate`, `pADCSampleRate` | scripts half-duplex e full-duplex | Define a portadora elétrica do enlace VLC e as taxas de DAC/ADC. |
| Execução integrada | `pCHDownlink`, `pCHUplink`, `pMaxIt`, `pMaxRetry` | `experiments/full-duplex/main_experimental.m` | Escolhe o elemento de canal por sentido e controla o número de iterações e retransmissões. |
| Varredura VLC | `DCItypes`, `Idc`, `distance`, `downlinkCH`, `uplinkCH` | `experiments/VLCTestBed/main_start.m` | Define MCSs testados, corrente de polarização, distância e elementos de canal da bancada. |

### Valores de partida

Para validar a cadeia sem hardware, use `pChannel = 'software'` nos scripts half-duplex. Nos experimentos full-duplex e VLC Test Bed, use `pCHDownlink = 'software'` e `pCHUplink = 'software'`. Um conjunto frequente de parâmetros no código é SCS de 15 kHz, grade de 25 RBs, uma camada, prefixo normal e `pNCellID = 102`.

### Migração para hardware

Para RF, altere o canal para `adalm-pluto`, ajuste os IDs/IPs para os dispositivos conectados, confirme `pCenterFrequency` e comece com ganho baixo (`pGain`). Para VLC, selecione `adalm2000`, confira a ligação entre DAC, driver óptico e receptor/ADC, então ajuste a frequência VLC e as taxas de amostragem. O script `initM2KSDR.m` realiza a abertura e calibração dos ADALM2000; `initPlutoSDR.m` configura transmissão e recepção com os filtros do PLUTO.

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

O arquivo `.gitignore` na raiz exclui `UNUSED Code/`, o arquivo ZIP, a imagem de condição de teste e `experiments/VLCTestBed/Results2/`. Além disso, arquivos de resultado, formas de onda, configurações binárias, figuras, imagens, PDFs, textos auxiliares e autosaves do MATLAB não são versionados. O repositório contém somente os scripts MATLAB, este README e a configuração de Git.

## Observações de segurança operacional

Comece sempre em `software` para conferir a configuração. Ao migrar para hardware, reduza os ganhos iniciais, confira frequência e taxas de amostragem compatíveis e confirme que os canais analógicos/RF estão conectados à bancada correta antes de transmitir.
