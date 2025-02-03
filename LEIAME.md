# Princípios da Utilização do System Design  

<div style="text-align: right; margin: 5 auto;">
    <a href='./LEIAME.md'>🇧🇷 Versão em Português</a> | <a href='./README.md'>🇺🇸 English Version</a>
</div>


System Design (SD) é a base para criar **sistemas escaláveis, robustos e distribuídos**. Um design bem estruturado ajuda a evitar gargalos, otimizar o uso de recursos e preparar o sistema para suportar um grande volume de usuários e dados. Além disso, facilita a colaboração entre times de desenvolvimento, reduz custos operacionais e evita retrabalho ao antecipar possíveis desafios técnicos. Investir tempo no planejamento da arquitetura antes de codificar é essencial para criar sistemas eficientes, resilientes e preparados para o crescimento.  

Um dos principais erros ao lidar com problemas de desempenho e escalabilidade é acreditar que basta **adicionar ou trocar bibliotecas no frontend ou backend** para solucioná-los. Muitas vezes, as dificuldades enfrentadas estão relacionadas à **arquitetura e ao comportamento geral do sistema**, e não à tecnologia específica utilizada no desenvolvimento. No contexto de SD, a **linguagem de programação ou o framework** não são o foco principal; o que realmente importa é garantir um **comportamento previsível, modular e resiliente**, onde os componentes do sistema possam se comunicar de maneira eficiente e suportar o crescimento sem comprometer a experiência do usuário. 


![Image title](/images/system-design.jpg)

Para ajuda a comprender o impacto, construímos este guia com os principais tópicos, além de um estudo de caso para aplicar os conceitos de forma prática. Vamos dividir o estudo em:  

