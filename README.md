Desafio: Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux.

Primeira Etapa:
Realizei a preparação do ambiente para desafio instalando o Virtual Box no meu computador em seguida criando duas VMs. Uma para Kali Linux e outra para Metasploitable 2, que será nosso host onde realizaremos as simulações de ataques do desafio.
Iniciando a simulações de ataque:
Primeira simulação:  força bruta em FTP.
Para iniciar a simulação vou primeiramente realizar enumeração, que seria descobrir serviços que estão disponíveis no meu host alvo do ataque.
Vou utilizar uma ferramenta que encontramos no Kali Linux, que será o NMAP com ele podemos realizar a enumeração e verificar serviços disponíveis. Sendo assim no terminal do Kali Linux vou executar o comando abaixo.

Nmap -Sv -p 21,22,80,445,139  192.168.56.105

Esse comando vai scanear as portas mencionadas no comando e tentar identificar a versão do serviço executado se elas estão abertas.  Onde tiver porta aberta, significa que tenho um serviço desatualizado e vulnerável para ataque.
Nosso caso nosso alvo será porta 21 que é porta serviço FTP.
Finalizando o scanner podemos identificar que porta 21 – FTP esta aberta, para confirmar que ela está aberta executamos o comando.

ftp 192.168.56.105

Pronto. Resultado, conexão realizada ao serviço FTP.
Agora sabendo que serviço está aberto e ativo, vou realizar ataque de força bruta com ajuda Medusa, ferramenta simples e fácil de usar do Kali Linux, mas antes de partir para Medusa preciso criar duas listas uma de usuários e umas senhas comuns usadas em arquivos do host qual está ‘com porta FTP aberta, para isso utilizo os seguintes comandos

Criando a lista de usuários

echo  -e  “user\nmsfadmin\nadmin\nroot”  > users.txt

Esse comando vai criar um arquivo txt com usuários comuns.

Criando a lista de senhas comuns

echo  -e “123456\npassword\nqwerty\nmsadmin”  >  pass.txt

Esse comando vai criar uma lista de senha em formato txt.

Com os arquivos txt da lista de usuários e senhas criados vou iniciar o ataque com ajuda do Medusa, para isso vamos executar o comando abaixo.

medusa -h 192.168.56.105 -U users.txt  -P  pass.txt  -M  ftp  -t  6

Esse comando irá atacar o serviço FTP buscando uma combinação de senha e usuário validos para acesso ao serviço. Encontrando uma combinação valida, mostrara essa combinação com a informação SUCESS indicando que combinação poderá ser usada para acessar serviço.

Para validar esse acesso vou novamente executar comando:
 ftp 192.168.56.105 e informar usuário e senha encontrado no ataque do Medusa.

Conclusão do ataque e Recomendações para mitigação.

Devemos saber quais versões dos serviços executados em nossos sistemas, essa informação deve ser bem guardada, pois se alguém descobre uma vulnerabilidade nessa versão podemos virar alvo de ataques.
Também que serviços mal configurados, com portas abertas, servidores desatualizados, com credenciais fracas viram alvos fáceis desses ataques.
Para se proteger recomendamos desativar serviços desnecessários,  usar protocolos mais modernos, se possível trocar a numero da porta do serviço por um não tão convencional, utilizar senhas fortes e politicas de bloqueio,  criar longas e que sejam alteras de tempo em tempo, bloqueio por numero de tentativas ajudam barra ataques de força bruta, usar autenticação multifator, pois mesmo que descubra a senha e usuário o segundo fator impede o acesso, realizar auditorias com frequência e monitorar e alertar a equipe de segurança toda tentativa repita de autenticação. 

Segunda simulação:  automação de tentativas em formulário web (DVWA)
Nesse tipo de ataque podemos verificar que ataque de força bruta não somente é aplicado em serviços, como FTP, mas também em sistemas web, um dos caminhos para esses ataques são os formulários de login que são portas de entrada praticamente em todas as aplicações web.
Com esse ataque automatizamos um tipo de ataque que automatiza tentativas de login em massa, simulando preenchimento de formulários com múltiplas combinações de usuários e senhas.
Para simular esse ataque vamos usar o sistema DVWA que está instalado em nosso servidor metasploitable, esse sistema apresenta formulário web que usuário irá digitar usuário e senha, vamos buscar entender como funcionar esse formulário de login e suas brechas.
Automatizando esse ataque vamos buscar descobrir usuários e senhas fracos para ter acesso a esse formulário.
Primeiro passo é entender como formulário funciona e suas possíveis brechas, digito endereço do formulário que será:
http://192.168.56.105/dwa/login.php
Agora no navegador usado vou abrir barra de desenvolver para entender funcionamento e como é enviado os dados para servidor do formulário web, com essas informações e como realizado o login junto ao servidor, posso iniciar automatização do ataque a esse formulário.
Para essa automatização vou fornecer ao Medusa duas Wordlist com usuários e senhas possíveis, 
Wordlist são arquivos textos que contêm senhas e usuários comuns que serão usados pelo Medusa.
Para criar essas duas Wordlist executo os seguintes comandos:
echo  -e  “user\nmsfadmin\nadmin\nroot”  > users.txt

echo  -e “123456\npassword\nqwerty\nmsadmin”  >  pass.txt

Com as Wordlist criadas vou utilizar o comando abaixo com Medusa, para automatizar o ataque.

