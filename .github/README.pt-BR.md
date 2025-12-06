[![English](https://img.shields.io/badge/lang-English-informational)](../README.md)
[![Português (BR)](https://img.shields.io/badge/idioma-Portugu%C3%AAs%20(BR)-blue)](./README.pt-BR.md)

# Configuração Armer Argent One + FlexASIO + RS_ASIO

Este repositório fornece a configuração e as instruções para usar a interface de áudio **Armer Argent One** com o **FlexASIO** (driver ASIO universal) e o **RS_ASIO** (patch do Rocksmith 2014 para habilitar ASIO).

O objetivo desta configuração é permitir o uso da interface Argent One com baixa latência no Rocksmith 2014, corrigindo erros comuns de incompatibilidade de canais.

* [RS_ASIO](https://github.com/mdias/rs_asio)
* [FlexASIO](https://github.com/dechamps/FlexASIO)
* [RSMods](https://github.com/Lovrom8/RSMods) (Necessário para o Fast Load)
* [FlexASIO GUI](https://github.com/flipswitchingmonkey/FlexASIO_GUI) (Opcional)

---

## Índice

1. [Visão Geral](#visão-geral)
2. [Pré-requisitos](#pré-requisitos)
3. [Arquivos de Configuração Inclusos](#arquivos-de-configuração-inclusos)
4. [Instruções de Instalação](#instruções-de-instalação)
5. [Testes e Ajuste de Latência](#testes-e-ajuste-de-latência)
6. [Resolução de Problemas (Troubleshooting)](#resolução-de-problemas-troubleshooting)
7. [Detalhes de Hardware da Argent One](#detalhes-de-hardware-da-argent-one)
8. [Créditos e Referências](#créditos-e-referências)

---

## Visão Geral

* **Argent One** é uma interface de áudio de 2 canais da Armer.
* **FlexASIO** faz a ponte entre hosts ASIO e o sistema de áudio do Windows (WASAPI).
* **RS_ASIO** injeta suporte ASIO no **Rocksmith 2014**.

**Por que esta configuração específica?**
A Argent One se apresenta ao Windows como um **dispositivo Estéreo de 2 canais**. O Rocksmith requer uma entrada Mono específica. Esta configuração força o FlexASIO a corresponder ao formato estéreo do Windows (evitando falhas e erros de formato) e usa o RS_ASIO para rotear o canal físico correto (Esquerdo ou Direito) para as funções do jogo.

---

## Pré-requisitos

* PC com Windows.
* Interface **Armer Argent One**, instalada e funcionando.
* **FlexASIO** instalado (versão mais recente).
* Arquivos do **RS_ASIO** copiados para a pasta do Rocksmith.
* **RSMods** instalado (Crucial para pular os vídeos de introdução e evitar travamentos de "tela branca").

---

## Arquivos de Configuração Inclusos

### 1. FlexASIO.toml
*Coloque este arquivo na sua pasta de Usuário (ex: `C:\Usuários\SEU_NOME\FlexASIO.toml`)*

**Importante:** Definimos `channels = 2` para corresponder ao Formato Padrão do Windows. Definir isso como 1 geralmente causa erros `AUDCLNT_E_UNSUPPORTED_FORMAT` com esta interface.

```toml
backend = "Windows WASAPI"
bufferSizeSamples = 512

[input]
device = "Microfone (Armer Argent)"
# Deve ser 2 para corresponder ao padrão Estéreo do Windows
channels = 2
# Defina como 'true' para menor latência (Apenas o Jogo terá som). 
# Defina como 'false' para permitir YouTube/Spotify ao fundo (Aumenta a latência).
wasapiExclusiveMode = true
wasapiAutoConvert = false

[output]
device = "Fones de ouvido (Armer Argent)"
# Deve ser 2 para corresponder ao padrão Estéreo do Windows
channels = 2
wasapiExclusiveMode = true
wasapiAutoConvert = false
````

### 2\. RS\_ASIO.ini

*Coloque este arquivo na pasta raiz do seu Rocksmith.*

Esta configuração mapeia as entradas físicas específicas da Argent One:

  * **Input 0 (Esquerda/Mic)** -\> Mapeado para o Microfone do Rocksmith.
  * **Input 1 (Direita/Inst)** -\> Mapeado para o Cabo da Guitarra do Rocksmith.

<!-- end list -->

```ini
[Config]
EnableWasapiOutputs=0
EnableWasapiInputs=0
EnableAsio=1

[Asio]
BufferSizeMode=driver
CustomBufferSize=

[Asio.Output]
Driver=FlexASIO
BaseChannel=0
EnableSoftwareEndpointVolumeControl=1
EnableSoftwareMasterVolumeControl=1

[Asio.Input.0]
; Esta é a GUITARRA (Player 1)
; Usamos Channel=1 porque a Entrada 2 (Inst) da Argent One é o canal Direito
Driver=FlexASIO
Channel=1
EnableSoftwareEndpointVolumeControl=1
EnableSoftwareMasterVolumeControl=1

[Asio.Input.Mic]
; Este é o MICROFONE VOCAL
; Usamos Channel=0 porque a Entrada 1 (Mic) da Argent One é o canal Esquerdo
Driver=FlexASIO
Channel=0
EnableSoftwareEndpointVolumeControl=1
EnableSoftwareMasterVolumeControl=1
```

-----

## Instruções de Instalação

1.  **Configurações de Som do Windows**

      * Abra o `mmsys.cpl` (Painel de Controle de Som).
      * Defina a Argent One como dispositivo Padrão de Reprodução e Gravação.
      * **Crucial:** Em Propriedades \> Avançado, certifique-se de que ambos estejam definidos para **2 canais, 16 bits (ou 24 bits), 48000 Hz**.

2.  **Instalar FlexASIO**

      * Baixe e instale a versão mais recente.

3.  **Instalar RS\_ASIO**

      * Copie `RS_ASIO.dll`, `RS_ASIO.ini` (conteúdo acima) e `avrt.dll` para a pasta do Rocksmith.

4.  **Configurar FlexASIO**

      * Crie o arquivo `FlexASIO.toml` com o conteúdo acima na sua pasta de usuário (`C:\Usuários\SEU_NOME\`).

5.  **Instalar RSMods (Corrigir Inicialização Lenta)**

      * O Rocksmith possui vídeos de introdução que não podem ser pulados nativamente. Com drivers ASIO personalizados, isso muitas vezes parece que o jogo travou (tela branca por \~20 segundos).
      * Instale o **RSMods**, vá para a aba "Set and Forget" e habilite o **Fast Load**. Isso remove a introdução e torna a inicialização instantânea.

6.  **Iniciar e Jogar**

      * Abra o Rocksmith. O jogo deve iniciar instantaneamente e o áudio deve funcionar tanto para a Guitarra quanto para o Microfone.

-----

## Testes e Ajuste de Latência

  * **Buffer Padrão:** `512` amostras (Equilíbrio seguro).
  * **Baixa Latência:** Tente alterar `bufferSizeSamples` no arquivo TOML para `256` ou `192`.
      * Se o áudio estalar/pipocar, aumente o número.
      * Se o áudio estiver atrasado, diminua o número.
  * **Modo Exclusivo:** A configuração fornecida usa `wasapiExclusiveMode = true` para melhor desempenho. Se você precisar assistir ao YouTube ou usar o Discord enquanto toca, altere para `false` no `FlexASIO.toml` (isso pode aumentar a latência).

-----

## Resolução de Problemas (Troubleshooting)

| Sintoma | Causa Provável | Solução Sugerida |
| :--- | :--- | :--- |
| **Jogo trava em tela branca** | Vídeos de intro rodando invisíveis | Instale o **RSMods** e ative o "Fast Load". |
| **Erro: Unsupported Format** | Incompatibilidade de canais | Garanta `channels = 2` no TOML e formato do Windows em 48kHz Estéreo. |
| **Sem som na Guitarra** | Canal de entrada errado | Verifique o `RS_ASIO.ini`. A entrada Inst da Argent One geralmente é o Canal 1 (Direita). |
| **Áudio estalando/pipocando** | Buffer muito baixo | Aumente `bufferSizeSamples` para 512 ou 1024. |
| **Vídeo do YouTube pausa** | Conflito de Modo Exclusivo | Defina `wasapiExclusiveMode = false` no TOML. |

-----

## Detalhes de Hardware da Argent One

  * [Página do Produto](https://armer.com.br/produtos/interface-de-audio-usb-armer-argent-one/)
  * [Manual Oficial (PDF)](https://drive.google.com/file/d/15iVYalthiWtdguu4y76YLcfhcIaOtVLd/view?usp=sharing)

### Principais Características

  * Duas entradas combo (XLR / P10 TRS).
  * \+48V Phantom Power (Canal 1).
  * Canal 2 alternável entre **Instrumento (INST)** e **Linha**.

### Notas Importantes de Comportamento

**A) Mic (Ch1) - Canal Esquerdo**

  * Tipo de conector: **XLR de 3 pinos**.
  * \+48 V Phantom Power disponível.
  * No Mix Estéreo do Windows, este aparece como o lado **Esquerdo** (Canal 0 no ASIO).

**B) Instrumento (Ch2) - Canal Direito**

  * Tipo de conector: **P10 TS (Cabo de Guitarra)**.
  * **Ative o botão INST** para Guitarras/Baixo (Alta Impedância/Hi-Z).
  * No Mix Estéreo do Windows, este aparece como o lado **Direito** (Canal 1 no ASIO).

**C) Monitoramento (Direct Monitor)**

  * **Direct Monitor ON:** Você ouve as entradas diretamente (Latência Zero). Bom para checar o sinal, mas você pode ouvir o som "seco" da guitarra misturado com o amplificador do jogo.
  * **Direct Monitor OFF:** Você ouve apenas o áudio processado do PC (Jogo/DAW). Recomendado para o Rocksmith.

-----

## Créditos e Referências

  * **RS\_ASIO** por mdias
  * **FlexASIO** por dechamps
  * **RSMods** por Lovrom8
  * **Armer Argent One** Informações de hardware baseadas no manual oficial.
