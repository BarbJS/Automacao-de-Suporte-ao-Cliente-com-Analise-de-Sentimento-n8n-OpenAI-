# 🦾 Automação de Suporte ao Cliente com Análise de Sentimento (n8n + OpenAI) 🤝

Este projeto consiste em um fluxo de trabalho automatizado (Workflow) desenvolvido na plataforma low-code n8n. O sistema monitora feedbacks de clientes, utiliza Inteligência Artificial para analisar o sentimento (Positivo, Neutro ou Negativo) e envia respostas de e-mail personalizadas automaticamente, garantindo agilidade e padronização no atendimento além de promover a fidelização e retenção B2C.

## 📖 Sobre o Projeto

O objetivo desta automação é eliminar a triagem manual e lenta de feedbacks. Em cenários de alto volume de dados, ler cada comentário e escrever uma resposta individual é inviável. Este sistema delega a leitura e a redação inicial para a IA, deixando para os humanos apenas a supervisão ou o tratamento de casos complexos. Ideal para operações B2C de alto volume (como Varejo, SaaS PLG ou Apps de Delivery), onde a velocidade de resposta é o principal fator de satisfação do cliente.

## 🚀 Objetivos e Benefícios

- Hiper-Personalização em Escala: Cada resposta é gerada pela IA considerando o conteúdo específico do feedback do cliente, evitando respostas genéricas de "copia e cola", além de promover o suporte ao cliente de uma forma dinâmica, natural e fluida.

- Tempo de Resposta Zero: O cliente recebe um retorno quase imediato após o processamento.

- Estruturação e Organização de Dados: ransforma texto livre (feedback) em dados categóricos (JSON) para análise de métricas. Classifica automaticamente a base de clientes por sentimento, gerando insumos valiosos para relatórios de CSM (Customer Success).

## 🔄 O Fluxo de Dados

- Monitoramento: O sistema verifica periodicamente uma base de dados (Forms) em busca de novos feedbacks.

- Cérebro (IA): O conteúdo é enviado para a OpenAI (GPT), que classifica o tom da mensage, baseado em System Prompt e Few-shot Prompting.

- Decisão: Com base no sentimento, o fluxo segue caminhos diferentes (Roteamento).

- Ação: Uma resposta adequada a cada sentimento é redigida e enviada via Gmail.

- Registro: O sistema atualiza a planilha de feeedbacks para marcar o item com dados de recebimento do feedback, evitando duplicidade, além de auxiliar na tomada de decisões do negócio.

## 🛠️ Tecnologias e Nós Utilizados

- n8n: O projeto roda sobre a orquestração do n8n, uma ferramenta de automação de fluxo de trabalho (alternativa open-source ao Zapier/Make).

- Google Gemini AI: Utilizado em duas etapa para classificação de sentimento (Sentiment Analysis) e para redação criativa do e-mail de resposta (Drafting).

- Google Sheets: Atua como banco de dados (leitura de feedbacks e atualização de status).

- Gmail: Gateway de envio de e-mails (SMTP/API).

## 📂 Estrutura Técnica e Explicação do Workflow

O fluxo foi desenhado para ser determinístico e à prova de falhas, manipulando dados estruturados em JSON ao longo de todo o pipeline.

1. Ingestão e Controle (Trigger & Read)

Schedule Trigger: Configurado para iniciar o ciclo periodicamente (ex: a cada 1 minuto).

Google Sheets: Atua como um buffer de entrada e gestor de estado.

2. Cérebro: Sentiment Analysis & Parse AI Response

O nó do Google Gemini não apenas "lê" o texto, mas é forçado a retornar dados estruturados.

- Prompt Engineering: O prompt instrui o modelo a agir como um analista de suporte experiente.

- Structured Output Parser (Parse AI Response): Ao invés de receber um texto solto, o workflow obriga o Gemini a devolver um objeto JSON rígido. Isso elimina a necessidade de expressões regulares (Regex) e previne erros onde a IA adiciona conversa fiada antes da resposta.

