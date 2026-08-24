# Biota-Geom — Backlog de Requisitos

Repositório de planejamento do sistema Biota-Geom: reúne épicos, user stories e a divisão em sprints. Não contém código — as tasks de implementação (fullstack) vivem em [`frontend-biota-geom`](https://github.com/biota-geom/frontend-biota-geom) e [`backend-biota-geom`](https://github.com/biota-geom/backend-biota-geom), referenciando as US daqui via `Closes biota-geom/planning#N`.

Time de execução: estudantes de 3º-5º semestre. Escopo pensado pra evitar tasks grandes/complexas e pra não exigir tasks de integração separadas (cada task já é fullstack).

## Modelo de dados — decisões-chave

- **Cliente**: o tenant do sistema. É a consultoria ambiental que assina o Biota-Geom (autocadastro com e-mail/senha, um login por Cliente, sem gestão de múltiplos usuários internos no MVP).
- **Empresa**: unidade operacional atendida por um Cliente (CNPJ, segmento, endereço, status). Um Cliente tem N Empresas. Toda Empresa/licença/obrigação/documento pertence a exatamente um Cliente — isolamento total entre tenants.
- **Parâmetro GRI**: lista padrão global de categorias (Resíduos, Emissões, Água, etc.); parâmetros customizados criados por um Cliente ficam privados a ele.
- **Licença Ambiental**: pertence a uma Empresa (tipo, número, órgão emissor, emissão, validade, PDF). Status calculado automaticamente pela data de validade.
- **Condicionante = Obrigação Ambiental**: mesma entidade, duas telas. Nasce vinculada a uma Licença (nome, descrição, categoria ligada ao GRI, vencimento). Status calculado automaticamente.
- **Indicador ESG**: registro mensal por Empresa (Água, Resíduos, Nº de Funcionários, Energia), alimenta a comparação "vs mês anterior" no Painel de Controle.
- **Documento**: arquivo anexado a uma Empresa (contratos, laudos, certidões), com exclusão lógica.
- **Legislação (Radar)**: cadastrada por um segundo ator, **Staff Biota-Geom** (login próprio, separado do Cliente). Impacto é modelado por Empresa, mas inicialmente todas recebem o mesmo valor global cadastrado junto com a legislação; análise técnica é um texto único global.

## Regras transversais de status (antecedência)

Duas constantes globais fixas no sistema, sem tela de configuração no MVP:

| Entidade | Regular | Atenção | Vencida / Risco |
|---|---|---|---|
| Licença | > 30 dias do vencimento | ≤ 30 dias do vencimento | já vencida |
| Condicionante / Obrigação | > 30 dias do vencimento | entre 8 e 30 dias | ≤ 7 dias **ou** já vencida |

**% Conformidade** = obrigações com status "Regular" / total de obrigações **ativas** da Empresa (exclui obrigações de licenças já Renovadas/Arquivadas).

**Cores do card de Empresa (US04)**: 🟢 ≥95% · 🟠 70–94% · 🔴 <70% (inferido do mockup, ajustável).

## Sprints (proposta)

Ordenado por dependência: primeiro o esqueleto (auth + Empresa), depois o motor de compliance (licenças/condicionantes/obrigações, o core do produto), depois as features de suporte, e por último o Radar (mais isolado do resto, e onde ficam os itens de menor prioridade).

### Sprint 1 — Fundação: acesso e cadastro de Empresas
US09, US10, US01, US02, US03, US05, US06, US07

### Sprint 2 — Motor de compliance: licenças, condicionantes e obrigações
US14, US16, US17, US19, US20, US21, US22, US23, US04

### Sprint 3 — Documentos, ESG, dashboard e relatórios
US18, US08, US11, US12, US13, US15

### Sprint 4 — Radar de Legislação e stretch goals
US26, US27, US28, US24, US25, US29

> Nota: US26–28 (feed/impacto/análise) dependem de ter alguma legislação cadastrada. Se US24/US25 (área admin do Staff) não couberem no tempo da Sprint 4, o time deve popular algumas legislações via seed/fixture direto no banco pra não travar a demonstração dessas três US.

**Itens de baixa prioridade / stretch** (fazer por último, só se sobrar tempo): US15, US24, US25, US29.

---

## Épico 1 — Gestão de Empresas e Parâmetros (CRM Básico)

### US01 — Cadastro de Empresas
**História:** Como Cliente autenticado na plataforma, eu quero cadastrar manualmente os dados das empresas que atendo (unidades/filiais), para gerenciar as obrigações legais de cada uma de forma individualizada.

**Critérios de Aceite:**
- O sistema deve permitir que o Cliente autenticado cadastre uma nova Empresa, vinculada automaticamente à sua conta (nenhuma outra consultoria pode ver essa Empresa).
- O cadastro deve incluir: Nome da Unidade, CNPJ, Segmento (ex: Siderurgia, Mineração, Agronegócio) e Endereço completo.
- Toda Empresa nasce com Status "Ativo" por padrão.
- O sistema deve impedir cadastro de CNPJ duplicado *dentro da mesma conta de Cliente* (a checagem de duplicidade é só contra as próprias Empresas do Cliente logado, não contra a base inteira).

### US02 — Parametrização de Atividades Baseada no GRI
**História:** Como Cliente, eu quero vincular os tipos de atividade de cada Empresa que atendo a parâmetros padronizados (ex: Resíduos, Emissões, Água), baseados na diretriz GRI, para evitar campos duplicados ou "lixo" no sistema.

**Critérios de Aceite:**
- Lista pré-cadastrada (seed) de parâmetros GRI padrão, disponível para todos os Clientes.
- O Cliente vincula um ou mais parâmetros da lista padrão a cada Empresa.
- Parâmetro customizado criado pelo Cliente (quando não existe um adequado na lista padrão) fica privado — só visível/aplicável às Empresas daquele Cliente.

### US03 — Visão Geral da Carteira de Empresas
**História:** Como Cliente, eu quero visualizar um painel inicial ("Empresas Cadastradas") com todas as minhas Empresas registradas em formato de cards, para acessar facilmente o painel de controle individual de cada uma.

**Critérios de Aceite:**
- A tela lista somente as Empresas vinculadas ao Cliente autenticado.
- Cabeçalho exibe o total de Empresas registradas daquele Cliente.
- Cada card exibe Nome da Unidade, Segmento e Localização.
- Cada card exibe uma etiqueta de Status ("Ativo" ou "Inativo").

### US04 — Resumo de Conformidade e Alertas por Card
**História:** Como Cliente, eu quero ver um resumo rápido das métricas principais de cada Empresa diretamente no seu card, para identificar rapidamente quais unidades precisam de atenção técnica urgente.

**Critérios de Aceite:**
- Cada card exibe 4 métricas: **Licenças** (total cadastrado), **Conformidade** (%), **Atenção** (contagem) e **Vencido** (contagem).
- Cor da Conformidade: 🟢 ≥95%, 🟠 70–94%, 🔴 <70%.
- % Conformidade segue a fórmula canônica (obrigações Regular / total ativas).
- Card exibe a data da "Última atualização" da Empresa.

### US05 — Busca Textual e Filtros de Listagem
**História:** Como Cliente, eu quero buscar empresas digitando textos e combinando filtros suspensos, para localizar rapidamente uma unidade específica em uma carteira volumosa.

**Critérios de Aceite:**
- Busca textual por nome da filial, estado ou segmento.
- Filtros dropdown por Segmento (Todos, Mineração, Agronegócio, ...) e Status (Ativos, Inativos).
- Listagem atualizada em tempo real ou por submissão dos filtros.

### US06 — Acesso Rápido ao Cadastro de Nova Empresa
**História:** Como Cliente, eu quero ter um botão principal e de fácil acesso ("+ Nova Empresa") na listagem geral, para iniciar o cadastro de uma nova Empresa.

**Critérios de Aceite:**
- O clique direciona para a tela/modal de criação de nova Empresa (US01 + US02).

### US07 — Acesso aos Detalhes da Unidade (Drill-down)
**História:** Como Cliente, eu quero clicar em "Ver detalhes" no card de uma Empresa, para ser direcionado ao painel específico daquela unidade.

**Critérios de Aceite:**
- O clique carrega o escopo da Empresa selecionada, levando ao Painel de Controle (US12) restrito aos dados daquela Empresa.

### US08 — Repositório Central de Documentos da Unidade
**História:** Como Cliente, eu quero anexar e armazenar documentos diversos da Empresa (contratos, laudos, certidões, fotos, etc.) em um repositório centralizado por unidade, para manter um arquivo digital organizado e de fácil consulta.

**Critérios de Aceite:**
- Upload de múltiplos arquivos por Empresa, com validação de tamanho e formato (PDF, JPG, PNG, DOCX até 20MB por arquivo).
- Campos obrigatórios por documento: Nome/Título, Tipo/Categoria (Contrato, Laudo, Certidão, Outros) e Data do Documento.
- Repositório acessível a partir do painel de detalhes da Empresa.
- Preview, download e exclusão lógica dos arquivos, com histórico de quem enviou e quando.
- Exclusão lógica: o arquivo some da listagem normal mas fica no banco com data de exclusão pra auditoria — sem tela de "lixeira"/restauração no MVP.

---

## Épico 2 — Governança e Acessos

### US09 — Cadastro de Cliente (Sign up)
**História:** Como uma consultoria ambiental interessada na plataforma, eu quero criar minha própria conta informando os dados básicos da consultoria e uma senha, para começar a usar o sistema e cadastrar minhas empresas clientes.

**Critérios de Aceite:**
- Tela de cadastro pede Nome da consultoria, E-mail (login) e Senha (com confirmação).
- Impede cadastro com e-mail já existente, com erro claro.
- Após cadastro bem-sucedido, autentica automaticamente e redireciona para "Empresas Cadastradas" (vazia).
- Senha com requisito mínimo (ex: 8 caracteres).
- Sem verificação de e-mail no MVP (login liberado na hora).

### US10 — Autenticação de Usuário (Login)
**História:** Como Cliente, eu quero acessar a plataforma inserindo meu e-mail e senha, para gerenciar de forma restrita o portfólio de licenças das empresas que atendo.

**Critérios de Aceite:**
- Autenticação com sucesso direciona para "Empresas Cadastradas".
- E-mail/senha incorretos: mensagem de erro genérica ("E-mail ou senha inválidos"), sem revelar se o e-mail existe.
- Fluxo de "Esqueci minha senha" com link seguro de redefinição por e-mail.
- Sessão expira automaticamente após período predefinido de inatividade.
- Isolamento multi-tenant: um Cliente nunca acessa dados de Empresas de outro Cliente.

---

## Épico 3 — Dashboards, KPIs da Biota-Geom e Relatórios para Auditoria e Órgãos

### US11 — Cadastro Mensal de Indicadores ESG
**História:** Como Cliente, eu quero registrar mensalmente os indicadores operacionais da minha Empresa (Consumo de Água, Resíduos Gerados, Nº de Funcionários, Consumo de Energia), para acompanhar a evolução desses dados e alimentar o Painel de Controle.

**Critérios de Aceite:**
- Registro de um valor mensal por indicador (Água em m³, Resíduos em toneladas, Nº de Funcionários, Energia em kWh) por Empresa.
- Só um registro por Empresa/mês/indicador — salvar de novo no mesmo mês atualiza o existente, não duplica.
- Valores alimentam o Painel de Controle (US12), incluindo variação % vs mês anterior.
- Histórico/gráfico de evolução completo fica fora do MVP — só a comparação com o mês imediatamente anterior é exigida.

### US12 — Painel de Controle da Empresa (Dashboard Situacional)
**História:** Como Cliente, eu quero visualizar um painel de controle centralizado ao entrar em uma Empresa, reunindo licenças, obrigações, legislação e indicadores ESG num só lugar, para ter um panorama rápido da saúde ambiental daquela unidade.

**Critérios de Aceite:**
- Cabeçalho com contadores rápidos: Licenças Ativas e Pendências.
- Bloco Licenças Ambientais: contagem por status (Regulares/Atenção/Vencidas) + lista resumida das mais críticas, link "Ver detalhes" pro Painel de Licenças completo (US19/US20).
- Bloco Obrigações Ambientais: barra de Conformidade Geral + lista resumida das mais críticas, link pro Monitor de Obrigações completo (US21/US22/US23).
- Bloco Radar de Legislação: lista resumida das leis mais recentes, link pro Radar completo (US26/US27/US28).
- Bloco Indicadores ESG: grid com os 4 KPIs e variação % vs mês anterior (US11).
- Blocos-resumo mostram só os itens mais críticos, não a lista inteira.

### US13 — Geração de Relatórios Executivos
**História:** Como Cliente, eu quero exportar os dados de uma Empresa na forma de um relatório executivo em PDF, para enviar formalmente aos clientes e órgãos ambientais.

**Critérios de Aceite:**
- Relatório gerado por Empresa (uma de cada vez).
- PDF contém só o resumo de KPIs (mesmos indicadores do Painel de Controle) — sem tabelas detalhadas de licenças/obrigações.
- Reflete os dados mais atualizados no momento da geração.

---

## Épico 4 — Motor de Condicionantes, Vencimentos e Metas Ligado à Licença

### US14 — Cadastro Manual de Licença Ambiental
**História:** Como Cliente, eu quero cadastrar manualmente uma licença ambiental de uma Empresa, informando seus dados principais e anexando o documento PDF, para manter o registro legal formalizado no sistema.

**Critérios de Aceite:**
- Campos: Tipo de Licença, Número, Órgão Emissor, Data de Emissão, Data de Validade, upload do PDF.
- Status calculado automaticamente pela Data de Validade (ver tabela de antecedência acima).
- Licença vinculada à Empresa selecionada.

### US15 — Auto-preenchimento de Licença via IA *(stretch)*
**História:** Como Cliente, eu quero que, ao subir o PDF da licença, o sistema tente extrair automaticamente os dados e sugerir as condicionantes descritas no documento, para acelerar o cadastro mesmo sem um padrão fixo de layout.

**Critérios de Aceite:**
- Ao anexar o PDF (fluxo da US14), o sistema envia o documento a um modelo de IA e recebe um JSON estruturado com os campos da licença + lista de condicionantes candidatas.
- Campos extraídos preenchem o formulário automaticamente mas ficam **editáveis** — o Cliente revisa e confirma antes de salvar, nunca salva direto sem revisão.
- Se a extração falhar/vier incompleta, o formulário continua disponível pra preenchimento 100% manual (US14 é o fallback).
- Nota técnica: usar um LLM direto sobre o PDF (ex: API da Anthropic/Claude, que lê PDF nativamente, inclusive digitalizado) em vez de uma pipeline de OCR + regex — lida muito melhor com documentos sem padrão fixo entre órgãos ambientais diferentes.

### US16 — Desmembramento de Licenças em Condicionantes
**História:** Como Cliente, eu quero vincular múltiplas condicionantes a uma licença específica, para desmembrar o documento jurídico em tarefas acionáveis e individualizadas.

**Critérios de Aceite:**
- A tela de detalhes da licença permite adicionar "N" condicionantes filhas.
- Cada condicionante tem nome, descrição, categoria (ligada à parametrização GRI da US02) e vencimento.
- Condicionante = Obrigação Ambiental (mesma entidade — toda condicionante criada aqui aparece automaticamente no Monitor de Obrigações, Épico 6).

### US17 — Parametrização de Vencimentos e Metas
**História:** Como Cliente, eu quero atribuir datas de vencimento às condicionantes, para que o sistema tenha os dados necessários para calcular o prazo de expiração.

**Critérios de Aceite:**
- O sistema bloqueia o salvamento de uma condicionante sem uma data limite válida.

### US18 — Renovação de Licença e Manutenção de Histórico
**História:** Como Cliente, eu quero registrar a renovação de uma licença (prestes a vencer ou já vencida) enviando o novo documento emitido pelo órgão, para atualizar os prazos legais sem perder o histórico da licença anterior.

**Critérios de Aceite:**
- **Status Histórico:** permite alterar o status da licença antiga de "Ativa"/"Vencida" para "Renovada/Arquivada", cessando os alertas daquele documento.
- **Proteção contra Sobrescrita:** a nova licença gera um novo registro (ou nova versão vinculada); proibido sobrescrever/deletar o PDF e os dados da licença antiga.
- **Migração de Condicionantes:** o fluxo permite copiar/reaproveitar as condicionantes/obrigações recorrentes da licença antiga para a nova.
- **Atualização do Dashboard/Conformidade:** a nova licença assume a posição "Ativa"; obrigações da licença antiga (Renovada/Arquivada) saem do cálculo de % Conformidade (US21).

---

## Épico 5 — Painel de Licenças Ambientais

### US19 — Indicadores de Status (Cards de Resumo)
**História:** Como Cliente, eu quero visualizar cards totalizadores no topo do Painel de Licenças de uma Empresa (Total, Regulares, Atenção, Vencidas), para ter uma visão gerencial imediata da saúde da carteira de licenças daquela unidade.

**Critérios de Aceite:**
- Contagem automática por status, calculada pela Data de Validade.
- "Atenção" via janela de antecedência fixa (30 dias) — constante global, sem tela de configuração.

### US20 — Listagem e Tabela de Licenças
**História:** Como Cliente, eu quero visualizar uma tabela com todas as licenças de uma Empresa (Tipo, Nº, Órgão, Emissão, Validade, Status, Nº de Condicionantes), para gerenciar os documentos de forma centralizada.

**Critérios de Aceite:**
- Ordenação prioriza licenças que exigem mais atenção (Vencidas/Atenção no topo).
- Coluna "Ações" permite acessar o PDF e o detalhe da licença (US16).
- Botão "+ Nova Licença" leva ao fluxo de cadastro (US14/US15).

---

## Épico 6 — Monitor de Obrigações Ambientais

### US21 — Barra de Progresso de Conformidade Geral
**História:** Como Cliente, eu quero visualizar uma barra de progresso indicando o percentual de "Conformidade Geral" de uma Empresa, para reportar facilmente a performance ambiental da unidade.

**Critérios de Aceite:**
- % Conformidade = obrigações "Regular" / total de obrigações ativas da Empresa (exclui obrigações de licenças Renovadas/Arquivadas, US18).

### US22 — Listagem de Obrigações por Nível de Risco
**História:** Como Cliente, eu quero ver uma lista em formato de cards detalhando cada obrigação e seu nível de risco visual (Regular, Atenção, Risco), para priorizar as ações da equipe técnica de forma clara.

**Critérios de Aceite:**
- Status calculado automaticamente pela Data de Vencimento: **Regular** (>30 dias), **Atenção** (entre 8 e 30 dias), **Risco** (≤7 dias **ou** já vencida).
- Obrigações em "Risco" destacadas visualmente (cor vermelha + ícone de alerta).

### US23 — Detalhamento do Escopo da Obrigação
**História:** Como Cliente, eu quero que cada card de obrigação exiba sua Categoria e a Data de Vencimento exata, para entender o escopo temático e o limite de tempo daquela pendência.

**Critérios de Aceite:**
- Categoria exibida está diretamente ligada à parametrização GRI (US02) daquela Empresa.

---

## Épico 7 — Radar de Legislação Ambiental

### US24 — Autenticação de Staff Biota-Geom *(baixa prioridade)*
**História:** Como Staff da Biota-Geom, eu quero acessar uma área administrativa separada com e-mail e senha internos, para gerenciar o conteúdo do Radar de Legislação sem misturar com o acesso dos Clientes.

**Critérios de Aceite:**
- Login separado do fluxo do Cliente (área/rota distinta).
- Só Staff autenticado acessa as telas de cadastro de legislação (US25).

### US25 — Cadastro de Legislação e Análise Técnica *(baixa prioridade)*
**História:** Como Staff da Biota-Geom, eu quero cadastrar uma nova legislação (data, título, resumo, órgão emissor, nível de impacto padrão, texto da análise técnica), para alimentar o Radar que os Clientes visualizam.

**Critérios de Aceite:**
- Formulário com Data, Título, Resumo, Órgão Emissor, Impacto padrão (Impacta sua operação / Impacto moderado / Baixo impacto) e texto da Análise Técnica.
- O nível de Impacto cadastrado aqui é o valor **global default** aplicado inicialmente a todas as Empresas (ver US27).

### US26 — Feed de Atualizações Regulatórias
**História:** Como Cliente, eu quero visualizar um feed (Radar) com as novas legislações ambientais (data, título, resumo, órgão emissor), para manter minha consultoria e meus clientes atualizados sobre as mudanças legais.

**Critérios de Aceite:**
- Lista ordenada cronologicamente (mais recente primeiro).

### US27 — Classificação de Impacto na Operação
**História:** Como Cliente, eu quero visualizar etiquetas indicando o nível de impacto de cada legislação para cada uma das minhas Empresas, para focar a atenção do time no que é crítico.

**Critérios de Aceite:**
- Estrutura de dados suporta impacto por Empresa (não só por legislação), mas **inicialmente todas as Empresas recebem o mesmo valor global** cadastrado na US25 — sem matching real ainda (isso é a US29, stretch).
- Tag "Novo" para legislações ainda não lidas/marcadas por aquele Cliente.

### US28 — Acesso à Análise Técnica
**História:** Como Cliente, eu quero clicar em "Ver análise técnica" em um item do radar, para ler o parecer sobre como aquela legislação afeta os processos práticos da minha Empresa.

**Critérios de Aceite:**
- Abre modal ou nova página com o texto da análise técnica única e global cadastrada na US25.

### US29 — Matching de Impacto Personalizado via IA *(stretch — última prioridade)*
**História:** Como Cliente, eu quero que o nível de impacto de cada legislação seja ajustado automaticamente conforme os parâmetros GRI da minha Empresa, para focar no que é realmente relevante pro meu negócio.

**Critérios de Aceite:**
- O sistema usa IA para comparar o conteúdo da legislação com os parâmetros GRI de cada Empresa e sobrescrever o valor padrão (US27) quando fizer sentido.
- Recurso opcional — só implementado se sobrar tempo após todo o resto do backlog.
