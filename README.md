# EmulatorJS_Docker

As arquiteturas suportadas por esta imagem são:

Arquitetura	Disponível	Marcação
x86-64	✅	amd64-<marca de versão>
braço64	✅	arm64v8-<marca de versão>
armhf	✅	arm32v7-<tag de versão>
Configuração do aplicativo
O Backend pode ser acessado em:

http://seuhost:3000/
A primeira coisa que você precisa fazer é clicar para baixar a arte/configurações padrão desta interface, isso irá configurar um diretório esqueleto em seu /datamount. A partir daí, adicione roms aos respectivos romsdiretórios e siga as instruções na tela para adicioná-los ao seu front-end da Web em execução na porta 80.

O aplicativo front-end foi inicialmente otimizado para ser usado com um gamepad padrão (mais especificamente para consoles Xbox modernos que possuem navegadores Edge baseados em cromo). A navegação gira em torno das teclas para cima/baixo/esquerda/direita para navegar pelos menus e iniciar jogos. Os navegadores móveis funcionarão, mas lembre-se de que a compatibilidade será reduzida, especialmente para jogos baseados em CD.

É importante notar que alguns dos emuladores atuais usados ​​para este frontend são códigos ofuscados, esforços estão sendo feitos para fazer engenharia reversa , mas você deve saber que ele pode potencialmente alcançar serviços de terceiros se você habilitar manualmente recursos como netplay (isso deve nunca acontece em uma configuração de estoque). O objetivo desta mensagem é que, além do esforço de desofuscação, há também um esforço para parar de usar blobs binários e mudar para blobs libretro emscripten criados a partir da fonte, por enquanto esta pilha de emulação baseada na Web é a melhor em termos de usabilidade e compatibilidade. Estamos em processo de transição para núcleos libretro para emuladores, atualmente 27/30 emuladores foram substituídos.

Para usuários do Xbox, clique no botão de seleção algumas vezes após iniciar um jogo para garantir que o botão B não acione uma ação de "voltar" no navegador. (nome oficial "botão de visualização" são os dois pequenos quadrados) A saída do modo do controlador e o retorno aos controles do navegador podem ser acionados pressionando o botão Iniciar por 3 segundos. (nome oficial "botão de menu" as três linhas) Você não poderá usar recursos como salvar estados e modificar layouts de controlador nos emuladores baseados em emulatorjs atualmente, pois não determinei uma metodologia de reentrar no modo de controlador depois de sair dele. Todos os jogos salvos normais funcionarão desde que você saia da tela de jogo usando o botão B para voltar, isso inclui jogos com vários discos para psx. Os jogos salvos são armazenados no armazenamento do navegador pelo nome do host, portanto, se você fizer qualquer alteração na configuração do seu host local (porta ou IP), os salvamentos não seguirão com ela. Para emuladores baseados em libretro, você pode usar a combinação de botões iniciar+selecionar+L+R para acessar o menu do libretro e alterar as configurações/salvar ou carregar/etc.

Conhecemos a maioria das esquisitices, como som de crepitação para alguns emuladores, problemas de renderização e jogos que iniciam automaticamente em tela cheia de maneira não confiável. Em geral, os jogos de CD completos no navegador do Xbox parecem não funcionar devido ao seu tamanho, se você tiver um chd/pbp menor que 450 megas, ele será executado. O Edge no Xbox tem algum tipo de limitação de RAM não documentada de cerca de um gigabyte. Até que todos os emuladores sejam transferidos para núcleos libretro, as esquisitices de usar o EmulatorJS auto-hospedado não serão algo que possa ou deva ser resolvido usando soluções alternativas hacky interagindo com código ofuscado. Apenas tenha em mente que estes são emuladores de máquina completos rodando em Javascript em um navegador, não espere desempenho bare metal.

A montagem em diretórios rom existentes pode ser obtida apontando para a estrutura de pastas padrão, ou seja, digamos que você gostaria de montar sua biblioteca NES:

-v /path/to/nes/roms:/data/nes/roms

Os nomes das pastas são:

3do
videogames
atari2600
atari7800
colecovisão
ruína
GB
gba
GBC
jaguar
lince
msx
n64
nds
nes
NGP
odisseia2
pce
psx
sega32x
segaCD
segaGG
segaMD
segaMS
segaSaturno
segaSG
snes
vb
vectrex
ws
Uso
Aqui estão alguns trechos de exemplo para ajudá-lo a começar a criar um contêiner.

docker-compose (recomendado, clique aqui para mais informações )
---
version: "2.1"
services:
  emulatorjs:
    image: lscr.io/linuxserver/emulatorjs:latest
    container_name: emulatorjs
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - SUBFOLDER=/ #optional
    volumes:
      - /path/to/config:/config
      - /path/to/data:/data
    ports:
      - 3000:3000
      - 80:80
      - 4001:4001 #optional
    restart: unless-stopped
docker cli ( clique aqui para mais informações )
docker run -d \
  --name=emulatorjs \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e SUBFOLDER=/ `#optional` \
  -p 3000:3000 \
  -p 80:80 \
  -p 4001:4001 `#optional` \
  -v /path/to/config:/config \
  -v /path/to/data:/data \
  --restart unless-stopped \
  lscr.io/linuxserver/emulatorjs:latest

Parâmetros
As imagens de contêiner são configuradas usando parâmetros passados ​​em tempo de execução (como os acima). Esses parâmetros são separados por dois pontos e indicam <external>:<internal>respectivamente. Por exemplo, -p 8080:80exporia a porta 80de dentro do contêiner para ser acessível a partir do IP do host na porta 8080fora do contêiner.