- Os [`Componentes`](#principais-componentes) do SD, abordando a parte mais teórica.  
- Os [`Princípios Fundamentais`](#principios-fundamentais-de-system-design), que sustentam as boas práticas de System Design.  
- Os [`Principais Dilemas`](#principais-dilemas) enfrentados na construção de uma solução.  
- Uma seção dedicada a entender como os [`DevOps podem auxiliar`](#como-devops-pode-ajudar-no-system-design), contribuindo para que engenheiros de software resolvam problemas de forma eficaz.  


Neste guia, utilizaremos o [`Caso de Estudo do Twitter`](#caso-de-estudo) para ilustrar os conceitos apresentados. Ao final, você terá uma compreensão clara do [`Resultado do Caso de Estudo`](#resultado-do-caso-de-estudo), analisando como o Twitter foi construído e como os principais conceitos de SD foram aplicados para evitar problemas de escalabilidade, disponibilidade e consistência.  

Durante cada tópico do SD, faremos algumas anotações seguindo o padrão abaixo:  


- 📖 **Caso de Uso**: aplicação prática do componente/conceito/dilema no desenvolvimento do sistema do Twitter.  
- 🙋🏽‍♂️ **Responda**: questionamentos para você refletir e responder, ajudando na fixação do conhecimento ou incentivando a busca por mais informações.  
- 🪧 **Dica**: sugestões para auxiliar na compreensão do conceito ou evitar armadilhas comuns.  
- 🚨 **Atenção**: alertas para evitar erros críticos ou grandes problemas.  

Antes de avançarmos para o caso de estudo, é importante ressaltar que, no contexto de System Design, todas as decisões envolvem um **trade-off**. Você pode encontrar diferentes soluções para um mesmo problema, pois, no fim das contas... *"Não existe bala de prata!"* 

---

## **Caso de Estudo**

Para facilitar a compreensão dos conceitos, utilizaremos o caso do X (sim, o antigo Twitter), um exemplo clássico de sistema distribuído que lida com milhões de usuários e tweets diariamente. Trata-se de um sistema complexo que enfrenta diversos desafios, como escalabilidade, disponibilidade, consistência e tolerância a falhas. Além disso, precisa lidar com a distribuição geográfica de conteúdos, incluindo imagens, vídeos e textos.  

### **1. Requisitos**

Para iniciar, algumas questões precisam ser respondidas para compreender o propósito do sistema, seu escopo e o comportamento esperado. No desenvolvimento e gestão de software, há várias formas de identificar os requisitos necessários para a entrega de valor esperada pelo sistema. Aqui, adotaremos a divisão clássica em duas categorias de requisitos:  

- **Requisitos Funcionais**: Definem *o que* o sistema deve fazer, ou seja, quais problemas e necessidades devem ser atendidos e resolvidos pelo software por meio de funções ou serviços.  

- **Requisitos Não Funcionais**: Definem *como* o sistema deve se comportar. Enquanto os requisitos funcionais focam no *o que será feito*, os não funcionais descrevem *como será feito*.

#### 🙋🏽‍♂️ **Responda**

1. Quais requisitos funcionais e não funcionais você adicionaria ao sistema do Twitter?

2. O sistema deverá ser capaz de lidar com questões de distribuição geográficas e de idiomas?


#### 📖  "Caso de Estudo"
    
Para o nosso caso de estudo do tweet, vamos considerar os seguintes requisitos funcionais e não funcionais:

**Requisitos Funcionais**

- Postar um tweet
- Seguir um usuário
- Visualizar o feed de tweets
- Retweetar um tweet
- Pesquisar por tweets
- Receber notificações

**Requisitos Não Funcionais**

- **Escalabilidade**: O sistema deve ser capaz de lidar com um grande número de usuários e tweets.
- **Disponibilidade**: O sistema deve estar disponível 24/7.
- **Consistência**: O sistema deve garantir que os dados estejam atualizados, mas não necessariamente em tempo real.
- **Tolerância a falhas**: O sistema deve ser capaz de lidar com falhas de hardware e software.

## **Principais Componentes**

Agora que entendemos os requisitos do sistema, vamos abordar os principais conceitos de System Design, essenciais para criar sistemas escaláveis e eficientes.  

### **1. Scaling** 

![Image title](/images/system-design-horizontal-vs-vertical.png)

O dimensionamento (*scaling*) é a capacidade de um sistema de lidar com um aumento na carga de trabalho. Como exemplo, pense em como o Twitter processa milhões de requisições, lidando com picos regionais e temporais, além da possibilidade de um aumento repentino no número de novos usuários. Existem duas abordagens principais para dimensionar um sistema e atender a essas demandas:  

- **Horizontal Scaling** (*Escalabilidade Horizontal*): Adicionar mais servidores para dividir a carga, ou seja, réplicas.  
    * Alocar servidores na infraestrutura para distribuir a carga.  
    * Mais fácil de escalar e geralmente mais econômico.  
    * Requer um sistema distribuído e pode ser complexo ao adicionar [load balancers](#3-load-balancing) e [caches](#2-caching).  

- **Vertical Scaling** (*Escalabilidade Vertical*): Aumentar os recursos (CPU, memória) de um único servidor.  
    * Adicionar mais CPU, memória ou armazenamento a um único servidor.  
    * Mais fácil de implementar, mas tem limitações.  
    * Pode ser mais caro e menos flexível.  

Em muitos casos, a escolha do modelo de escalabilidade depende não apenas dos requisitos do sistema, mas também dos recursos financeiros disponíveis para o projeto. Projetos iniciais, como POCs (*Proof of Concepts*), tendem a utilizar a escalabilidade horizontal devido ao seu custo-benefício.  

#### 📖 "Responda"

1. Como sua empresa lida com o aumento da carga de trabalho em um sistema? 

2. Como é a estrutura de dimensionamento do seu sistema atua

#### 🪧 "Dica"

Algumas soluções de infraestrutura permitem um uso mais eficiente do dimensionamento horizontal, como [Docker Swarm](#3-load-balancing) ou uma abordagem híbrida com [Kubernetes](https://www.geeksforgeeks.org/kubernetes-horizontal-vs-vertical-scaling/).  

Algumas soluções de infraestrutura permitem que você trabalhe de forma mais eficiente com o dimensionamento horizontal como o uso de [Docker Swarm](#3-load-balancing) ou uma abordagem mista como o [Kubernetes](https://www.geeksforgeeks.org/kubernetes-horizontal-vs-vertical-scaling/).
    
#### 📖 **Caso de Uso**

No caso do Twitter, utilizamos os conceitos de escalabilidade vertical e horizontal para dimensionar o sistema. O banco de dados e os servidores de aplicativos são escalados de maneira diferente:  

- O **banco de dados** é escalado **verticalmente** para aumentar a capacidade de armazenamento e realizar a [replicação](#411-replication).  
- Os **servidores de aplicativos** são escalados **horizontalmente** para lidar com o tráfego de usuários.  

**Obs**: Vamos discutir melhor sobre o banco e as APIs nos demais tópicos.

----

### **2. Caching**  

![Image title](/images/system-design-cache.jpg)

Você já se deparou com requisições sendo executadas no seu back-end diversas vezes para obter a mesma informação dentro de um curto período de tempo? Esse tipo de requisição é um forte candidato ao uso de um sistema de cache, permitindo uma entrega mais rápida e reduzindo o esforço da sua API, do back-end e do banco de dados. No entanto, a escolha da estratégia de cache depende de diversos fatores intrinsecamente ligados à sua aplicação, como o tamanho e a volatilidade dos dados, a frequência de acesso, a capacidade de armazenamento e as metas de desempenho. Implementações bem planejadas podem melhorar significativamente a escalabilidade, reduzir o consumo de recursos e proporcionar uma melhor experiência ao usuário.  

O cache é apropriado para:  

- [x] Armazenar dados temporariamente para melhorar o desempenho.  
- [x] Reduzir a latência entre o cliente e o servidor. 

Existem diversas soluções no mercado, como `Redis` e `Memcached`, que podem ser utilizadas. Porém, mais importante do que escolher a ferramenta certa é **compreender** a política de permanência do cache e o modo de escrita que será adotado na sua aplicação.  

#### 2.1 Políticas LRU/LFU

Em um mundo ideal, poderíamos armazenar em cache todos os dados e manter a menor latência possível na aplicação e no banco de dados. No entanto, sempre há limitações de recursos, e precisamos fazer escolhas. Quando falamos de **LRU/LFU**, estamos nos referindo às estratégias para decidir quais dados manter em cache quando o espaço disponível se esgota:  

- **Least Recently Used (LRU)**: Remove as chaves mais antigas para dar espaço as novas chaves e dados.
- **Least Frequently Used (LFU)**: Remove as chaves menos usadas recentemente para dar espaço aos novos dados.


#### 2.2 Cache Write-through/Write-behind

Agora que entendemos como os dados permanecem em cache, precisamos discutir como eles são persistidos e qual a estratégia de escrita. Pense no cenário da inserção de tweets: certamente é mais importante que um tweet seja salvo no banco de dados do que no cache, enquanto a leitura do feed de um usuário pode tolerar a ausência de alguns tweets temporariamente.  .

- **Write-through**: Os dados são escritos no cache e na fonte original ao mesmo tempo.
- **Write-behind**: Os dados são escritos no cache primeiro e depois na fonte original.
- **Refresh-ahead**: Os dados são atualizados no cache antes de expirar.

A escolha da política de escrita depende do seu sistema e da sua aplicação. No entanto, é importante ter em mente que o **Write-through** é mais seguro, mas pode ser mais lento, enquanto o **Write-behind** é mais rápido, porém menos seguro. Como você já percebeu, o principal *trade-off* aqui é a **importância da consistência na escrita dos dados**, ou seja, se a consistência é crítica para a aplicação.  


###### 🚨 **Atenção**

Quando as políticas de cache e escrita não são implementadas corretamente, sua aplicação pode sofrer com alta latência e inconsistência dos dados! Seja cuidadoso ao definir essas estratégias.  

##### 📖 **Caso de Uso**

No caso do Twitter, utilizamos cache para armazenar tweets recentes, feeds de usuários e tendências, melhorando o desempenho e reduzindo a latência. A estratégia adotada é a seguinte:  

- Para o [`Timeline Service`](#33-timeline-service), responsável pela criação da timeline dentro da [`Read API`](#22-read-api), utilizamos a política **LFU**, priorizando em cache os tweets mais frequentemente acessados pelos usuários.  

- Para a geração do feed de um usuário específico via [`Timeline Service`](#33-timeline-service), optamos pela política **Write-behind**, pois a consistência (caso um tweet não apareça imediatamente) não é tão crítica.  

- Para a criação de tweets, por meio do [`Fan Out Service`](#34-fan-out-service), dentro da [`Write API`](#23-write-api), adotamos a política **Write-through**, garantindo a consistência dos tweets no banco de dados, mesmo que isso resulte em um tempo de resposta maior.  

- Já para serviços derivados, como o [`User Graph Service`](#35-user-graph-service) e o [`Search Service`](#36-search-api), utilizamos a política **Write-behind**, pois a consistência dos dados nesses serviços não é tão crítica.  
    
----

### **3. Load Balancing**  

![Image title](/images/system-design-loader-balance.jpg)

Os balanceadores de carga distribuem solicitações de clientes para recursos de computação, como servidores de aplicativos, APIs e bancos de dados. Em cada caso, o balanceador de carga retorna a resposta do recurso de computação para o cliente solicitante. 

Os balanceadores de carga são eficazes em:

- [x] Ajudar a eliminar um [único ponto de falha](#1-failovers-and-spof).  
- [x] Evitar que solicitações sejam enviadas para servidores não saudáveis.  
- [x] Evitar a sobrecarga de recursos como bancos de dados e conteúdo estático. 

A distribuição de requisições entre vários servidores para garantir alta disponibilidade e desempenho é aplicada em sistemas distribuídos horizontalmente. No entanto, essa abordagem introduz algumas complexidades, como a necessidade de clonagem de servidores. Tenha em mente que:  

- Os servidores [`stateful`](#3-stateful-vs-stateless) são mais difíceis de escalar, pois exigem custos adicionais para gerenciamento de endereços e sincronização de estado.  
- As sessões devem ser armazenadas em um banco de dados e/ou um cache persistente para garantir a consistência dos dados.  

#### **3.1 Algoritmos de Load Balancing**

![Image title](/images/system-design-loader-balance-algoritmos.gif)

Antes de falar dos algoritmos de balanceamento, já refletiu sobre como pode ser complexo distribuir a carga entre os servidores? Esse problema é mais difícil do que parece. Imagine um cenário em que você tem dois servidores, um terceiro é adicionado e o primeiro deixa de responder. Como você distribui as requisições?  

Para entender melhor esse problema, recomendo que leia primeiro o conceito de [Consistent Hashing](#2-consistent-hashing) na seção 6, pois ele ajuda a manter a consistência na distribuição de carga.  

Agora, voltando à questão de distribuição, existem diversos algoritmos de balanceamento de carga, cada um com suas vantagens e desvantagens. Alguns dos mais comuns são:  

- **Random**: Distribui solicitações aleatoriamente entre os servidores saudáveis, porém não garante a distribuição uniforme;
- **Least loaded**: Distribui solicitações para o servidor com a menor carga, porém pode levar a sobrecarga de servidores mais rápidos;
- **Session/cookies-based**: Distribui solicitações com base em sessões ou cookies, porém pode levar a problemas de escalabilidade e consistência.
- **Round robin e weighted round robin**: Distribui solicitações em uma ordem circular de uniforme ou não (quando atribuído por peso);
- **Least connections**: Distribui solicitações para o servidor com o menor número de conexões, porém pode levar a sobrecarga de servidores mais rápidos.

Perceba que a escolha do algoritmo influencia diretamente a distribuição da carga. Existem diversas soluções de mercado para implementar *load balancing*, como `Nginx`, `HAProxy`, `AWS ELB`, `Google Cloud Load Balancer`, `Azure Load Balancer`, entre outras.  
    
O Load Balancer distribui a carga de trabalho entre vários servidores. Vamos compará-lo com algumas soluções populares:  

**Load Balancer vs. Docker Swarm**  
O *load balancing* (balanceamento de carga) é uma técnica que distribui a carga de trabalho entre servidores, enquanto o Docker Swarm é uma solução de orquestração de contêineres que permite criar e gerenciar clusters de contêineres.  

**Load Balancer vs. Kubernetes**  
O Kubernetes é uma plataforma de orquestração de contêineres de código aberto que automatiza a implantação, o dimensionamento e a operação de aplicativos em contêineres.  



##### 🙋🏽‍♂️ **Responda**  

Como você resolveria o problema de *load balancing* com a configuração abaixo?  

- Atualmente, há 4 servidores saudáveis operando: [`s1`, `s2`, `s3`, `s4`]
- O usuário `u1` foi o último a fazer uma requisição e anteriormente foi direcionado para o servidor `s3`.  
- No próximo segundo (`t1`), `u1` fará uma nova requisição.  
- Um novo usuário, `u2`, que ainda não foi distribuído, fará sua primeira requisição no segundo seguinte (`t2`).  
- No terceiro segundo (`t3`), `u1` fará uma segunda requisição.  


Com base nesse cenário, responda às seguintes questões considerando diferentes algoritmos de *load balancing*:  


1. Para qual servidor o usuário `u1` será direcionado em cada algoritmo de *load balancing*?  

2. Para qual servidor `u2` será alocado em cada algoritmo?  

3. Se o servidor `s3` estiver indisponível, para qual servidor a requisição `t3` do usuário `u1` será alocada?  

4. Se um novo servidor `s5` for adicionado entre os segundos `t2` e `t3`, como isso afetaria a alocação da requisição `t3` do usuário `u1` em cada algoritmo?

    
#### 📖 **Caso de Uso**
    
As requisições dos usuários (*Client*) serão intermediadas pelo [`Load Balancer`](#12-load-balancer), que distribuirá as requisições entre os [`Web Servers`](#21-web-server) saudáveis da aplicação. Utilizaremos o algoritmo *Round Robin* para o nosso serviço de [`Load Balancer`](#12-load-balancer). Isso ajudará a distribuir a carga de forma uniforme e a evitar sobrecarga em servidores individuais. 

----

### **4. Database**

Os bancos de dados são a base de muitas aplicações, e a escolha do banco de dados certo é fundamental para o desempenho, escalabilidade e disponibilidade do sistema. Em geral, os bancos de dados são divididos em dois tipos principais:  

- **Relational Database**: Armazena dados de forma estruturada, organizados em forma de table com linhas e colunas, com suporte a transações ACID.

- **NoSQL Database**: Armazena dados em documentos, colunas ou pares de chave-valor e gráfos com foco em escalabilidade, disponibilidade e particionamento, porém não focam na garantia do ACID.

#### **4.1 Relational Database**

Os bancos de dados relacionais são fundamentados no conceito de **ACID**, garantindo confiabilidade nas transações:  

- **Atomicity**: Todas as operações de uma transação são executadas com sucesso ou nenhuma delas é aplicada.  
- **Consistency**: Qualquer transação leva o banco de dados de um estado válido para outro, sem violações de regras de integridade.  
- **Isolation**: As transações são isoladas umas das outras até serem concluídas, evitando efeitos colaterais.  
- **Durability**: Após a confirmação (*commit*), as mudanças são permanentes, mesmo em caso de falha do sistema.  

##### **4.1.1 Replication**

Para evitar problemas de escalabilidade e disponibilidade, uma estratégia comum é a **replicação de dados**, aumentando a tolerância a falhas e a capacidade de leitura. Algumas das principais abordagens incluem:  

- **Master-Slave Replication**: Uma cópia única dos dados é replicada para vários servidores de leitura, ou seja, há um potencial para perda de dados se o mestre falhar antes que quaisquer dados recém-gravados possam ser replicados para outros os escravos e se houver muitas gravações, as réplicas de leitura podem ficar atoladas com a repetição de gravações e não podem fazer tantas leituras.

- **Master-Master Replication**: Dados são replicados entre dois servidores para leitura e gravação, permitindo a escalabilidade de leitura e tolerância a falhas, porém muitos destes sisteams master-master violam o `ACID` e aumentam a latência nas operações. O ponto crítico é notado na resolução de conflitos quando entra em ação à medida que mais nós de gravação são adicionados e a latência aumenta . 

- **Sharding**: O sharding distribui dados entre diferentes bancos de dados, de modo que cada banco de dados só pode gerenciar um subconjunto dos dados. Tomando um banco de dados de usuários como exemplo, conforme o número de usuários aumenta, mais shards são adicionados ao cluster.

Observe que ambas estratégias são usadas para aumentar a disponibilidade e a tolerância a falhas, mas cada uma tem suas vantagens e desvantagens na aplicabilidade de um projeto.


###### 🙋🏽‍♂️ **Responda**

Como você lidaria com um cenário envolvendo milhões de usuários e seus seguidores? Como modelaria essa relação em um banco de dados relacional?  

#### **4.2 NoSQL Database**

Os bancos de dados NoSQL são uma alternativa aos bancos de dados relacionais e são projetados para atender a requisitos que envolvem **dados desnormalizados e agrupamento de informações em um único documento**. De forma geral, os NoSQL são descritos como bancos de dados **não normalizados**, onde o agrupamento de dados é feito pela aplicação, priorizando **disponibilidade e particionamento**.  


- **Key-Value Stores**: Armazena dados em pares de chave-valor.
- **Document Stores**: Armazena dados em documentos semelhantes a JSON.
- **Column-Family Stores**: Armazena dados em colunas em vez de linhas.


##### 📖 **Caso de Uso**

Usaremos um banco de dados relacional com replicação, chamado [`SQL Write Master-Slave`](#43-sql-write-master-slave), para aumentar a disponibilidade e a tolerância a falhas. Utilizaremos a estratégia de **replicação mestre-escravo** para distribuir os dados de forma eficiente.  

Além disso, empregaremos um banco de dados NoSQL chamado [`NoSQL Database`](#42-nosql-database) para armazenar **relações de grafos de relacionamento**, facilitando a busca de usuários e seguidores.  
    
----

### **5. Content Delivery Networks (CDN)**  


![Image title](/images/system-design-cdn.jpg)

Como entregar o meu conteúdo de forma rápida para um usuário na China, sabendo que meu servidor está em São Paulo? Esse questionamento norteia a existência das **CDNs (Content Delivery Networks)**, redes de servidores distribuídos geograficamente que aceleram a entrega de conteúdo estático aos usuários.  

O uso de CDNs exige a alteração das aplicações para considerar o contexto do usuário, garantindo que as URLs de conteúdo estático apontem para um CDN específico.  

O uso de CDNs promove:

- [x] Menor latência entre a solicitação do usuário e o atendimento
- [x] Redução do tráfego e custo de rede
- [x] Os servidores de API não precisam atender a solicitações que o CDN atende como conteúdos estáticos (imagens, vídeos, etc)


#### **5.1 CDN Push/Pull**

Os CDNs podem ser configurados para operar em dois modos principais:

- **Push CDN**: O conteúdo é enviado para o CDN e armazenado em cache antes que seja solicitado.
- **Pull CDN**: O conteúdo é solicitado ao CDN e armazenado em cache apenas quando é solicitado.


##### 🪧 **Dica**

Os custos de CDN podem ser significativos, dependendo do volume de tráfego. No entanto, devem ser comparados aos custos adicionais que você incorreria ao **não utilizar** um CDN, como maior consumo de banda e sobrecarga nos servidores.  

##### 🚨 **Atenção**

A configuração do **TTL (Time to Live)** no CDN é essencial para evitar a distribuição de conteúdos desatualizados. Um TTL muito alto pode resultar na exibição de versões antigas do conteúdo, enquanto um TTL muito baixo pode aumentar a carga nos servidores de origem.

##### 📖 **Caso de Uso**

Utilizaremos a estratégia de CDN para armazenar **imagens de perfil de usuário, vídeos e conteúdos estáticos**, replicando-os regionalmente para melhorar o desempenho e a disponibilidade. Dessa forma, o mesmo conteúdo poderá ser consumido por milhares de usuários em diferentes partes do mundo com **menor latência e maior eficiência**.  

## **Princípios Fundamentais de System Design**

### **1. Failovers and SPOF**  

Failover é um mecanismo de redundância que garante a continuidade dos serviços em caso de falhas em sistemas ou componentes de software. Esse conceito é fundamental em ambientes críticos, onde a disponibilidade e a confiabilidade são essenciais.  


- [x] É necessária a garantia de continuidade do sistema em caso de falhas.  
- [x] Inclui troca automática para backups ou réplicas ativas.

Algumas estratégias comuns de failover já foram abordadas em tópicos anteriores, como:  

- [**Loading Balancing**](#3-load-balancing): Distribuição de carga entre servidores para garantir alta disponibilidade e desempenho.
- [**Database Replication**](#411-replication): Duplicação de dados para aumentar a disponibilidade e a tolerância a falhas.

#### **1.1 Single Point of Failure (SPOF)**

![Image title](/images/system-design-SPOF.jpg)

A discussão sobre failover começa com a identificação dos pontos críticos de falha do sistema. O **Single Point of Failure (SPOF)** refere-se a qualquer componente cuja falha possa comprometer todo o sistema.


##### 🙋🏽‍♂️ **Responda**

1. Quais são os possíveis pontos de falha que você identificou no seu sistema atual?

2. Como você lidaria com um SPOF em um sistema distribuído?

##### 📖 **Caso de Uso**

Para evitar falhas, utilizaremos **replicação de banco de dados** para garantir a disponibilidade dos dados e **balanceamento de carga** para distribuir requisições entre servidores de aplicativos. Essa configuração deve levar em conta custos e disponibilidade geográfica.  

----

### **2. Consistent Hashing**  

![Image title](/images/system-design-consistent-hashing.png)

Consistent Hashing é uma técnica eficiente para distribuir dados entre servidores, minimizando redistribuições quando há mudanças na estrutura do cluster. Compreender esse conceito é essencial para entender como os servidores são alocados e como a carga de trabalho é distribuída.  

📖 Para aprofundamento, leia [The Simple Magic of Consistent Hashing](https://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html).  


----

### **3. Consistency patterns**

Consistência refere-se à garantia de que todos os nós de um sistema possuem os mesmos dados. Existem vários padrões para sincronizar os dados e garantir que os clientes tenham uma visão consistente:  

- **Eventual Consistency**: Após uma gravação, as leituras eventualmente o verão (normalmente em milissegundos). Os dados são replicados de forma assíncrona.
- **Strong Consistency**: Após uma gravação, os leitores o verão. Os dados são replicados de forma síncrona. 
- **Weak consistency**: Após uma gravação, os leitores podem ou não vê-la. Uma abordagem de melhor esforço é adotada.

#### 📖 **Caso de Uso**

Para o serviço de [`Timeline Service`](#33-timeline-service), utilizaremos **consistência eventual**, permitindo que o feed de tweets seja construído e armazenado em réplicas de banco.  

Para o serviço [`Fan Out Service`](#34-fan-out-service), aplicaremos **consistência forte**, garantindo que os tweets sejam imediatamente persistidos no banco de dados.  

----

### **4. Availability patterns**

Disponibilidade é a garantia de que um sistema está sempre disponível para os usuários. Existem vários padrões de disponibilidade que podem ser usados para garantir a disponibilidade do sistema:

- **Active-Passive**: Um servidor ativo e um servidor passivo. O servidor passivo assume quando o servidor ativo falha.

- **Active-Active**: Ambos os servidores estão ativos e podem atender solicitações. Se um falhar, o outro assume.

Para banco de dados o conceito de disponibilidade é demonstrada por meio do item [`replication`](#411-replication).

----

### **5. Asynchronism and Queues**

Fluxos assíncronos ajudam a reduzir o tempo de resposta de operações complexas que, de outra forma, seriam executadas de forma síncrona. **Filas (queues)** são utilizadas para armazenar e processar solicitações em momentos oportunos, garantindo escalabilidade e confiabilidade.  

#### **5.1 Message and Task queues**

As filas de mensagens são usadas para armazenar mensagens de forma temporária até que possam ser processadas. As mensagens são enviadas para a fila e processadas por um ou mais consumidores. As filas de mensagens são úteis para:

- **Decoupling**: Desacoplar componentes do sistema.
- **Scalability**: Lidar com picos de tráfego.
- **Reliability**: Garantir que as mensagens sejam processadas.

Já as filas de tarefas são usadas para armazenar tarefas que precisam ser executadas em um momento posterior porém o princípio é o mesmo das filas de mensagens.

#### **5.2 Back pressure**

Se uma fila crescer excessivamente, pode consumir muita memória, causando falhas de cache, leituras frequentes no disco e degradando o desempenho. Estratégias de **back pressure** ajudam a mitigar esse risco.  

##### 🪧 **Dica**

As filas de mensagens são úteis para desacoplar componentes do sistema como no caso do twitter se você estiver postando um tweet, o tweet pode ser postado instantaneamente na sua linha do tempo, mas pode levar algum tempo até que seu tweet seja realmente entregue a todos os seus seguidores.

##### 📖 **Caso de Uso**

Utilizaremos **message queues** para serviços como [`User Graph Service`](#35-user-graph-service) e [`Search Service`](#37-search-service), garantindo que atualizações (ex.: novos tweets, mudanças de perfil) sejam propagadas corretamente.  

Para o [`Notification Service`](#38-notification-service) e [`Timeline Service`](#33-timeline-service), utilizaremos um modelo **pub/sub** para entrega em tempo real. Isso permite maior escalabilidade e eficiência, aceitando alguma latência.  

## **Principais Dilemas**

### **1. Performance vs scalability**


![Image title](/images/system-design-performance-vs-scalability.png)

Nem sempre é fácil distinguir entre **desempenho** e **escalabilidade**, mas compreender essa diferença é essencial:  

* Se você tem um problema de **desempenho**, seu sistema é lento para um único usuário.
* Se você tem um problema de **escalabilidade**, seu sistema é rápido para um único usuário, mas lento sob carga pesada.

Geralmente você deve buscar o máximo de escalabilidade com desempenho aceitável.

### **2. Latency vs throughput**

![Image title](/images/system-design-latency-throughput-bandwidth.png)

Outro dilema comum é a latência versus o rendimento, onde a latência é o tempo que leva para uma solicitação ser processada e o rendimento é o número de solicitações que um sistema pode processar em um determinado período de tempo.

* **Latência**: O tempo que leva para uma solicitação ser processada.
* **Throughput**: O número de solicitações que um sistema pode processar em um determinado período de tempo.

Geralmente, você deve buscar o máximo de rendimento (Throughput) com latência (Latency) aceitável.

----

### **2. Availability vs consistency**


Pense no problema entre a disponibilidade e a consistência, onde a disponibilidade é a garantia de que um sistema distribuído está sempre disponível para os usuários e a consistência é a garantia de que todos os nós de um sistema de banco de dados possuem os mesmos dados ao mesmo tempo. Como garantir isso? Este é um problema conhecido como [CAP Theorem](#21-cap-theorem).

#### **2.1 CAP Theorem**  

![Image title](/images/system-design-cap-teorm.png)

O teorema CAP diz que um sistema distribuído pode fornecer apenas duas de três características desejadas: consistência, disponibilidade e tolerância a partições (o 'C', 'A' e 'P' no CAP).

Teorema que descreve o equilíbrio entre:  

1. **Consistência (Consistency)**: Cada solicitação recebe a resposta mais recente, que pode não refletir as alterações de outras solicitações. 

2. **Disponibilidade (Availability)**: Cada solicitação recebe uma resposta sobre se foi bem-sucedida ou falhou.

3. **Tolerância a Partições (Partition Tolerance)**:  O sistema continua a funcionar mesmo quando um ou mais componentes falham.


----

### **3. Stateful vs Stateless**

![Image title](/images/system-design-stateful-vs-stateless.jpg)

A diferença entre **stateful** e **stateless** é fundamental para arquiteturas modernas, como microsserviços e sistemas distribuídos. Um caracterisca fundamental de diferenciação é a capacidade de armazenar informações sobre o estado atual das interações do usuário, ou seja, é a capacidade de armazena informações sobre o estado atual das interações do usuário enquanto o outro trata cada solicitação como uma transação independente e isolada.


-  **stateful** permitem que os usuários armazenem, gravem e consultem pela Internet procedimentos e informações já estabelecidas. O servidor monitora o estado de cada sessão do usuário e armazena as informações sobre solicitações antigas e interações

- **stateless** não retêm informações sobre interações anteriores do usuário. Não há nenhum conhecimento ou referência armazenados sobre transações anteriores.

A maioria das aplicações que utilizamos no dia a dia é stateful, no entanto, o avanço de outras tecnologias, como microsserviços e containers, vem tornando mais fácil construir e implantar aplicações stateless. Compreender o ecopo da aplicação, se ela é `stateful` ou `stateless` é fundamental para compreender a viabilidade da estratégia no [load balancing](#3-load-balancing) que precisamos. 

#### 🪧 **Dica**

Em aplicações stateful tende a usar os mesmos servidores toda vez que processam uma solicitação de um usuário para evitar inconsistência de cache distribuído.


#### 📖 **Caso de Uso**

Adotaremos um modelo híbrido:  
**Stateful** para armazenar informações críticas, como perfis de usuário e tweets.  
**Stateless** para serviços de leitura e recuperação de dados públicos

----

### **4. SQL vs NoSQL**

O dilema de qual banco de dados usar é uma questão comum em SD. Aqui estão algumas considerações para usar uma ou a outra opção:

- Motivos para SQL:

    - [x] Dados são considerados estruturados e um esquema estrito
    - [x] Há um alto relacionamento entre os dados e necessidade de junções complexas
    - [x] Transações com a estrura do [ACID](#411-replication) são necessárias

- Motivos para NoSQL:
    - [x] Dados semiestruturados com informaçãos dinâmico ou flexível
    - [x] A relação entre os dados não é tão importante e possivelmente não há necessidade de junções complexas
    - [x] O armazena será de muitos TB (ou PB) de dados


#### 📖 **Caso de Uso**
    Vamos utilizar uma abordagem híbrida, onde usaremos um banco de dados relacional para armazenar dados estruturados, como informações de usuários e tweets, e um banco de dados NoSQL para armazenar dados semiestruturados, como feeds de usuários e tendências.

----

### **5. APIs (REST/gRPC/GraphQL)**  

![Image title](/images/system-design-REST-gRPC-GraphQL.png)


Aa escolha do serviço de entrega de dados ou comunicação entre serviços é fundamental para o desempenho e a escalabilidade do sistema. Porém a escolha do tipo de API depende do tipo de aplicação e dos requisitos de desempenho. Aqui estão algumas considerações para escolher entre diferentes tipos de APIs:


- **REST**: Modelo amplamente utilizada, porém a estrutura de dados é rígida e pode ser ineficiente para comunicação entre serviços.
- **gRPC**: Usa Protobufs para comunicação eficiente e de baixa latência entre serviços, porém deverá ser usado apenas em ambientes internos.
- **GraphQL**: Uma alternativa ao REST que permite aos clientes solicitar apenas os dados necessários, porém poderá ser mais complexo de implementar.

#### 📖 **Caso de Uso**

Nesse caso optaremos por utiliza o GraphQL porque é uma linguagem de consulta que permite acessar dados de forma mais eficiente, retornando apenas o desejado. Lemnbrando que o GraphQL é ideal para fontes de dados complexas, inter-relacionadas e grandes.

---

## **Como DevOps pode ajudar no System Design?**


O **DevOps** é uma abordagem que integra **desenvolvimento, operações e segurança** para melhorar a colaboração, automação e eficiência dos sistemas.  

Um erro comum é pensar que o **DevOps se limita à automação de infraestrutura e entra em ação apenas no final do desenvolvimento**. Na realidade, muitas decisões de **arquitetura afetam diretamente o desempenho e a escalabilidade**, tornando essencial o envolvimento do DevOps desde o início do **System Design (SD)**.  


A incorporação do DevOps desde as fases iniciais do SD oferece vários benefícios:  

- **Automatizar a implantação e o gerenciamento de infraestrutura**: Ferramentas como Terraform pode ser usada para automatizar a implantação e o gerenciamento de infraestrutura, impactando diretamente na escalabilidade e disponibilidade do sistema.

- **Monitorar e otimizar o desempenho do sistema**: Ferramentas de monitoramento como Prometheus e Grafana podem ser usadas para monitorar o desempenho e identificar gargalos de desempenho auxiliando no aperfeiçoamento do sistema.

- **Garantir a segurança e a conformidade**: Ferramentas de segurança como Vault e ferramentas de conformidade como Chef InSpec podem ser usadas para garantir a segurança e a conformidade do sistema.


O DevOps não é apenas sobre **automação**, mas sim sobre **integração, monitoramento e segurança contínua**. Quando incorporado ao **System Design**, ele melhora a **escalabilidade, confiabilidade e eficiência** do sistema, reduzindo custos e prevenindo problemas críticos antes que eles aconteçam. 🚀  


## **Resultado do Caso de Estudo**

Após explorarmos os principais componentes, teorias e dilemas do **System Design (SD)**, chegamos à nossa **arquitetura escalável para o Twitter**. O foco está na arquitetura e nas interações entre os componentes do sistema, e não nos detalhes específicos de implementação do frontend ou backend. Isso ocorre porque o SD visa garantir escalabilidade, desempenho, disponibilidade e confiabilidade, definindo como os componentes se comunicam, armazenam e processam dados de forma eficiente. Detalhes como linguagens de programação, frameworks ou lógica interna de cada serviço são decisões que podem variar conforme a equipe e os requisitos do projeto.

Abaixo, apresentamos um diagrama que ilustra o processo de **System Design** e o resultado final da nossa implementação:  

![System Design](/images/system-design-result.png)
[Abri o Diagrama do System Design](/images/system-design-result.png)

Agora, vamos resumir como a nossa arquitetura está estruturada em **camadas e serviços especializados** para garantir **desempenho, disponibilidade e escalabilidade**.  


### **Layers da Aplicação**

Nossa aplicação está dividida em **quatro camadas principais**, cada uma com serviços especializados:  

#### 1. Camada de Entrada

##### 1.1 **DNS & CDN:** 

Adotamos **CDNs** para melhorar a distribuição de conteúdo, como imagens, vídeos e áudios, reduzindo a latência globalmente.  

##### 1.2 **Load Balancer** 
Distribui as requisições entre múltiplos **Web Servers**, garantindo **alta disponibilidade** e **balanceamento de carga**.  

#### 2. Camada de Aplicação

##### **2.1 Web Server** 

Processa as requisições dos clientes e as direciona para as **APIs apropriadas** para leitura e escrita.  

##### **2.2 Read API** 

Responsável por operações de leitura **rápidas e otimizadas**.  

##### **2.3 Write API**

Gerencia operações de escrita, garantindo **consistência e segurança**.  

#### 3. Camada de Serviços Backend

##### **3.1 Tweet Info Service**

Gerencia informações específicas sobre tweets, ou seja, um serviço especializado para armazenar e recuperar informações sobre tweets.

##### **3.2 User Info Service** 
Gerenciam informações específicas sobre o usuário.

##### **3.3 Timeline Service** 
Monta a linha do tempo dos usuários otimizando leituras de tweets em cache.

##### **3.4 Fan Out Service** 
Distribui novos conteúdos para seguidores, otimizando a disseminação de tweets em cache através de um serviço `Pub/Sub`

##### **3.5 User Graph Service** 
Mantém a estrutura de conexões entre usuários como seguidores e seguidos e serviços de similaridade.

##### **3.6 Search API** 
Permitem busca eficiente de tweets e usuários.

##### **3.7 Search Service**
Indexa e pesquisa tweets para fornecer resultados rápidos e precisos.

##### **3.8 Notification Service**
Gerencia notificações para os usuários.

#### 4. Camada de Cache & Armazenamento

##### **4.1 Memory Cache** 
Serviço de cache para a redução da latência ao armazenar dados frequentemente acessados pelos serviços.

##### **4.2 SQL Read Replicas**

##### **4.3. SQL Write Master-Slave** 
Estratégia de replicação para escalabilidade e consistência dos dados.

##### **4.4 Object Store** 

Armazena grandes volumes de mídia, como imagens e vídeos.

### **Atendimento aos Requisitos**

Agora, vamos verificar como a arquitetura atende aos **requisitos funcionais e não funcionais** do Twitter.  

#### Requisitos Funcionais

1. **Postar um tweet**  
    - O **Write API** recebe a requisição e armazena os dados no banco **SQL Write Master-Slave** bem o disparo do serviço Fan Out Service para distribuir o tweet para os seguidores.
    - O **Fan Out Service** é um serviço paralelo de distribuição do novo tweet para os seguidores e serviços como `Pub/Sub`, busca e outros serviços.  
    - O **Memory Cache** atuará para o armazenar tweets recentes e redução do tempo de recuperação.

2. **Seguir um usuário**  
    - O **User Graph Service** gerencia conexões entre usuários e fornecerá a estrutura e inteligência necessária.
    - Quando um usuário segue outro, essa relação é armazenada no banco de dados e refletida no cache.  

3. **Visualizar o feed de tweets**  
    - O **Read API** serviço especializado para a consulta de dados otimizados de leitura.  
    - O **Timeline Service** compõe o feed do usuário, buscando tweets do cache e do banco de dados.  

4. **Retweetar um tweet**  
    - O **Write API** registra a ação no banco de dados sem precisar duplicar o tweet.
    - O **Fan Out Service**, serviço paralelo e distribuído, redistribui o retweet para os seguidores do usuário que retweetou.

5. **Pesquisar por tweets**  
    - A **Search API** processa a consulta e interage com o **Search Service**, que a partir dos indexes dos tweets armazenados, retorna os melhores resultados.  

6. **Receber notificações**  
    - O **Notification Service** gerencia e entrega notificações em tempo real, através do serviço `Pub/Sub`, quando há novos tweets e conteúdos do usuário seguidos pelo usuário.

---

#### Requisitos Não Funcionais

1. **Escalabilidade**  
    - Uso de **Load Balancer** para distribuir carga.  
    - Separação entre APIs de leitura e escrita (`Write API` e `Read API`) para melhor desempenho.  
    - **Memory Cache** acelera acessos frequentes.  
    - **SQL Read Replicas** aumentam capacidade de leitura.  

2. **Disponibilidade**  
    - Uso de múltiplos **Web Servers** e **APIs distribuídas** garante alta disponibilidade.  
    - **CDN** melhora entrega de conteúdo globalmente.  

3. **Consistência**  
    - **SQL Write Master-Slave** assegura integridade dos dados.  
    - O sistema adotar modelo de **eventual consistency** para escalabilidade sem comprometer a experiência do usuário.  

4. **Tolerância a falhas**  
    - **Replicação de Banco de Dados** evita perda de informações.  
    - **Memory Cache** reduz impacto de falhas em serviços principais.  
    - Componentes distribuídos garantem que falhas em um serviço não impactem todo o sistema.  

Nossa arquitetura foi projetada para garantir **eficiência, desempenho e confiabilidade**, atendendo tanto aos requisitos **funcionais** quanto **não funcionais** da aplicação.  

No entanto, **System Design é um processo contínuo** e deve ser revisado conforme o crescimento e as necessidades do sistema.  

A partir do diagrama e das explicações fornecidas, você pode continuar a explorar e aprimorar a arquitetura, adicionando novos **recursos, serviços e otimizações**.  

De forma proposital, deixamos em aberto algumas questões, como a **distribuição regional de conteúdos estáticos** e **réplicas do banco de dados**.  

👉 **Como você resolveria esses desafios?** 

---

### **Conclusão**  

Como vimos, os princípios de **System Design (SD)** são fundamentais para a construção de sistemas **escaláveis, robustos e distribuídos**. O foco principal do SD não está na tecnologia em si, mas na **arquitetura e no comportamento desejado do sistema**. A escolha de tecnologias, frameworks e mecanismos de mensageria devem ser feita com base nas **necessidades arquiteturais** e nos desafios específicos que precisam ser resolvidos, e não o contrário.  

Além disso, o SD não é um processo estático. Ele exige uma abordagem **contínua e iterativa**, onde decisões precisam ser revisadas e ajustadas conforme o sistema cresce e novos desafios surgem. As escolhas arquiteturais sempre envolvem **trade-offs**, e compreender esses compromissos é essencial para garantir um equilíbrio entre **desempenho, escalabilidade, confiabilidade e custo**.  

Por isso, o desenvolvimento de um sistema eficaz não depende apenas de engenheiros de software, mas de uma colaboração multidisciplinar entre as equipes de **planejamento, desenvolvimento, segurança e manutenção da infraestrutura**. Somente com essa visão holística é possível criar sistemas que sejam **eficientes, resilientes e preparados para o futuro**. Aproveite as [questões](#resultado-do-caso-de-estudo) e perguntas que deixamos intencionalmente em aberto para explorar as [referências](#referencias) e aprofundar-se em cada tópico.

---

### **Referências**

Este post foi baseado nas experiências ao longo dos anos trabalhando com **System Design**, bem como em diversos materiais de referência que ajudaram a consolidar o conhecimento compartilhado aqui. O objetivo foi oferecer uma visão geral sobre os principais desafios e conceitos do SD, mas vale ressaltar que cada tópico abordado possui **muito mais profundidade e nuances**. Por isso, **é altamente recomendável que o leitor explore as referências citadas**, aprofunde-se nos temas e busque aplicar os conceitos no seu próprio contexto. O aprendizado em System Design é contínuo e se torna cada vez mais valioso à medida que novas tecnologias e arquiteturas emergem.

[IBM - CAP Theorem: O que é o Teorema CAP?](https://www.ibm.com/br-pt/topics/cap-theorem)

[Entendendo os algoritmos utilizados em Load Balancers para alocação de servidores](https://www.youtube.com/watch?v=_JHWUKaZ2Do&list=PLFmBNfkCoVHJq7FfY8MyZfEZGE4JIaUkW&index=5)

[Consistent Hashing Explained - System Design Fundamentals](https://www.youtube.com/watch?v=qqqdAPM1l2M)

[O que é Single Point of Failure](https://profissaocloud.com.br/glossario/o-que-e-single-point-of-failure)

[The System Design Primer](https://github.com/donnemartin/system-design-primer)

[Requisitos funcionais e não funcionais: o que são?](https://www.mestresdaweb.com.br/tecnologias/requisitos-funcionais-e-nao-funcionais-o-que-sao)

[Stateful e stateless](https://www.redhat.com/pt-br/topics/cloud-native-apps/stateful-vs-stateless)
[Design Twitter System Design](https://www.youtube.com/watch?v=o5n85GRKuzk&list=PLot-Xpze53le35rQuIbRET3YwEtrcJfdt&index=3)
[Lecture 9 Scalability Harvard Web Development David Malan](https://www.youtube.com/watch?v=-W9F__D3oY4)
[Definitive guide for System Design Interview](https://www.systemdesignhandbook.com/guides/system-design-interview/)
[How to Effectively Use Caching to Improve Microservices Performance](https://dev.to/amplication/how-to-effectively-use-caching-to-improve-microservices-performance-21c1)



Algumas palavras e termos foram destacados ao longo do texto, aqui estão suas definições:

**trade-off**: O *trade off* ou escolha é o nome que se dá a uma decisão que consiste na escolha de uma opção em detrimento de outra. Para se tratar de um trade off o indivíduo deve, necessariamente, deixar de lado alguma opção em sua escolha.

**scaling**: *Scaling*  ou dimensionamento é a capacidade de um sistema de lidar com um aumento na carga de trabalho

**SD**: System Design ou Design de Sistemas

**ACID**: Atomicidade, Consistência, Isolamento e Durabilidade