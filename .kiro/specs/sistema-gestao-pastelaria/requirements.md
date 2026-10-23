# Requirements Document

## Introduction

Sistema completo de gestão para pastelaria com 3 unidades (Centro, Norte, Sul), focado em registro de pedidos, acompanhamento de preparação e impressão física de comandas em impressora térmica. O sistema permite o controle de todo o fluxo operacional desde o registro do pedido até a impressão da comanda para a cozinha, com acompanhamento visual do status de preparação e entrega.

## Glossary

- **Sistema**: O sistema de gestão de pastelaria completo (frontend + backend + serviço de impressão)
- **Unidade**: Uma das três localizações físicas da pastelaria (Centro, Norte, Sul)
- **Pedido**: Registro de compra de produtos por um cliente, contendo itens, valores e informações de entrega
- **Comanda**: Documento físico impresso contendo os detalhes do pedido para a cozinha
- **Serviço_Impressão**: Script Python local que processa a fila de impressão e comunica com impressoras térmicas
- **Fila_Impressão**: Estrutura persistente no backend que armazena trabalhos de impressão pendentes
- **Trabalho_Impressão**: Registro individual na fila de impressão associado a um pedido específico
- **Impressora_Térmica**: Dispositivo físico de impressão ESC/POS conectado via Ethernet (Elgin/Bematech i9 Full 2)
- **API_Key**: Chave de autenticação gerada no painel admin para o Serviço_Impressão
- **Status_Pedido**: Estado atual do pedido (Recebido, Em preparação, Pronto, Entregue)
- **Tipo_Atendimento**: Modalidade de venda (Balcão, Retirada, Entrega)
- **Modo_Operação**: Configuração do sistema (Demonstração ou Impressora Real)
- **Backend**: Servidor FastAPI em Python que gerencia dados e lógica de negócio
- **Frontend**: Interface React com TypeScript para interação do usuário
- **Dashboard**: Painel administrativo com métricas e indicadores de vendas

## Requirements

### Requirement 1: Gestão de Pedidos

**User Story:** As a atendente da pastelaria, I want to cadastrar pedidos com todas as informações necessárias, so that a cozinha receba os detalhes completos para preparação

#### Acceptance Criteria

1. WHEN um novo pedido é criado, THE Sistema SHALL salvar no Backend com unidade, nome do cliente, tipo de atendimento, produtos, quantidades, adicionais, observações, forma de pagamento, taxa de entrega e desconto
2. WHEN produtos e adicionais são selecionados, THE Sistema SHALL calcular automaticamente o valor total considerando quantidades, taxas e descontos
3. WHEN um pedido é confirmado, THE Sistema SHALL atribuir um número único sequencial e horário de registro
4. THE Frontend SHALL enviar dados do pedido para o Backend via API REST
5. THE Backend SHALL persistir pedidos no banco de dados SQLite
6. WHEN um pedido é salvo, THE Sistema SHALL criar um Trabalho_Impressão na Fila_Impressão com estado "Aguardando"

### Requirement 2: Visualização Kanban de Pedidos

**User Story:** As a gerente da pastelaria, I want to visualizar pedidos em colunas por status, so that eu possa acompanhar o fluxo de preparação em tempo real

#### Acceptance Criteria

1. THE Frontend SHALL exibir colunas: Recebidos, Em preparação, Prontos, Entregues
2. WHEN pedidos são carregados, THE Frontend SHALL posicionar cada pedido na coluna correspondente ao seu Status_Pedido
3. THE Frontend SHALL exibir em cada pedido: número, unidade, horário, cliente, Tipo_Atendimento, itens, adicionais, observações, forma de pagamento, valor total e status de impressão
4. WHEN um usuário clica em um pedido, THE Sistema SHALL abrir detalhes completos em modal ou painel expandido
5. WHEN um usuário altera o Status_Pedido, THE Sistema SHALL atualizar o Backend e mover o pedido para a coluna correspondente
6. THE Frontend SHALL consultar o Backend periodicamente via polling para atualizar a visualização com novos pedidos e mudanças de status

### Requirement 3: Dashboard Administrativo

**User Story:** As a proprietário da pastelaria, I want to visualizar métricas consolidadas de vendas, so that eu possa acompanhar o desempenho do negócio

#### Acceptance Criteria

1. THE Dashboard SHALL exibir total de vendas do dia em valor monetário
2. THE Dashboard SHALL exibir quantidade de pedidos por Status_Pedido (recebidos, em preparação, prontos, concluídos)
3. THE Dashboard SHALL exibir faturamento individual por Unidade
4. WHEN um filtro de período é aplicado, THE Dashboard SHALL recalcular métricas para o intervalo selecionado
5. WHEN um filtro de Unidade é aplicado, THE Dashboard SHALL exibir métricas apenas da Unidade selecionada
6. THE Frontend SHALL incluir menu lateral com navegação para: painel, pedidos, produtos, unidades, relatórios, configurações

