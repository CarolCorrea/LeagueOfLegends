# LeagueOfLegends
Projeto SSIS para extrair Dados da API League of Legends e gravar em Banco de Dados SQL Server, para uso em análises de dados no Power BI.

Link Projeto PowerBI: xxxx

<br />

## Pré-requisitos
Instalação do Visual Studio e SQL Server.

<br />

## API Riot - League of Legends
A Riot disponibliza diferentes APIs para extrair dados de seus jogos League of Legends, Legends of Runeterra e Teamfight Tatics.
No menu "Docs" é possível consultar as orientações sobre como usar as APIs de cada jogo.
No menu "APIS" é possível consultar as APIs disponíveis, como acessa-las, os parâmetros necessários, a descrição dos dados retornados, e os erros de resposta. Também é possível fazer requisições para verificar os retornos.
Obs. Há um limte de 20 requisições por segundo, e 100 requisições por minuto.

Link da API: https://developer.riotgames.com/

Nesse projeto utilizamos as APIs:
- SUMMONER-V4 (Get Summoner by SummonerName)
- MATCH-V5 (Get MatchIds by PuuId)
- MATCH-V5 (Get Match by MatchId)

Pré-requisitos para utilização das APIs:
1º. Criar uma conta e fazer login.
2º. Acessar o menu "Dashboard" e gerar uma nova API Key. Esta chave é usada para desenvolvimento, portanto expira em 24h. Após esse período é necessário gerar uma nova chave.

<br />

## Orientações sobre o Projeto SSIS
O projeto é composto por 4 pacotes SSIS.

Pacote Dimensões Stage: 
- Busca as Tabelas Dimensões (Modos, Mapas, Filas, Campeões, Itens e Feitiços) a partir de arquivos JSON disponibilizados pela Riot em links fixos, e adiciona no Banco de dados Stage. 
- Busca a Tabela Dimensão Lanes a partir de um arquivo CSV gerado manualmente, e adiciona no Banco de dados Stage. 
- Variáveis: url (Link do diretório base onde a Riot disponibiliza as imagens. Usada para buscar as imagens dos Campeões, Itens e Feitiços no Power BI. Precisa ser atualizado, caso haja alteração pela Riot)

Pacote Dimensões DW:
- Busca as Tabelas Dimensões (Modos, Mapas, Filas, Campeões, Itens, Feitiços e Lanes) no Banco de Dados Stage e replica para o Banco de Dados DW.

Pacote Fatos Stage:
- Busca o Invocador através da API SUMMONER-V4 (Get Summoner by SummonerName), e adiciona a Tabela Invocador no Banco de dados Stage. 
- Busca os IDs das Partidas através da API MATCH-V5 (Get MatchIds by PuuId), e adiciona a Tabela ID_Partidas no Banco de dados Stage. 
- Busca as Partidas através da API MATCH-V5 (Get Match by MatchId), e adiciona as Tabelas Partidas, Times, Times_Banimentos, Participantes, Participantes_Itens no Banco de dados Stage. 
- Variáveis: * apiKey (API Key gerada no Site da Riot. Usada para acessar as APIs. Precisa ser atualizada manualmente a cada dia, pois expira em 24h)
	     * invocador (Nome do Invocador. Usada para buscar os dados das APIs. Precisa ser atualizada manualmente com o SummonerName do jogador que deseja buscar os dados)
	     * url (Link do diretório base onde a Riot disponibiliza as imagens. Usada para buscar a imagem do Ícone de Invocador no Power BI. Precisa ser atualizado, caso haja alteração pela Riot)
	     * puuId (puuId do invocador principal. Usada para buscar os IDs das Partidas. É atualizada automaticamente, portanto não precisa de alteração manual)
	     * id (Número da Partida Inicial. Usada para buscar os dados das Partidas. Mante-la igual a 1)
	     * idMax (Número da Partida Final. Usada para buscar os dados das Partidas. É atualizada automaticamente, portanto não precisa de alteração manual)
	     * idCont Contador de Partidas. Usada para pausar a execução por 2min, após 95 requisições. É atualizada automaticamente, portanto não precisa de alteração manual)
	     * idPartida (ID da Partida. Usada para buscar os dados das Partidas. É atualizada automaticamente, portanto não precisa de alteração manual)
	     * partidaIni (Número da Partida Inicial. Usada para buscar os IDs das Partidas. Mante-la igual a 1)
	     * partidaFim (Número da Partida Final. Usada para buscar os IDs das Partidas. Está definida como 1000, mas caso desejar buscar mais partidas, o número pode ser aumentado)
	     * partidaImportada (Variável lógica. Usada para identificar se a partida foi importada ou não. É atualizada automaticamente, portanto não precisa de alteração manual)

Pacote Fatos DW:
- Busca as Tabela Dimensão Invocador no Banco de Dados Stage e replica para o Banco de Dados DW.
- Busca as Tabelas Fatos (Partidas, Times, Times_Banimentos e Participantes_Itens) no Banco de Dados Stage, executa algumas tarefas para preparar os dados para análise no Power BI, e replica para o Banco de Dados DW.
- Busca a Tabelas Fato Participantes no Banco de Dados Stage, executa algumas tarefas para preparar os dados para análise no Power BI, e divide entre as Tabelas Participante_Principal, Aliados e Adversários do Banco de Dados DW.
- Variáveis: invocador (SummonerName do invocador principal. Usada para dividir as Tabelas de Participantes e realizar transformações na Tabela Times. É atualizada automaticamente, portanto não precisa de alteração manual)

