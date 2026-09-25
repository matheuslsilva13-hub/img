Importação de Módulos e Configuração Inicial
const express = require('express');: Importa a framework Express para criação do servidor web e gestão de rotas HTTP.

const mysql = require('mysql2');: Importa o driver do MySQL para permitir a comunicação com a base de dados.

const path = require('path');: Importa o módulo nativo do Node.js para manipular caminhos de ficheiros e diretorias.

const app = express();: Inicializa a aplicação Express.

app.use(express.json());: Middleware que converte automaticamente o corpo das requisições (body) em formato JSON para objetos JavaScript.

app.use(express.static(path.join(__dirname, 'publics')));: Define a pasta publics como repositório de ficheiros estáticos (HTML, CSS, imagens, JS do cliente).

2. Ligação à Base de Dados MySQL
const banco = mysql.createConnection({...}): Cria as configurações de acesso à base de dados MySQL local (127.0.0.1, utilizador root, base de dados cinema_db).

banco.connect((erro) => {...}): Tenta estabelecer a ligação. Exibe uma mensagem de sucesso no terminal se a ligação for bem-sucedida ou a mensagem de erro caso falhe.

3. Rota Raiz
app.get('/', (req, res) => {...}): Trata o acesso à URL principal (/). Envia o ficheiro cadastro.html localizado dentro da pasta publics.

4. Gestão de Utilizadores e Autenticação
POST /usuarios: Valida se todos os campos (nome, email, senha) foram enviados. Executa o comando INSERT. Se o e-mail já existir na base de dados (erro.code === 'ER_DUP_ENTRY'), devolve a mensagem de erro HTTP 400.

POST /login: Recebe o e-mail e a palavra-passe e executa um SELECT com a cláusula WHERE. Se não encontrar correspondências (resultados.length === 0), devolve o estado 401 (Não Autorizado).

GET /usuarios / GET /usuarios/:id: Retornam a lista completa de utilizadores ou um utilizador específico pelo seu identificador único.

PUT /usuarios/:id / DELETE /usuarios/:id: Atualizam e eliminam registos de utilizadores na base de dados verificando resultado.affectedRows para garantir que o ID existia.

5. Módulo de Filmes
GET /filmes: Executa uma consulta INNER JOIN entre as tabelas filmes e sessoes para listar apenas os filmes que possuem sessões ativas com bilhetes disponíveis (ingressos_disponiveis > 0).

CRUD de Filmes (GET /:id, POST, PUT, DELETE): Operações normais de leitura, criação, alteração e eliminação na tabela filmes.

6. Módulo de Salas
CRUD de Salas (GET, GET /:id, POST, PUT, DELETE): Operações de gestão da tabela salas, armazenando o número da sala e a sua capacidade.

7. Módulo de Sessões
GET /sessoes (Primeira versão): Projeta dados compostos relacionando as tabelas sessoes, filmes e salas. Utiliza DATE_FORMAT do MySQL para formatar a data (%d/%m/%Y) e a hora (%H:%i).

Nota Importante sobre Duplicação no Código: Existe uma segunda definição de app.get('/sessoes', async ...) logo após a primeira. Esta segunda definição irá sobrepor a primeira e gerar um erro em execução, pois tenta utilizar db.query assíncrono com await, mas a variável declarada no topo do ficheiro foi banco (usando funções de callback). Para o código funcionar sem falhas, a segunda definição deve ser removida.

POST /sessoes: Cria uma sessão atribuindo um filme, sala, horário e definindo a quantidade inicial de bilhetes (com valor padrão de 100 caso não seja informado).

8. Módulo de Bilhetes (Ingressos)
Operações diretas de leitura, criação e remoção de registos na tabela ingressos.

9. Finalização da Compra
POST /finalizar-compra:

Recebe o nome do cliente e um array de itens (bilhetes selecionados).

executarQuery: Cria uma wrapper function baseada em Promises para permitir o processamento sequencial de operações assíncronas utilizando async/await.

Ciclo for...of: Para cada item do carrinho:

Insere o registo de venda na tabela ingressos.

Atualiza a tabela sessoes reduzindo os bilhetes disponíveis com a instrução GREATEST(0, ingressos_disponiveis - ?), garantindo que o saldo de bilhetes nunca fique negativo.

10. Histórico de Compras e Arranque do Servidor
GET /compras: Executa um SELECT com múltiplos INNER JOIN unindo ingressos, sessoes, filmes e salas para construir o histórico consolidado de vendas, ordenando das compras mais recentes para as mais antigas (ORDER BY ingressos.id DESC).

app.listen(3000, ...): Inicia o servidor Web a escutar no porto 3000 da máquina local.
