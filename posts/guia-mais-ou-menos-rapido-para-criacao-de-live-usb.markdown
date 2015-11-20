## Índice:
1. [Introdução](#intro)
2. [Uma nota rápida sobre particionamento](#nota-particionamento)
3. [Mãos à Obra!](#maos-a-obra)
	1. [Windows](#windose)
	2. [Linux](#linux)
		1. [gdisk/fdisk + dd](#gdisk-fdisk-dd)
			* [fdisk (cheat sheet)](#fdisk)
			* [gdisk (cheat sheet)](#gdisk)
			* [Passo-a-passo](#passos)
		2. [gparted + unetbootin](#gparted-unb)
		3. [Menções honrosas](#extras)
	3. [OSX](#osx)
4. [Bonus stage: conversões entre GPT e MBR](#bonus)

<h2 id="intro">1.Introdução</h2>

Olá! Este guia foi feito na intenção de ser um tutorial simples de criação de um disco inicializável via USB, em virtude da oficina do próximo **7 Marias** que ocorre no próximo domingo (corre lá e [preencha nosso formulário][1] que ainda tem vaga), mas me dei liberdade de estender um pouco sobre os sistemas de partições e métodos disponíveis, priorizando a clareza em detrimento do tamanho, pois seria muito fácil mandar ir no Google ao invés de condensar conhecimento básico de forma didática. Vamos nessa?
   
<h2 id="nota-particionamento">2. Uma nota rápida sobre particionamento</h2>

Para começar, é importante explicar um detalhe sobre como funciona o boot, pois será necessário determinar isso para criar nosso pen drive inicializável. Durante muio tempo, leia-se até a era Windows 7, usamos um setor de inicialização de 512 bytes chamado MBR. A partir do Windows 8, a Micro$oft estabeleceu com diversas fabricantes de hardware um padrão chamado GPT (esse usa 440 bytes, só pra tu saber), feito para arquitetura 64-bits e que permite implementar um sistema de carregamento que é o UEFI, que (promete) dispensar a necessidade de bootloaders (como o GRUB e mesmo o sistema de inicialização do NT, configurável via msconfig) e implementar um método seguro de boot. 

Ok, não foi tão rápido, talvez muito técnico, mas era importante pra poder trocar em miúdos com vocês:

- Tornando simples: se sua máquina é anterior ao Windows 8, você vai usar instruções pra MBR, se ela já veio com o 8, certamente foi formatada como GPT. 
- A maioria das empresas formata seus OEMs (sistemas pré-instalados) com a opção de MBR protetiva, um modo híbrido de compatibilidade do GPT com MBR. Tirando a Dell, isso já vem habilitado. Neste caso, use a opção de boot legado para bootar. 
- À exceção de distribuições com foco mais corporativo de Linux, a maioria delas não conta com chaves de autenticação do módulo EFI. Logo, é recomendado desabilitar o boot seguro nas configurações da CMOS/BIOS. Consulte como fazer isso para o seu hardware. 

(Mas se quiser tentar criar sua própria chave assinada no formato kext não tô te impedindo, certas pessoas por aí fazem isso sabe…)

<h2 id="maos-a-obra">3. Mãos à obra!</h2>

<h3 id="windose">3.1 Windose</h3>

Era uma vez um mundo em que até havia ferramentas pra criar USBs inicializáveis, mas eram instáveis, ou de fontes duvidosas, e vez ou outra ainda não funcionavam legal. Até que fizeram o Rufus, que é tão fácil que decidi encher linguiça aqui. 

O que você fará é:
 
1. Conectar seu pen drive
2. Baixar o Rufus, míseros 845k
3. Abrir
4. Escolher o tipo de partição, com base no explicado lá em cima
5. Selecionar o local da imagem ISO
6. OK
7. Eventualmente ele vai pedir pra baixar uns arquivos adicionais de boot da internet. Pode confiar nele e confirmar
8. Ir pegar sua bebida favorita, fazer um xixi, ver uns gatos no youtube até ele dizer que acabou

<h3 id="linux">3.2 Linux</h3>

Se você prefere ou se sente mais confortável com programas gráficos, vá direto pra sessão [Gparted + Unetbootin][gparted-unb]. 

Vou começar pelo segundo método mais difícil (mas nem tanto) pois ele atende alquer sabor de Linux (e BSD, incluindo Mac OSX, mudando apenas as nomenclaturas dos discos). Mais difícil apenas por ser linha de comando. E se está curiosa sobre o primeiro lugar, é algo bem manual e inclusive com risco de perder o firmware do pen drive. Funciona, mas já tive que sacrificar pen drive por isso. Tejem avisadas. Mas não vou falar dele aqui. 

<h4 id="gdisk-fdisk-dd">3.2.1 gdisk/fdisk + dd</h4>

Resumo rápido: pra quem já tem familiaridade ou tá sem tempo mesmo  (se não entender nadinha, siga adiante)

<h5 id="fdisk">fdisk (cheat sheet)</h5>


	$ sudo umount
	$ sudo fdisk /dev/sdb
	Command (m for help): o
	Command (m for help): n
	Command (m for help): t
	Hex code (type L to list all codes): b
	Command (m for help): a
	Command (m for help): w
	$ sudo mkfs.vfat -F32 /dev/sdb1
	$ cd ~/ISO && cp image.iso image-hybrid.iso
	$ isohybrid image-hybrid.iso
	$ sudo dd bs=4M if=image-hybrid.iso of=/dev/sdb1 && sync


<h5 id="gdisk">gdisk (cheat sheet)</h5>

	$ sudo umount
	$ sudo fdisk /dev/sdb
	Command (? for help): o
	Command (? for help): n
	Command (? for help): t
	Hex code or GUID (L to show codes, Enter = 8300): 0007
	Command (? for help): x
	Command (? for help): n
	Command (? for help): w
	$ sudo mkfs.vfat -F32 /dev/sdb1
	$ cd ~/ISO && cp image.iso image-hybrid.iso
	$ isohybrid -u image-hybrid.iso
	$ sudo dd bs=4M if=image-hybrid.iso of=/dev/sdb1 && sync

<h5 id="passos">Passo-a-passo</h5>

1. Abra um terminal
2. Insira um pen drive. Ele vai montar, mas beleza
3. Normalmente o nome do primeiro disco rígido externo vem na sequência da letra do último interno. Por exemplo, se você tem um HD só ele seria /dev/sdb, três? /dev/sdd. Mas se precisar confirmar, digite `lsblk -o NAME,VENDOR`, que mostra a marca do disco pra você saber
4. Aí sim, você desmonta o pen drive. Se não desmontar, não funciona, se desmontar pelo modo de janela, vai ejetar o disco (deixar de ser reconhecido como bloco) e também não funciona. Digite `sudo umount /dev/sdX` (X é a letra correspondente ao pen drive)
5. Agora vai. Você vai usar `fdisk /dev/sdX` se for MBR e `gdisk /dev/sdX` se for GPT. Sempre. Os comandos são (quase) os mesmos, então feito isso seguimos assim:
	1. digite o e confirme
	2. Digite n, e confirme todas as informações sem hesitar, até a mensagem de partição linux criada
	3. Agora digite t. Geralmente as ISOs híbridas não são “cruas”, elas possuem informação de sistema de arquivos. Mas o dd, que vamos usar pra escrever a ISO, escreve cru. Logo vamos criar um disco FAT, formato padrão de USBs. 
	4. Agora, digite b se for fdisk, ou 0007 se for gdisk e taca o enter
	5. Quase pronto:
		1. No caso do fdisk, digite a para habilitar a MBR como inicializável. Digite w e fim da brincadeira. 
		2. No gdisk precisamos ativar o modo protetivo. Digite x e entraremos em modo avançado. Digite n. Aí sim, m pra retornar ao menu principal e w. Ié.
6. Pra fazer a partição uma partição, não basta falar isso pra ela, tem que formatar. E usaremos o seguinte comando pra isso: `mkfs.vfat -F32 /dev/sdX`
7. A maioria das ISOs de sistemas live já são híbridas por padrão, ou seja, fazem o boot se forem escritas em DAO (leia-se escrita de acesso direto usada em disco óptico) ou em raw (escrita crua em dispositivos de bloco). Mas por via das dúvidas: vá até o diretório da sua imagem ISO e digite (substitua image.iso pelo nome do arquivo baixado): `cp image.iso image-hybrid.iso && isohybrid image-hybrid.iso`. No caso do GPT, é preciso usar a flag `-u` para indicar à ISO o boot pela UEFI.
8. Agora o golpe final hein. Você vai digitar essa beleza: `dd bs=4M if=image-hybrid.iso of=/dev/sdX && sync`
9. Isso vai fazer com que a ISO seja passada bloco a bloco para o pen drive. Agora é só esperar o comando sair e ser feliz

<h4 id="gparted-unb">3.2.2 Gparted + Unetbootin></h4>

O Gparted é uma ferramenta muito conhecida por ser a primeira ferramenta livre vom funcionalidades rivais às do clássico Partition Magic para Windows, especialmente quando passou a vir em live distros como o Knoppix e o recem-lançado Ubuntu. É uma poderosa e intuitiva ferramenta de particionamento e madura o bastante para estar presente em possivelmente qualquer distribuição atual. 

O Unetbootin nasceu no mundo Ubuntu e, por isso (e outro motivo) não é a ferramenta mais estável fora deste mundo. Motivo pelo qual preferi um método mais genérico primeiro; o que a ferramenta faz é extrair a ISO para o pen drive e criar um bootloader de acordo com ela. Mas a capacidade de iniciar o disco ainda depende da formatação prévia. 

Como são ferramentas gráficas, documentei todo o processo passo-a-passo com imagens e postei [aqui][2] para não sobrecarregar sua rede de graça (a senha é gparted). Os passos são os seguintes:

1. Abra o Gparted a partir do menu. Ele solicitará autenticação
2. Ao iniciar, o gparted mostra o seu disco principal. Vá até a caixa de seleção no canto superior direito e escolha o seu pen drive
3. No menu superior, vá até Dispositivo > criar tabela de partição
4. Selecione de acordo: msdos para MBR ou gpt. Haverá uma confirmação da destruição dos dados
5. Feito isso, use o primero botão à esquerda no menu para criar uma partição
6. Selecione fat32 na caixa Sistema de Arquivos e confirme
7. Clique no último botão à direita para aplicar as operações e aguarde o fim do processo
8. Voltando à tela principal, clique com o botão direito na partição recém-criada, na janela inferior. Selecione Gerenciar sinalizadores
9. Caso sua partição seja MBR, basta marcar a opção boot. Na GPT, boot e boot_legacy (isso ativa a opção esp, mantenha marcada). Confirme e feche o gparted ao fim disso
10. Agora, abra o Unetbootin, que também requer autenticação. Remonte seu pen drive antes de continuar (retirar e inserir é ok)
11. Selecione Criar imagem ISO, e então clique no botão de seleção de arquivo (…) para indicar o local do arquivo
12. Selecione o pen drive, aperte ok e aguarde a operação

<h4 id="extras">3.2.3 Menções honrosas</h4>

**mintstick** - programa padrão de criação de USBs inicializáveis do Linux Mint, disponível para qualquer distribuição da família Debian e mesmo algumas de fora. Sua simplicidade mereceria primeiro lugar se também não fosse seu maior pecado: você ainda precisa criar a tabela de partição e a iso híbrida a parte (mas no caso dela um simples `fdisk -o /dev/sdX` (ou gdisk) resolve tudo

**multibootUSB** - essa ferramenta é overpower demais pra fazer um pen drive simples, mas precisa ser lembrada pela capacidade de criar um disco com vários boots diferentes. Alegria das mina do suporte e das testadeiras

**live-USB-install** - tenho a impressão de que essa não é mais mantida por conta de alguns bugs apresentados, mas posso estar errada. Ela teria tudo pra ser a preferida entre todas: possui uma lista extensa de live distros que podem ser baixadas usando a libtorrent como motor, permite uma ISO já baixada, cria a tabela de partição e até permite indicar alguma distro faltante na lista. Infelizmente as duas últimas não funcionaram no Arch (que eu utilizo) e talvez o binding do python  para a libtorrent (necessário para essa funcionalidade) precise ser compilado

<h3 id="osx">3.3 OSX</h3>

Achou que eu ia esquecer vocês? Aqui não tem segregação, mana. O OSX possui restrições para boot de kernel Linux, transpostas utilizando o rEFIt bootloader, ou Clover. O mesmo para criar um USB inicializável. Mas aí tem o Mac Linux USB Creator pra matar os dois coelhos geneticamente modificados com sangue de troll. Os passos foram extraídos do [How to Geek][3] então dá uma olhada que tem os passos com imagem lá. 

1. Insira o pen drive e abra o DiskUtility
2. Escolha o disco do pen drive e delete a partição, criando uma partição FAT em seguida
3. Feche, e então abra o MLUSBC
4. Selecione Create live USB
5. Escolha a ISO e em seguida o pen drive
6. Selecione a família da distro: por exemplo, o AV Linux que usaremos nas oficinas é baseado em Debian. Mint em Ubuntu. Slax em Slackware. Manjaro em Arch. Fedora em Red Hat. É por aí
7. Aí é só aplicar e comemorar

<h2 id="bonus">Bonus stage: conversão de GPT para MBR</h2>

Em alguns casos específicos, seja teste ou compatibidade com sistemas mais antigos, especialmente Windows ou Linuxes dependentes de aplicações 32-bit, pode ser uma ideia saudável converter a tabela GPT em MBR, e isso é simples. Requer apenas duas condições devido a limitações da MBR:

- Seus discos rígidos internos não podem ter capacidade maior do que 2 TB, individualmente
- Seu disco inicializável não pode contar mais de 4 partições primárias

Dito isso, o processo utiliza o mesmo gdisk já apresentado anteriormente, e claro, um Linux. Para exemplificar, uso o processo como feito através de um live do Ubuntu. 

1. Inicie o sistema operacional através do menu de boot do seu computador
2. Instale o gdisk. No Ubuntu isso seria feito acessando a Central de Software, e escolhendo Fontes no menu. Marque a opção relativa ao repositório (universe)
3. Em seguida, use os comandos:

<pre><code>
	$ sudo apt-get update
	$ sudo apt-get install gdisk
</code></pre>

4. Verifique corretamente o nome do disco inicial. O padrão é /dev/sda. Prossiga.	

<pre><code>
	$ gdisk /dev/sda
	Command (? for help): r
	Command (? for help): g
	Command (? for help): m
	Command (? for help): w
</code></pre>

5. O r acessa o modo de transformação, e o g converte a tabela. Simples assim. 

##### Converter de volta para GPT

Caso seja preciso voltar a usar GPT, o processo é igualmente simples. Repita os passos de 1 a 4, e na tela do gdisk (aquela que mostra "Command"), digite apenas w (e y para confirmar).

EOF gente, espero ter ajudado e qualquer coisa dá uma chegadinha na nossa lista de e-mails, teremos prazer em ajudar (e no fórum, em breve!).

[1]:http://goo.gl/forms/eZhr6PKyM0
[2]:http://marialab.com.br/opt/guia-visual-gparted-unetbootin
[3]:http://www.howtogeek.com/213396/how-to-boot-a-linux-live-usb-drive-on-your-mac/