Parâmetro	Função
-p 3000	Interface de gerenciamento de ROM/artwork, usada para gerar/gerenciar arquivos de configuração e baixar a arte
-p 80	Frontend de emulação contendo arquivos da web estáticos usados ​​para navegar e iniciar jogos
-p 4001	Porta de emparelhamento IPFS, se você deseja participar da rede P2P para distribuir arte de front-end, encaminhe-a para a Internet
-e PUID=1000	para UserID - veja abaixo para explicação
-e PGID=1000	para GroupID - veja abaixo para explicação
-e TZ=Etc/UTC	especificar um fuso horário para usar, veja esta lista .
-e SUBFOLDER=/	Especifique uma subpasta para proxies reversos IE '/FOLDER/'
-v /config	Caminho para armazenar perfis de usuário
-v /data	Caminho para armazenar roms/artwork
Variáveis ​​de ambiente de arquivos (segredos do Docker)
Você pode definir qualquer variável de ambiente de um arquivo usando um prefixo especial FILE__.

Como um exemplo:

-e FILE__PASSWORD=/run/secrets/mysecretpassword
Definirá a variável de ambiente PASSWORDcom base no conteúdo do /run/secrets/mysecretpasswordarquivo.

Umask para aplicativos em execução
Para todas as nossas imagens, fornecemos a capacidade de substituir as configurações de umask padrão para serviços iniciados nos contêineres usando a -e UMASK=022configuração opcional. Lembre-se de que umask não é chmod, ele subtrai das permissões com base no valor que não adiciona. Por favor, leia aqui antes de pedir suporte.

Identificadores de usuário/grupo
Ao usar volumes ( -vsinalizadores), problemas de permissão podem surgir entre o sistema operacional host e o contêiner, evitamos esse problema permitindo que você especifique o usuário PUIDe o grupo PGID.

Certifique-se de que todos os diretórios de volume no host pertençam ao mesmo usuário especificado e que quaisquer problemas de permissão desapareçam como mágica.

Neste caso PUID=1000e PGID=1000, para encontrar o seu, use id usercomo abaixo:

  $ id username
    uid=1000(dockeruser) gid=1000(dockergroup) groups=1000(dockergroup)
Mods do Docker
Mods do Docker Mods universais do Docker

Publicamos vários Docker Mods para permitir funcionalidades adicionais dentro dos contêineres. A lista de Mods disponíveis para esta imagem (se houver), bem como mods universais que podem ser aplicados a qualquer uma de nossas imagens podem ser acessados ​​através dos emblemas dinâmicos acima.

Informações de suporte
Acesso ao shell enquanto o contêiner está em execução:docker exec -it emulatorjs /bin/bash
Para monitorar os logs do container em tempo real:docker logs -f emulatorjs
número da versão do contêiner
docker inspect -f '{{ index .Config.Labels "build_version" }}' emulatorjs
número da versão da imagem
docker inspect -f '{{ index .Config.Labels "build_version" }}' lscr.io/linuxserver/emulatorjs:latest
Atualizando informações
A maioria de nossas imagens são estáticas, com versão e requerem uma atualização de imagem e recriação de contêiner para atualizar o aplicativo interno. Com algumas exceções (por exemplo, nextcloud, plex), não recomendamos nem oferecemos suporte à atualização de aplicativos dentro do contêiner. Consulte a seção Configuração do aplicativo acima para ver se é recomendado para a imagem.

Abaixo estão as instruções para atualizar os contêineres:

Via Docker Compose
Atualize todas as imagens:docker-compose pull
ou atualizar uma única imagem:docker-compose pull emulatorjs
Vamos compor atualizar todos os contêineres conforme necessário:docker-compose up -d
ou atualize um único contêiner:docker-compose up -d emulatorjs
Você também pode remover as antigas imagens pendentes:docker image prune
Via Docker Run
Atualize a imagem:docker pull lscr.io/linuxserver/emulatorjs:latest
Pare o contêiner em execução:docker stop emulatorjs
Exclua o contêiner:docker rm emulatorjs
Recrie um novo contêiner com os mesmos parâmetros de execução do docker conforme instruído acima (se mapeado corretamente para uma pasta de host, sua /configpasta e configurações serão preservadas)
Você também pode remover as antigas imagens pendentes:docker image prune
Via atualizador automático da Watchtower (use apenas se você não se lembrar dos parâmetros originais)
Puxe a imagem mais recente em sua tag e substitua-a pelas mesmas variáveis ​​de ambiente em uma execução:

docker run --rm \
-v /var/run/docker.sock:/var/run/docker.sock \
containrrr/watchtower \
--run-once emulatorjs
Você também pode remover as antigas imagens pendentes:docker image prune

Observação: não endossamos o uso do Watchtower como uma solução para atualizações automatizadas de contêineres Docker existentes. Na verdade, geralmente desencorajamos atualizações automáticas. No entanto, esta é uma ferramenta útil para atualizações manuais únicas de contêineres nos quais você esqueceu os parâmetros originais. A longo prazo, recomendamos o uso do Docker Compose .

Notificações de atualização de imagem - Diun (Docker Image Update Notifier)
Recomendamos Diun para notificações de atualização. Outras ferramentas que atualizam contêineres automaticamente sem supervisão não são recomendadas ou suportadas.
Construindo localmente
Se você quiser fazer modificações locais nessas imagens para fins de desenvolvimento ou apenas para personalizar a lógica:

git clone https://github.com/linuxserver/docker-emulatorjs.git
cd docker-emulatorjs
docker build \
  --no-cache \
  --pull \
  -t lscr.io/linuxserver/emulatorjs:latest .
As variantes ARM podem ser construídas em hardware x86_64 usandomultiarch/qemu-user-static

docker run --rm --privileged multiarch/qemu-user-static:register --reset
Uma vez registrado, você pode definir o dockerfile para usar com -f Dockerfile.aarch64.
