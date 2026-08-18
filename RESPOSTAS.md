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