### Requirement 4: Configuração de Impressora

**User Story:** As a administrador do sistema, I want to configurar os parâmetros da impressora térmica, so that o sistema consiga enviar comandas para impressão física

#### Acceptance Criteria

1. THE Sistema SHALL permitir cadastrar nome da impressora, Unidade associada, endereço IP local, porta de comunicação, largura do papel e modo de conexão
2. WHEN um usuário clica em "testar comunicação", THE Sistema SHALL verificar conectividade com a Impressora_Térmica via Serviço_Impressão e exibir resultado
3. WHEN um usuário clica em "imprimir página de teste", THE Sistema SHALL enviar comando de impressão de teste para a Impressora_Térmica
4. THE Sistema SHALL exibir status da conexão (Conectada, Desconectada, Erro)
5. THE Sistema SHALL exibir data e hora da última impressão bem-sucedida
6. THE Sistema SHALL exibir histórico dos últimos erros de impressão registrados
7. THE Sistema SHALL garantir que cada Unidade esteja associada a no máximo uma Impressora_Térmica

### Requirement 5: Serviço Local de Impressão

**User Story:** As a sistema, I want to processar trabalhos de impressão de forma confiável e segura, so that comandas sejam impressas corretamente sem duplicações

#### Acceptance Criteria

1. THE Serviço_Impressão SHALL ser executado como script Python no computador da loja
2. WHEN iniciado, THE Serviço_Impressão SHALL autenticar no Backend usando API_Key configurada
3. THE Serviço_Impressão SHALL consultar periodicamente a Fila_Impressão no Backend para identificar Trabalhos_Impressão com estado "Aguardando"
4. WHEN um Trabalho_Impressão é identificado, THE Serviço_Impressão SHALL verificar se a Unidade corresponde à Impressora_Térmica configurada localmente
5. WHEN a Unidade corresponde, THE Serviço_Impressão SHALL formatar a comanda e enviar comando ESC/POS para a Impressora_Térmica via rede Ethernet
6. WHEN impressão é bem-sucedida, THE Serviço_Impressão SHALL notificar o Backend que atualizará o estado do Trabalho_Impressão para "Impresso"
7. IF impressão falha, THEN THE Serviço_Impressão SHALL atualizar o estado do Trabalho_Impressão para "Falhou" com registro do erro
8. WHEN um Trabalho_Impressão falha, THE Serviço_Impressão SHALL realizar retry com intervalo exponencial sem criar Trabalhos_Impressão duplicados
9. THE Serviço_Impressão SHALL registrar logs de todas as operações (consultas, impressões, erros) localmente

### Requirement 6: Fila de Impressão Persistente

**User Story:** As a sistema, I want to gerenciar trabalhos de impressão de forma persistente e controlada, so that não ocorram duplicações ou perdas de comandas

#### Acceptance Criteria

1. THE Backend SHALL criar um Trabalho_Impressão único para cada pedido com identificador UUID
2. THE Backend SHALL armazenar Trabalhos_Impressão no banco de dados com estados: Aguardando, Em processamento, Impresso, Falhou
3. WHEN um Trabalho_Impressão é consultado pelo Serviço_Impressão, THE Backend SHALL atualizar estado para "Em processamento" atomicamente
4. THE Backend SHALL registrar horário de criação, horário de impressão, resultado e mensagens de erro em cada Trabalho_Impressão
5. THE Backend SHALL prevenir criação de Trabalhos_Impressão duplicados para o mesmo pedido (exceto reimpressões explícitas)
6. WHEN um usuário solicita reimpressão, THE Backend SHALL criar novo Trabalho_Impressão marcado como "Segunda via" com registro do usuário solicitante
7. THE Backend SHALL expor endpoint protegido por API_Key para o Serviço_Impressão consultar e atualizar Trabalhos_Impressão

### Requirement 7: Formatação de Comanda

**User Story:** As a cozinheiro, I want to receber comandas impressas com informações claras e organizadas, so that eu possa preparar os pedidos corretamente

#### Acceptance Criteria

