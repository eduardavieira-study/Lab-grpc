## Respostas

## 4.1 Tarefa

### **1) O endereço do servidor (localhost, IP, grupo multicast) está escrito diretamente no código do cliente? Isso favorece ou prejudica a transparência de localização?**

Sim. Em TCP, UDP e WebSocket, o endereço do servidor e a porta estão definidos diretamente no código do cliente, o que prejudica a transparência de localização, pois o cliente depende de uma localização específica do servidor. No Multicast, o cliente utiliza o endereço do grupo multicast 230.0.0.1 e a porta, em vez do endereço físico do servidor. Isso favorece a transparência de localização, pois diferentes servidores podem enviar para o mesmo grupo sem que o cliente precise conhecer a máquina de origem.

---

### **2) Para “perguntar uma coisa” ao servidor, o cliente precisa montar uma string de texto manualmente (e o servidor precisa interpretá-la/fazer parsing)? Isso é meio-termo, presença ou ausência de transparência de acesso?**

Sim. Nas implementações TCP e UDP, o cliente envia mensagens de texto e o servidor precisa interpretar o conteúdo recebido para determinar o que fazer. Isso representa um meio-termo de transparência de acesso: existe uma abstração simples de envio e recebimento de mensagens, mas o programador ainda precisa conhecer o protocolo da aplicação e formatar as mensagens manualmente. Não há uma abstração completa como RPC/RMI, em que a chamada remota se aproxima de uma chamada local.

---

### **3) O que aconteceria com o cliente se o servidor mudasse de máquina amanhã? Alguma dessas quatro soluções sobreviveria a essa mudança sem alterar o código-fonte do cliente?** 

Em TCP, UDP e WebSocket, se o servidor mudar de máquina, o cliente continuará tentando se conectar/enviar mensagens para localhost ou para o endereço configurado anteriormente. Assim, a comunicação falharia e seria necessário alterar a configuração/endereço utilizado pelo cliente. Nas implementações apresentadas, isso significa alterar o código-fonte.
Multicast é a exceção, o cliente não se conecta ao endereço físico do servidor, mas ao grupo lógico `230.0.0.1`. Se o novo servidor continuar enviando mensagens para o mesmo grupo e porta, o cliente continuará recebendo as mensagens sem nenhuma alteração em seu código-fonte.


## Parte A

### 1) Dentre os 8 tipos de transparência listados, qual você diria que é a mais visível para o programador que está usando um serviço remoto (e não construindo a infraestrutura por trás dele)? Justifique.

A transparência de acesso é a mais visível para o programador que consome um serviço remoto. Isso ocorre porque ela afeta diretamente a forma como o código de integração é escrito no dia a dia, ocultando toda a complexidade de baixo nível da comunicação de rede (como manipulação de sockets, serialização de tipos de dados e tratamento de protocolos de transporte) sob a sintaxe familiar de uma chamada de função local. Assim, para o desenvolvedor da aplicação cliente, a facilidade de usar uma chamada de método limpa e uniforme representa o impacto prático mais imediato na integração de sistemas distribuídos.

---

### 2) Transparência total é sempre desejável? Dê um exemplo (pode ser hipotético) de uma situação em que esconder completamente que uma operação é remota atrapalharia mais do que ajudaria (dica: pense em desempenho ou em tratamento de falhas).

Não, a transparência total nem sempre é desejável. Se uma chamada de rede for completamente mascarada como local, o desenvolvedor pode acidentalmente colocá-la dentro de um laço de repetição intenso, gerando um gargalo severo de latência que inviabilizaria o desempenho da aplicação. Além disso, operações remotas estão sujeitas a falhas parciais, perdas de pacotes e timeouts que exigem lógicas específicas de tratamento de erros, como políticas de novas tentativas (retries) ou fallbacks. Ocultar totalmente o aspecto distribuído impede que o programador enxergue esses riscos e implemente a resiliência adequada para mitigar falhas de rede.

### 3) Comparando o cliente TCP do laboratório anterior com o cliente gRPC que você vai construir agora: qual dos dois exige que você “pense em rede” (sockets, send/receive, parsing de string) e qual permite que você “pense no problema” (chamar uma função e receber um resultado)? A que tipo de transparência isso se relaciona?

