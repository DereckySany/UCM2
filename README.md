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