1. THE Sistema SHALL formatar comanda contendo: nome da pastelaria, número do pedido em destaque, Unidade, data e horário, Tipo_Atendimento, nome do cliente
2. THE Sistema SHALL incluir na comanda lista de itens com quantidades, adicionais selecionados e observações específicas
3. THE Sistema SHALL incluir observações para cozinha em seção destacada quando presentes
4. THE Sistema SHALL incluir forma de pagamento e valor total na comanda
5. THE Sistema SHALL ajustar formatação para largura do papel térmico configurado (geralmente 58mm ou 80mm)
6. THE Sistema SHALL codificar texto com suporte a acentuação em português (UTF-8 ou Code Page 850)
7. WHEN a Impressora_Térmica suporta corte automático, THE Sistema SHALL enviar comando de corte ao final da comanda
8. WHERE o Trabalho_Impressão é marcado como "Segunda via", THE Sistema SHALL incluir indicação visível "SEGUNDA VIA" no topo da comanda

### Requirement 8: Modos de Operação

**User Story:** As a usuário do sistema, I want to alternar entre modo demonstração e impressora real, so that eu possa testar o fluxo sem equipamento físico ou usar impressão real quando disponível

#### Acceptance Criteria

1. THE Sistema SHALL permitir configurar Modo_Operação como "Demonstração" ou "Impressora Real" por Unidade
2. WHERE Modo_Operação é "Demonstração", THE Frontend SHALL exibir prévia visual da comanda sem enviar para Fila_Impressão
3. WHERE Modo_Operação é "Demonstração", THE Sistema SHALL exibir indicador visual claro "MODO DEMONSTRAÇÃO" na interface
4. WHERE Modo_Operação é "Impressora Real", THE Sistema SHALL criar Trabalho_Impressão na Fila_Impressão ao confirmar pedido
5. WHERE Modo_Operação é "Impressora Real", THE Sistema SHALL indicar sucesso de impressão somente após confirmação do Serviço_Impressão
6. THE Frontend SHALL exibir status de impressão diferenciado: "Simulado" para demonstração e "Aguardando/Impresso/Falhou" para modo real

### Requirement 9: Separação por Unidade

**User Story:** As a sistema, I want to garantir que pedidos sejam associados corretamente às unidades e impressoras, so that comandas sejam impressas no local correto

#### Acceptance Criteria

1. WHEN um pedido é criado, THE Sistema SHALL exigir seleção obrigatória de uma Unidade
2. THE Sistema SHALL persistir a associação entre pedido e Unidade de forma imutável
3. THE Backend SHALL filtrar Trabalhos_Impressão por Unidade ao responder consultas do Serviço_Impressão
4. THE Serviço_Impressão SHALL processar apenas Trabalhos_Impressão cuja Unidade corresponda à Impressora_Térmica configurada localmente
5. THE Dashboard e visualização Kanban SHALL permitir filtragem por Unidade
6. THE Sistema SHALL prevenir envio de pedidos para Impressora_Térmica de Unidade diferente da associada ao pedido

### Requirement 10: Relatórios e Financeiro

**User Story:** As a gerente financeiro, I want to gerar relatórios de vendas e faturamento, so that eu possa analisar desempenho e fazer planejamento

#### Acceptance Criteria

1. THE Sistema SHALL calcular e exibir faturamento total por Unidade em período selecionável
2. THE Sistema SHALL contabilizar quantidade de pedidos por Unidade e por Status_Pedido
3. THE Sistema SHALL consolidar vendas por forma de pagamento em período selecionável
4. THE Sistema SHALL listar cancelamentos e descontos aplicados com justificativas quando disponíveis
5. THE Sistema SHALL permitir exportar relatórios em formato CSV ou PDF
6. WHERE relatórios incluem dados de demonstração, THE Sistema SHALL identificar claramente com marcador visual "DADOS DE DEMONSTRAÇÃO"

### Requirement 11: Dados de Demonstração

**User Story:** As a avaliador do sistema, I want to visualizar dados pré-cadastrados de demonstração, so that eu possa explorar funcionalidades sem configuração inicial

#### Acceptance Criteria

1. WHEN o sistema é iniciado pela primeira vez, THE Backend SHALL criar três Unidades: Centro, Norte e Sul
2. THE Sistema SHALL criar pedidos fictícios de demonstração distribuídos entre as três Unidades
3. THE Sistema SHALL criar produtos de demonstração (pastéis, bebidas, adicionais) com preços
4. THE Frontend SHALL exibir indicador visual "DEMO" em dados fictícios na interface
5. THE Sistema SHALL permitir exclusão de dados de demonstração via painel administrativo
6. THE Sistema SHALL prevenir mistura não identificada de dados reais com dados de demonstração em relatórios

### Requirement 12: Gestão de Produtos

**User Story:** As a administrador da pastelaria, I want to cadastrar e gerenciar produtos, so that o cardápio esteja atualizado para registro de pedidos