O cliente TCP exige que o programador pense em rede, pois requer a manipulação direta de sockets, o gerenciamento manual do fluxo de envio/recebimento de bytes e o parsing textual das strings transmitidas. Por outro lado, o gRPC permite pensar diretamente no problema de negócio, já que a interação com o servidor ocorre por meio de stubs gerados automaticamente, nos quais o programador simplesmente chama um método comum e recebe o objeto de resposta estruturado. Essa diferença está diretamente relacionada à transparência de acesso, que oculta a representação de dados e a comunicação física subjacente, e também à transparência de localização, que mascara os endereços físicos de rede por trás de um canal de comunicação abstrato.


---

## Parte B

### 1) No laboratório anterior, cada um de vocês definiu o formato das mensagens de forma implícita (comentários e convenção entre quem escreveu o cliente e o servidor). Aqui, o formato está no central.proto. Qual a vantagem de ter esse contrato explícito e gerado automaticamente em vez de combinado apenas “de boca”?

A principal vantagem de possuir um contrato explícito no arquivo `.proto` é a garantia de consistência estrutural e a eliminação de erros humanos de integração. Em um acordo informal, qualquer desvio de caractere ou de tipo de dado quebra a comunicação e exige depuração manual complexa. Com o Protocol Buffers, o contrato é rígido e atua como uma especificação formal de tipos e campos, permitindo que o compilador gere automaticamente o código de serialização e desserialização robusto para ambos os lados, o que previne incompatibilidades durante o desenvolvimento de sistemas distribuídos.

---

### 2) O mesmo arquivo central.proto gerou código para Java e para Python. O que isso sugere sobre como equipes que usam linguagens diferentes podem se comunicar em um sistema distribuído real?

Isso sugere que em um sistema distribuído real é possível obter alta interoperabilidade e independência tecnológica, permitindo que diferentes equipes de desenvolvimento escolham a linguagem de programação mais adequada para seus respectivos serviços (como Java para sistemas transacionais robustos e Python para ciência de dados). O uso de um contrato neutro comum, como o Protocol Buffers, garante que todas essas aplicações consigam conversar perfeitamente entre si na rede, delegando a tradução dos dados e a geração de stubs para as ferramentas automatizadas.

---

### 3) Observe os arquivos gerados (target/generated-sources/.../CentralAtendimentoGrpc.java ou central_pb2_grpc.py). Sem entender todo o código gerado, você consegue identificar onde ficam definidas as operações ConsultarHorario e AcompanharAvisos? Cite o nome de pelo menos uma classe ou método gerado que você reconheceu.

Sim, é possível identificar onde as operações estão mapeadas no código gerado. No arquivo Python `central_pb2_grpc.py`, as operações são explicitadas na classe `CentralAtendimentoStub` (para a chamada do cliente) e na classe base `CentralAtendimentoServicer`, que contém os métodos `ConsultarHorario` e `AcompanharAvisos` prontos para serem sobrescritos na implementação do servidor. Em Java, as operações são geradas como assinaturas de métodos como `consultarHorario` e `acompanharAvisos` dentro da classe abstrata `CentralAtendimentoImplBase` no arquivo `CentralAtendimentoGrpc.java`.

## Parte C
### 1) No cliente, a linha stub.consultarHorario(pergunta) (Java) ou stub.ConsultarHorario(...) (Python) parece uma chamada de método comum. Cite, em alto nível, pelo menos três coisas que acontecem “por baixo dos panos” entre essa chamada e o return da função no servidor.

Por baixo dos panos, a chamada ao stub desencadeia uma série de mecanismos de rede e representação de dados gerenciados pelo framework. Primeiramente, ocorre a serialização, em que o stub converte o objeto de requisição da linguagem para um fluxo binário compacto definido pelo Protocol Buffers. Em seguida, o framework realiza a transmissão de rede, encapsulando essa mensagem binária em frames HTTP/2 e enviando-a pelo canal TCP estabelecido. Por fim, ao chegar no servidor gRPC, ocorre a desserialização dos bytes binários recebidos de volta em um objeto correspondente da linguagem local, que é então repassado como argumento à execução do método correspondente do servidor.

---

### 2) Compare esta implementação com o ClienteTCP do roteiro anterior. Onde estava, no TCP, o equivalente a “montar a mensagem” e “interpretar a resposta”? Quem faz esse trabalho agora, no gRPC?

