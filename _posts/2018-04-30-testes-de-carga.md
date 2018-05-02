---
published: false
---
Próximo da data de um lançamento de uma aplicação, um cliente solicitou que queria fazer um teste para garantir que o sistema pudesse suportar 10 mil usuários simultâneos. 

Como não tinhamos esperiência nestes tipos de teste, surgiram diversas dúvidas. 10 mil usuários seriam 10 mil request por segundo? Como medir isso? Sabiamos da existência de frameworks que poderiam nos ajudar, como por exemplo o jmeter, mas, como gerar o tráfego esperado?

Bem o que é um teste de carga? Este questionamento é algo muito comum e é necessário entender o seu funcionamento para fazer simulações mais realistas com um sistema que irá para a produção.

Para nós, ao executarmos este tipo de teste, foi fundamental para encontrar problemas de configuração, que somente detectaríamos quando o aplicativo já estivesse em produção e com muitos usuários insatisfeitos.

Um teste de carga tem como objetivo simular o trabalho que o sistema teria quando estivesse em produção. Neste tipo de teste não testamos apenas um componente isolado mais a pilha toda. Da chamada HTTP ao insert/update no banco de dados.

Para a escrita de um bom teste de carga, é necessário entender alguns conceitos: O que seria os usuários simultâneos, o que é taxa de transferência? 

**Usuários simultâneos**

Digamos que você solicitou aos seus colegas de trabalho que fossem a uma sala de reuniões, nesta sala você os orienta que executem o sistema fazendo uma série de passos dentro da aplicação.