medusa  -h 192.168.56.105  -U  user.txt  -P pass.txt  -M http \
-m PAGE: ‘ /dwa/login.php  \
-m FORCE: ‘usernane=^USER^&password=^PASS^&Login’ \
-m ‘FAIL=Login failed’ – t 6

Nesse comando informamos o endereço do nosso alvo, arquivo com usuários e senhas, informo caminho do formulário de login, corpo da requisição e por fim informa à medusa a informação de falha e usara 6 conexões simultâneas para acelerar o processo do ataque.
Vamos procurar na lista gerada as linhas com a informação ACCOUNT FOUND, nessas linhas estarão os possíveis usuários e senhas que podemos usar no formulário para acesso.

Forma de mitigação

Incorporar a esses formulários e sistemas web controles de segurança como:
Limites de tentativas por IP, bloqueio temporário após várias tentativas, utilizar captcha, autenticação multifator, monitoramento de logs com alerta. Esses ajustes tornam os ataques mais difíceis de acontecer.

Terceira simulação:  password spraying em SMB com enumeração de usuários.
Esse tipo de ataque de força bruta usa o protocolo SMB como entrada do nosso ataque.
SMB é protocolo de rede Microsoft utilizado para compartilhar arquivos, pastas, impressoras, autenticação de usuários e comunicação entre máquinas Windows e Linux via Samba.
Nesse tipo de ataque descobrimos um servidor SMB ativo e aberto, em seguida descobrimos usuários existentes e depois testamos senhas fracas em todos eles de forma discreta, pois não usa vulnerabilidades de softwares, mas sim inteligência, paciência e senhas fracas criadas pelos usuários para realizar o ataque.
Para que isso aconteça de forma discreta sem ser identificado por sistema de irão bloquear os usuários testados uma a técnica de password spraying. Password Spraying é técnica furtiva de ataque a senhas, então ao invés de testar várias senhas para um usuário, realizado teste de uma senha comum para muitos usuários diferentes.
Essa técnica é usada por ransonware, botnets automatizados, grupo mal-intencionados etc. É um ataque silencioso, difícil de ser detectado e muito eficaz se os usuários colocarem senhas fracas.
Nosso cenário será um servidor de SMB qual configurado e usuários com senhas fracas.
Para iniciar processo de ataque vamos realizar a enumeração de usuários com ajuda do ENUM for Linux. ENUM interage com serviço SMB mal configurado e extrai as informações necessários para nosso ataque. Para executar a ferramenta utilizo o seguinte comando:

enum4linux -a 192.168.56.105 | tee enum4_output.txt

Ao finalizar execução do Enum4Linux será gerado um arquivo txt com resultado nosso scanner ao serviço SMB, para visualizar o conteúdo do arquivo utilizo o comando:

less enum4_output.txt

Com acesso ao arquivo do comando posso explorar as informações encontradas, com grupos, contas reais dos usuários. Com essas informações descobrimos quais serão nossos alvos. Essa base também poderia vir de vazamentos de dados.
Agora nossa próxima etapa é criar a wordlist de usuários, para isso executamos o comando:

echo -e “user\nmsfadmin\nservice” > smb_users.txt

Vamos agora criar nossa wordlist de senhas reais para isso executo o comando:

echo -e “password\n123456\nwelcome123\nmsfadmin” > senhas_spray.txt

Ideal seria montar uma Wordlist com base na política de senhas da empresa alvo ou em dados vazados dela.
Com as Wordlist criadas vou executar o ataque através da ferramenta medusa.
Executo o comando abaixo.

medusa -h 192.168.56.105 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 56

Ao finalizar a execução do comando será exibido uma lista onde cada linha foi teste de usuário e senha, nessas linhas onde estiver ACCOUNT FOUND são ao possíveis senhas que poderemos usar para acesso ao serviço SMB.
Com essas informações vou acessar servidor SMB do meu servidor alvo. Para isso executo o comando:

smbclient -L //192.168.56.105 -v nmsfadmin

Informo usuário e senha encontrado no ataque com medusa.

Pronto, acessamos o servidor SMB do nosso alvo.
Nesse ataque podemos ver que nenhum código malicioso foi utilizado, nenhuma vulnerabilidade foi explorada, somente obtivemos informações públicas accessíveis como enumeração SMB, testamos senhas fracas que vários usuários ainda usam e usamos ferramenta simples que é medusa. É tipo de ataque que rápido, silencioso e ocorre em todos os momentos.
Isso nos mostra que não adianta investir milhões em firewall e outras ferramentas se usuário ainda usa senhas simples, fracas e comum de ser usadas por várias pessoas.
Para mitigar esse tipo de ataque precisamos usar autenticação multifator, criar senhas fortes e que expiram regularmente, pois senhas que não são eternizadas são mais difíceis de serem descobertas, bloquei de ips com bloqueio depois de múltiplas tentativas, monitoramento de logs do sistema e com base nesses logs descobrir falhas, aplicar segmentação da rede para que serviços não fiquem expostos. 
Após todas essas simulações de ataques concluímos que segurança ela falha quando o básico é negligenciado, pois podemos verificar que para realizar os cenários de nossos ataques utilizados uma ferramenta simples e gratuita e lista com usuários e senhas fracas criada pelo próprio usuário dos serviços alvo dos nossos ataques
Para solucionar isso como já foi mencionado anterior é necessário configurar melhor serviços, desativar o serviço se ele não é usado. Fazer que o usuário adote a ideia e pensamento de criar senhas fortes não previsíveis. 
Fundamentalmente implantar controles de segurança em nível estrutural e comportamental dos usuários, da empresa. 