#### Acceptance Criteria

1. THE Sistema SHALL permitir cadastrar produtos com nome, descrição, categoria, preço e disponibilidade
2. THE Sistema SHALL permitir cadastrar adicionais com nome e preço adicional
3. WHEN um produto é marcado como indisponível, THE Frontend SHALL ocultar o produto no cadastro de novos pedidos
4. THE Sistema SHALL permitir editar informações de produtos existentes
5. THE Sistema SHALL permitir associar adicionais disponíveis para cada categoria de produto
6. THE Backend SHALL validar que preços sejam valores numéricos positivos

### Requirement 13: Autenticação e Segurança

**User Story:** As a administrador do sistema, I want to controlar acesso ao painel administrativo e ao serviço de impressão, so that apenas usuários autorizados realizem operações

#### Acceptance Criteria

1. THE Sistema SHALL permitir gerar API_Keys no painel administrativo
2. THE Backend SHALL validar API_Key em todos os endpoints acessados pelo Serviço_Impressão
3. WHEN uma API_Key inválida é fornecida, THE Backend SHALL retornar erro HTTP 401 Unauthorized
4. THE Sistema SHALL permitir revogar API_Keys existentes
5. THE Sistema SHALL registrar em log todas as operações autenticadas com API_Key (consultas, atualizações de estado)
6. THE Backend SHALL armazenar API_Keys de forma segura (hash ou criptografia)

### Requirement 14: Tratamento de Erros de Impressão

**User Story:** As a atendente, I want to ser notificada quando comandas não forem impressas, so that eu possa tomar ação corretiva rapidamente

#### Acceptance Criteria

1. WHEN um Trabalho_Impressão falha após todos os retries, THE Sistema SHALL notificar usuários via interface do Frontend
2. THE Frontend SHALL exibir indicador visual de "Erro de impressão" em pedidos com falha
3. THE Sistema SHALL registrar mensagem de erro detalhada (conexão recusada, impressora offline, erro de protocolo)
4. THE Sistema SHALL permitir reimpressão manual de pedidos com falha via interface
5. THE Dashboard SHALL exibir contador de comandas com erro de impressão pendentes de ação
6. WHEN comunicação com Impressora_Térmica é restaurada, THE Serviço_Impressão SHALL retomar processamento de Trabalhos_Impressão pendentes

### Requirement 15: Parser e Pretty Printer de Comandas

**User Story:** As a desenvolvedor do sistema, I want to ter componentes robustos para gerar e validar comandas, so that a formatação seja consistente e testável

#### Acceptance Criteria

1. THE Sistema SHALL implementar Parser que converte dados de pedido em estrutura de Comanda
2. WHEN dados de pedido válidos são fornecidos, THE Parser SHALL gerar objeto Comanda com todas as seções (cabeçalho, itens, rodapé)
3. WHEN dados de pedido inválidos são fornecidos, THE Parser SHALL retornar erro descritivo indicando campo problemático
4. THE Sistema SHALL implementar Pretty_Printer que formata objeto Comanda em texto para impressão
5. THE Pretty_Printer SHALL ajustar largura de linhas conforme configuração da Impressora_Térmica
6. FOR ALL objetos Comanda válidos, THE Sistema SHALL satisfazer propriedade: Parser(Pretty_Printer(Comanda)) produz objeto equivalente (round-trip property)
7. THE Parser e Pretty_Printer SHALL suportar corretamente caracteres acentuados em português

### Requirement 16: Testes Automatizados

**User Story:** As a desenvolvedor do sistema, I want to executar testes automatizados, so that funcionalidades críticas sejam validadas continuamente

#### Acceptance Criteria

1. THE Sistema SHALL incluir testes automatizados para cadastro de pedido com dados válidos e inválidos
2. THE Sistema SHALL incluir testes automatizados para cálculo de valor total com diferentes combinações de produtos, taxas e descontos
3. THE Sistema SHALL incluir testes automatizados para geração de comanda validando formato e conteúdo
4. THE Sistema SHALL incluir testes automatizados para associação entre Unidade e Impressora_Térmica
5. THE Sistema SHALL incluir testes automatizados para Fila_Impressão validando prevenção de duplicações
6. THE Sistema SHALL incluir testes automatizados para retry de impressão em caso de falha temporária
7. THE Sistema SHALL incluir testes automatizados para reimpressão com marcação de segunda via
8. THE Sistema SHALL incluir testes automatizados para operação em modo demonstração vs modo impressora real
9. THE Sistema SHALL incluir testes de propriedade round-trip para Parser e Pretty_Printer de comandas

