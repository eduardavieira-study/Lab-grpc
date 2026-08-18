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