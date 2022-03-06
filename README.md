# UCM2
**Alsa UCM ou UCM2**.

O "Alsa [U]se [C]ase [M]anager" descreve como configurar o mixer para determinados casos de uso (como "tocar áudio", "chamar"). Ele também descreve como modificar o estado do mixer para rotear o áudio para determinadas saídas e entradas e como controlar esses dispositivos.

- Isso basicamente cobre as mesmas coisas que os perfis fazem no Pulseaudio, exceto que os arquivos UCM são mais fáceis de escrever e também funcionam sem o Pulseaudio em execução. Se uma configuração UCM estiver presente para um cartão, o pulseaudio ignorará os perfis integrados e gerará um perfil baseado nos arquivos UCM.

### Nome do cartão
- A primeira coisa que você precisa é o nome do cartão, que é usado como diretório e nome de arquivo para o arquivo principal UCM. Você pode encontrá-lo usando o comando aplay do pacote alsa-utils
```
$ sudo apt install alsa-utils
```
```
$ aplay -l
 card 0: sun50ia64audio [sun50i-a64-audio], dispositivo 0: 1c22c00.dai-sun8i-codec-aif1 sun8i-codec-aif1-0 [1c22c00.dai-sun8i-codec- aif1 sun8i-codec-aif1-0] 
  Subdispositivos: 1/1 
  Subdispositivo #0: subdispositivo #0
```
- Neste caso, o nome que você precisa é o primeiro nome entre colchetes:sun50i-a64-audio

### O primeiro caso de uso
- A próxima etapa é criar um arquivo que contém a lista de casos de uso disponíveis para o mixer. Este arquivo será nomeado após o nome do mixer da seção acima:
```
$ vim /usr/share/alsa/ucm2/sun50i-a64-audio/sun50i-a64-audio.conf
```
- no arquivo sun50i-a64-audio.conf
```
Syntax 2

SectionUseCase."HiFi" {
	File "HiFi"
	Comment "Play high quality music"
}
```
- Para cada caso de uso haverá um `"SectionUseCase"`. O nome entre aspas logo após é o "verb", que é o nome interno do perfil. Este deve ser um dos casos de uso predefinidos do alsa. Os que estão atualmente definidos são:
* "Inactive"
* "HiFi"
* "HiFi Low Power"
* "Voice"
* "Voice Low Power"
* "Voice Call"
* "Voice Call IP"
* "FM Analog Radio"
* "FM Digital Radio"

- O perfil mais importante a ser criado é o HiFi, que é o principal perfil de reprodução de áudio da placa. Para os outros perfis o uso real deles não é muito bem definido pela Alsa.

- A primeira configuração para o perfil é a linha `File`. Isso define o nome do arquivo que contém a configuração exata do mixer para esse caso de uso.

- A segunda configuração neste exemplo é o `Comment`. Isso é usado como o nome de exibição na ferramenta alsaucm e no Pulseaudio. Isso pode ser definido livremente.

### A definição de caso de uso
A parte mais complicada é o conteúdo real do arquivo `HIFI` que precisa ser criado. Este arquivo é basicamente uma coleção de scripts para habilitar/desabilitar o mixer e comandos para passar de uma entrada/saída para outra.

- Este é um exemplo mínimo de um arquivo de caso de uso.
```
$ vim /usr/share/alsa/ucm2/sun50i-a64-audio/HiFi
```
- no arquivo HiFi
```
SectionVerb {
	EnableSequence [
		cset "name='Headphone Playback Switch' off"
		cset "name='Headphone Source Playback Route' DAC"
		cset "name='Headphone Playback Volume' 50%"
	]
	DisableSequence [
	]

	Value {
		PlaybackPCM "hw:${CardId},0"
		CapturePCM "hw:${CardId},0"
	}
}
```
- O `SectionVerb` define quais comandos precisam ser executados para habilitar ou desabilitar este caso de uso. Isso é usado principalmente para trazer todo o mixer em um estado conhecido na inicialização, então provavelmente é uma boa ideia definir um valor para todos os controles no mixer aqui.

- Os comandos nas seções desta seção são todos comandos `cset`. Esses comandos correspondem aos comandos que você normalmente usaria com a ferramenta `amixer` do pacote alsa-utils. O primeiro comando faz isso quando executado com o amixer:
```
$ amixer cset name='Headphone Playback Switch' off
numid=36,iface=MIXER,name='Headphone Playback Switch'
  ; type=BOOLEAN,access=rw------,values=2
  ; values=on,on
```
- Para obter uma lista de todos os controles que você pode definir, você também pode usar o comando amixer:
```
$ amixer controls
numid=35,iface=MIXER,name='Headphone Source Playback Route'
numid=36,iface=MIXER,name='Headphone Playback Switch'
numid=17,iface=MIXER,name='Headphone Playback Volume'
... a lot more lines here ...
```
- Eles devem ser bastante fáceis de correlacionar com os controles que você vê no alsamixer. 
* Os controles seguem um esquema de nomenclatura padrão.
* Os controles deslizantes de volume para dispositivos na página de reprodução do alsamixer são chamados `$name Playback Volume` 
* O mesmo controle na página de captura é chamado `$name Capture Volume`.
* O botão mudo abaixo dos controles deslizantes de volume de reprodução é chamado `$name Playback Switche` os botões de ativação na página Captura são chamados `$name Capture Switch`. 
* Controles que não possuem um controle deslizante de volume,mas uma seleção de texto cujo nome é `$name Playback Route`.

![800px-Alsa_mixer_mapping](https://github.com/DereckySany/UCM2/blob/main/src/800px-Alsa_mixer_mapping.png)
### Depurando configurações do UCM
- Recarregue as configurações do UCM
`alsaucm reload` deve recarregar a configuração para você, mas isso não funciona no momento. Uma reinicialização completa é necessária para recarregar totalmente a configuração do UCM.

- Monitorar eventos ALSA
`alsactl monitor` permite monitorar os eventos ALSA, especialmente para detecção de jack. 
- O nome do evento no log deve corresponder à configuração do UCM JackControl se você quiser alterar as configurações do ALSA quando um conector for conectado ou removido.
# Ajuda e Creditos
[![Ajuda](https://github.com/DereckySany/UCM2/blob/main/src/logo.png)](https://wiki.postmarketos.org/wiki/Alsa_UCM).
