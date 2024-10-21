# Mergulhando no Delta Lake: Desvendando o Log de Transações

O log de transações é a chave para entender o Delta Lake, pois é o elo comum que percorre muitos de seus recursos mais importantes, incluindo transações ACID, manuseio escalável de metadados, time travel e muito mais. Neste artigo, exploraremos o que é o log de transações do Delta Lake, como ele funciona no nível de arquivo e como oferece uma solução elegante para o problema de leituras e gravações concorrentes.

## O que é o Log de Transações do Delta Lake? 

O log de transações do Delta Lake (também conhecido como DeltaLog) é um registro ordenado de todas as transações que já foram realizadas em uma tabela Delta Lake desde a sua criação.

## Para que Serve o Log de Transações? 

###Fonte Única de Verdade: O Delta Lake é construído sobre o Apache Spark para permitir que múltiplos leitores e gravadores de uma tabela trabalhem ao mesmo tempo. Para mostrar aos usuários visões corretas dos dados, o log de transações serve como uma *fonte única de verdade* - o repositório central que rastreia todas as mudanças feitas pelos usuários na tabela.

Quando um usuário lê uma tabela Delta Lake pela primeira vez ou executa uma nova consulta em uma tabela aberta que foi modificada, o Spark verifica o log de transações para ver quais novas transações foram realizadas e, em seguida, atualiza a tabela do usuário com essas alterações. Isso garante que a versão da tabela esteja sempre sincronizada com o registro principal, e que os usuários não possam fazer mudanças conflitantes.

## Implementação da Atomicidade no Delta Lake 

Uma das quatro propriedades das transações ACID, a atomicidade, garante que operações (como um INSERT ou UPDATE) realizadas no data lake sejam concluídas totalmente ou não concluídas. Sem essa propriedade, falhas de hardware ou bugs de software podem resultar em dados parcialmente gravados, causando dados corrompidos.

**O log de transações é o mecanismo pelo qual o Delta Lake oferece a garantia de atomicidade.** 

Se não está registrado no log de transações, nunca aconteceu. Apenas transações concluídas são registradas, o que permite que os usuários confiem na integridade dos dados, mesmo em escala de petabytes.

## Como Funciona o Log de Transações? 

### Dividindo Transações em Commits Atômicos 

Sempre que um usuário modifica uma tabela (como um INSERT, UPDATE ou DELETE), o Delta Lake divide essa operação em uma série de etapas discretas compostas por uma ou mais das seguintes ações:

- **Adicionar arquivo** - adiciona um arquivo de dados.
- **Remover arquivo** - remove um arquivo de dados.
- **Atualizar metadados** - atualiza os metadados da tabela (como nome, esquema ou - particionamento).
- **Definir transação** - registra que um trabalho de streaming estruturado cometeu um micro-batch com o ID fornecido.
- **Alterar protocolo** - habilita novos recursos ao mudar o log de transações para o protocolo mais recente.
- **Informações de commit** - contém informações sobre o commit, como operação realizada, de onde e em que momento.

Essas ações são então registradas no log de transações como unidades atômicas ordenadas, conhecidas como *commits*.

Por exemplo, suponha que um usuário crie uma transação para adicionar uma nova coluna a uma tabela e, ao mesmo tempo, insira mais dados. O Delta Lake dividiria essa transação em suas partes componentes e, uma vez que a transação fosse concluída, as adicionaria ao log de transações como os seguintes *commits*:

- Atualizar metadados - alterar o esquema para incluir a nova coluna
- Adicionar arquivo - para cada novo arquivo adicionado.