No ClienteTCP do roteiro anterior, a montagem da mensagem era feita manualmente pelo próprio programador concatenando strings, adicionando quebras de linha (`\n`) e convertendo o texto para bytes com `encode("utf-8")`. A interpretação da resposta também era manual, exigindo ler a linha crua recebida do socket com `readline()` e decodificá-la. Agora, no gRPC, todo esse fluxo repetitivo de formatação, codificação e decodificação de dados em rede é abstraído e executado de forma transparente pelas classes stubs geradas automaticamente pelo compilador do Protocol Buffers a partir do contrato `.proto`.

---

### 3) O que aconteceria se você chamasse stub.consultarHorario(pergunta) com o servidor desligado? Teste e descreva o comportamento observado (em qualquer uma das duas linguagens).

Ao executar o cliente Python com o servidor desligado, a aplicação é interrompida lançando a exceção `grpc._channel._InactiveRpcError`. O comportamento observado detalha que a requisição RPC foi terminada com o status `StatusCode.UNAVAILABLE` e a mensagem de erro específica `"failed to connect to all addresses; last error: UNKNOWN: ipv4:127.0.0.1:50135: Failed to connect to remote host: Connection refused"`. Isso comprova que, sem o servidor ativo para receber e escutar a conexão na porta configurada (no caso, a porta 50135), o cliente gRPC não consegue estabelecer a sessão HTTP/2 subjacente e aborta imediatamente a operação remota.

## Parte D

### 1) No laboratório anterior, o Multicast usava um endereço de grupo (230.0.0.1) para alcançar vários clientes com um único envio; aqui, o streaming gRPC é um servidor conversando com um cliente por vez, só que ao longo de uma conexão só. Se você quisesse que vários clientes gRPC recebessem os mesmos avisos ao mesmo tempo, o que precisaria mudar na implementação do servidor?

Para que múltiplos clientes gRPC recebam os mesmos avisos ao mesmo tempo, o servidor precisaria implementar um padrão de comunicação Publish-Subscribe (Pub/Sub) mantendo um registro ativo dos clientes conectados. Em vez de gerar e enviar os dados em um laço fechado e isolado para um único cliente, o servidor precisaria armazenar em uma lista concorrente compartilhada as referências dos canais de resposta ativos de cada requisição (como os objetos `StreamObserver` no Java). Dessa forma, sempre que um novo aviso fosse produzido no servidor, um despachante percorreria essa lista e enviaria a mensagem individualmente para a conexão gRPC ponto a ponto de cada cliente registrado, simulando o efeito de broadcast do multicast.

---

### 2) Compare o método de streaming em Java (StreamObserver, chamando onNext() repetidamente) com o de Python (uma função geradora usando yield). Os dois alcançam o mesmo resultado - qual das duas abordagens você achou mais natural de entender? Justifique.

A abordagem em Python utilizando uma função geradora com `yield` é mais natural e intuitiva de entender. Isso ocorre porque o Python permite expressar o fluxo de dados sequencialmente através de estruturas de controle comuns, onde o desenvolvedor usa o laço de repetição usual e delega o envio incremental das mensagens ao comportamento nativo de geradores da linguagem. Em contrapartida, o modelo em Java utiliza callbacks reativos (`StreamObserver`) com métodos específicos como `onNext()`, `onError()` e `onCompleted()`, o que exige uma mentalidade de programação orientada a eventos mais verbosa e menos linear.

---

### 3) No método acompanharAvisos/AcompanharAvisos, o que aconteceria se o cliente fechasse a conexão (por exemplo, fechando o terminal) no meio do envio dos 5 avisos? Pesquise ou teste o comportamento e descreva o que observou.

Se o cliente fechar a conexão abruptamente antes de receber todas as mensagens, o servidor gRPC detecta que o canal de comunicação foi encerrado e interrompe o processamento do streaming. Em Python, a execução do gerador é interrompida pelo framework gRPC e a verificação do contexto (`context.is_active()`) passa a retornar falso, impedindo o avanço do laço do servidor. Em Java, a próxima chamada ao método `observador.onNext()` no servidor falha e lança uma exceção do tipo `io.grpc.StatusRuntimeException` com o status `CANCELLED`, informando que o cliente cancelou o canal de comunicação e forçando a execução a desviar para a captura de erro.