# Super Bot DevOps: Gestão de AWS & Servidores via Telegram com n8n + Gemini IA

Um fluxo avançado construído no **n8n** que transforma o Telegram em uma central de comando completa para operações de DevOps. 

Este projeto combina o controle rigoroso e determinístico da infraestrutura na nuvem (AWS EC2) com a flexibilidade da Inteligência Artificial (Google Gemini) para a gestão interna do sistema operacional Linux via SSH, usando linguagem natural.

---

## 🎯 O que este bot faz?

O fluxo atua em duas frentes principais:

1. **Gestão de Infraestrutura AWS (Hard-coded & Seguro):**
   Comandos exatos que chamam a API da AWS para gerenciar instâncias EC2. Não há margem para alucinações de IA aqui.
   - Ligar, Desligar e Reiniciar instâncias.
   - Verificar o Status (IP, Estado atual, Hardware).
   - Alterar o tamanho da máquina (Ex: `t2.micro` para `t3.medium`) de forma automatizada com a máquina parada.

2. **Gestão de SO via SSH (Powered by IA):**
   Acesso seguro ao terminal Linux onde o usuário pede o que quer em *linguagem natural*. A IA traduz para um comando seguro, o n8n executa via SSH e a IA "mastiga" o log técnico, devolvendo uma resposta amigável no Telegram.
   - Ex: *"Como está o consumo de memória?"* -> Executa `free -m`.
   - Ex: *"Quais sites estão configurados no Apache?"* -> Lista o diretório `sites-enabled`.
   - Ex: *"Reinicia o container do banco de dados"* -> Executa o `docker-compose restart`.

---

## ⚙️ Arquitetura e Lógica dos Nós (n8n)

O fluxo foi desenhado com o conceito de **Roteamento Mestre**, separando comandos rígidos de requisições de IA.

### 🛡️ 1. Camada de Entrada e Segurança
* **Telegram Trigger:** Ouve todas as mensagens enviadas para o bot.
* **Verifica Usuários (If):** Um filtro de segurança vital. O fluxo só avança se o `chat.id` do remetente estiver na lista de IDs autorizados. Solicitações de estranhos são ignoradas.

### 🧠 2. O Cérebro Roteador
* **Cérebro Lógico (Code Node):** É o maestro da orquestra. Ele analisa o primeiro termo da mensagem. Se for um comando de infraestrutura (ex: `/ligar`, `/mudar`), ele roteia para a AWS. Se for `/ajuda`, exibe o menu. Se for qualquer texto em linguagem natural, ele classifica como `ssh` e envia para a Inteligência Artificial.
* **Roteador Mestre (Switch):** Distribui fisicamente o fluxo para os 6 caminhos possíveis baseados na decisão do Cérebro Lógico.

### ☁️ 3. Braço de Infraestrutura (AWS EC2 API)
* **Nós de HTTP Request (AWS Ligar, Desligar, etc):** Autenticados via AWS IAM, enviam requisições seguras via `POST` (`form-urlencoded`) para a API do EC2.
* **Resp: Status / Ação Executada (Telegram):** Extrai os dados do XML retornado pela AWS (usando Regex) e devolve o status real ou a confirmação de sucesso para o usuário.

### 🤖 4. Braço de Sistema Operacional (SSH + Gemini IA)
* **IA - Entende o Pedido (Chain LLM):** Recebe o texto livre do usuário e avalia sob um prompt de *Segurança Máxima*. Ele tenta mapear a intenção para uma lista estrita de comandos Linux predefinidos. Se a intenção for destrutiva ou fora do escopo, ele retorna "ERRO".
* **Comando Permitido? (If):** Verifica se a IA gerou um comando válido ou barrou a execução.
* **SSH - Executa:** Acessa o servidor via chave privada (autenticação segura) e executa o terminal.
* **IA - Explica o Resultado:** Pega o `stdout` (log técnico do Linux) e usa o Gemini novamente para traduzir a sopa de letrinhas em um resumo claro, em português, formatado para o Telegram (sem usar asteriscos de lista para não quebrar a API do mensageiro).
* **Envia Resposta IA (Telegram):** Entrega o diagnóstico final ao desenvolvedor.

---

## 🛠️ Pré-requisitos para uso

Para importar e rodar este fluxo na sua instância n8n, você precisará de:

1. **Telegram Bot Token:** Criado via *BotFather*.
2. **Credenciais AWS IAM:** Um usuário na AWS com políticas restritas (Least Privilege) para `ec2:DescribeInstances`, `ec2:StartInstances`, `ec2:StopInstances`, `ec2:RebootInstances` e `ec2:ModifyInstanceAttribute`.
3. **Chave Privada SSH:** Para acesso ao servidor alvo.
4. **Google Gemini API Key:** Para o processamento de linguagem natural (O plano gratuito do Google AI Studio é suficiente).
5. (Opcional) **n8n v1.0+**: O fluxo utiliza a versão atualizada do nó `Switch` e `LangChain`.

---

## 💡 Como Configurar

1. Importe o arquivo `workflow.json` para o seu n8n.
2. Adicione as suas credenciais em todos os nós correspondentes (Telegram, AWS, SSH, Google Gemini).
3. No nó **Verifica Usuários**, substitua o valor `YOUR_CHAT_ID_HERE` pelo seu Telegram Chat ID (e adicione os IDs da sua equipe).
4. No nó **Cérebro Lógico**, altere a variável `instance_id` para o ID da sua máquina EC2 e defina sua `regiao` (ex: `sa-east-1`).
5. No nó **IA - Entende o Pedido**, edite o Prompt para incluir os caminhos reais dos seus containers Docker, serviços ou scripts que a IA tem permissão para rodar.

---

## 💬 Exemplos de Uso no Telegram

**Para Infraestrutura:**
> `/status` -> Retorna Estado, IP e Tipo de Hardware.  
> `/desligar` -> Para a máquina para manutenção.  
> `/mudar t3.medium` -> Faz o upgrade do servidor.

**Para o Sistema Operacional:**
> "Como está o espaço em disco?" -> *A IA acessa, roda `df -h` e responde de forma amigável.* > "Quais domínios estão rodando no Apache?" -> *A IA roda a checagem de vhosts e lista os sites.* > "Reinicia o container do ERP pra mim" -> *A IA encontra o path correto, roda o docker-compose restart e avisa que voltou.*

---

## 🤝 Contribuição
Sinta-se à vontade para fazer um *fork* do projeto, criar *pull requests* com novos comandos ou melhorias nos prompts de proteção da IA.

Desenvolvido para simplificar a vida de SysAdmins e Engenheiros DevOps. Faca na caveira! 🔪💀