- Formato de Saída (Output Schema):
  JSON
{
  "sentiment": "positive | neutral | negative",
  "category": "product_quality | delivery | support | other",
  "urgency": "low | medium | high"
}

3. Critical Feedback Router (Roteamento Lógico)

Após o parsing, entra em ação o nó Switch (Router), que atua como o "semáforo" do sistema.

Lógica de Decisão: O roteador lê a chave output.sentiment do JSON gerado pelo Gemini.

Rota 1 (Crítico/Negativo): Se sentiment contém negative OU urgency = high.

Rota 2 (Neutro): Se sentiment é neutral.

Rota 3 (Positivo): Se sentiment é positive.

*Segurança*: Se o JSON vier quebrado ou com uma classificação desconhecida, o roteador envia para uma saída de "Fallback", garantindo que nenhum erro técnico deixe o cliente sem resposta.

4. Geração de Resposta e Envio
   
Para cada rota, existe um novo prompt especializado enviado ao Gemini:

- Draft Response: A IA gera o corpo do e-mail usando o contexto específico (ex: para negativos, o tom é apologético; para positivos, pede review).

- Gmail & Update: Envia o e-mail final e atualiza a planilha original com as informações fechando o ciclo.

*Observação*: Se o feedback negativo for crítico e urgente, o workflow envia uma mensagem ao responsável pelo suporte ao cliente indicando urgência na resolução do problema.

## ⚡ Evolução da Arquitetura: De Polling para Webhooks

Atualmente, este projeto utiliza um gatilho de agendamento (Schedule Trigger). Para integrar este sistema a um aplicativo real em produção (App Delivery, E-commerce, Typeform), recomenda-se substituir o gatilho por um Webhook.

- O que muda?

Em vez do n8n perguntar "tem dados novos?" a cada minuto (Polling), o seu App avisa o n8n "aqui estão dados novos!" instantaneamente (Push).

- Como funcionaria a implementação:

No n8n: Substituir o nó Schedule Trigger pelo nó Webhook.

- Configuração: O nó Webhook gera uma URL única (ex: https://seu-n8n.com/webhook/feedback-recebido).

- No seu App/Site: Configura-se uma chamada HTTP POST para essa URL sempre que um usuário enviar um feedback.

- Payload (Dados enviados):
JSON
{
  "cliente_email": "joao@email.com",
  "feedback_texto": "Adorei o serviço, muito rápido!",
  "cliente_nome": "João"
}

- Otimização: Remove-se o nó Google Sheets (Read), pois os dados chegam prontos no gatilho. O Sheets é mantido apenas no final para fins de registro (Log).

## 💻 Como Executar o Projeto

- Pré-requisitos: Uma instância do n8n (Cloud, Desktop ou Docker).

- Credenciais configuradas no n8n para: Google Gemini (AI Studio), Google Sheets e Gmail.

- Passo a Passo:
  
1. Importar: No painel do n8n, vá em "Workflows" > "Import from File" e selecione o arquivo .json deste repositório.

2. Configurar Planilha: Crie uma planilha no Google Sheets com as colunas: Feedback, Email, Name, Sentiment e Timestamp.

3. Vincular Credenciais:

No nó do Gemini, insira sua Google AI Studio API Key.

Nos nós do Google, autentique com sua conta (OAuth2 ou Service Account).

Mapear IDs: No nó do Google Sheets, selecione o arquivo da planilha que você criou.

Ativar: Mude o switch no topo direito para Active.

4. Executar o workflow: Clique na opção 'Executar' na parte inferior central do worflow e aguarde o fluxo.

## ⚠️ Limitações e Custos

- Limites de API: O uso da API do Gemini está sujeito a limites de taxa (Rate Limits). Verifique a cota da sua chave no Google AI Studio.

- Interpretação de Contexto: Embora o Gemini seja avançado, sarcasmo sutil pode ser mal interpretado (ex: "Maravilha, esperei 3 horas" classificado como Positivo). A implementação do "Critical Feedback Router" ajuda a mitigar isso, mas revisão humana periódica é recomendada. Recomenda-se implantar Human in the loop com monitoramento contínuo.
