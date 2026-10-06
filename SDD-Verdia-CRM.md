> **Plano vigente:** [MVP com ferramentas gratuitas](MVP-Gratuito-Verdia.md). O SDD abaixo é a referência ampliada de evolução.

# Verdia — CRM e gestão clínica

**Software Design Document · Versão 1.0 · 5 de outubro de 2026**

**Status:** proposta técnica para validação de produto e planejamento de desenvolvimento. **Mercado inicial:** Brasil. **Nome provisório:** Verdia. **Responsáveis a designar:** proprietário do produto, líder técnico e responsável clínico. Este documento especifica o sistema; não representa um sistema já implementado, uma cotação comercial ou certificação de conformidade.

## 1. Decisão executiva

Desenvolver uma plataforma SaaS em nuvem para profissionais e clínicas de saúde, começando por biomedicina estética. O produto deve organizar o relacionamento com o paciente desde o cadastro e agendamento até o atendimento, retorno e acompanhamento. O núcleo é um CRM associado a um prontuário, com separação clara entre informações administrativas, clínicas e comerciais.

A arquitetura recomendada é um **monólito modular**, com aplicação web responsiva, API, banco PostgreSQL gerenciado, armazenamento privado de documentos e processamento assíncrono. Um mesmo produto atende várias clínicas, preservando o isolamento de dados. Essa estrutura permite pequenas entregas sem assumir, no início, o custo operacional de microsserviços.

**Primeiro marco de valor:** cadastro de pacientes, agenda e retornos em 6–8 semanas. **MVP clínico:** adicionar anamnese, histórico, finalização de registros e exportação em 10–14 semanas acumuladas. **Versão operacional ampliada:** fotos, financeiro e estoque em 23–32 semanas acumuladas. Os prazos são hipóteses para a equipe descrita na seção 18 e precisam ser recalibrados após a descoberta.

Segurança, isolamento, auditoria e restauração de backups são requisitos desde a primeira versão com dados reais. Funções que sugerem condutas ou interpretam imagens clinicamente ficam em uma trilha posterior, condicionada à avaliação técnica, clínica e regulatória.

## 2. Fontes, premissas e limites

### 2.1 Origem dos requisitos

Foram usados o pedido desta conversa e a transcrição disponível no chat **“Transcrição de fotos”**. A transcrição contém três fotos de anotações. Nesta elaboração, foi lido o texto transcrito; as palavras marcadas como ilegíveis não foram reconstruídas nem confirmadas nos manuscritos.

Classificação usada neste SDD: **U** = solicitado nesta conversa; **T** = presente na transcrição; **P** = proposta de arquitetura ou evolução. A transcrição é insumo de descoberta, não especificação clínica validada.

### 2.2 Premissas de dimensionamento

- Produto comercial para múltiplas clínicas; uma clínica também pode ser um profissional autônomo. Cada organização constitui um tenant, isto é, uma fronteira de dados e permissões.
- Piloto: 3–5 clínicas, até 25 usuários internos, 10 mil pacientes no total, 2 mil agendamentos/mês e 50 GB de anexos.
- Primeiro patamar comercial: até 50 clínicas, 250 usuários internos, 100 mil pacientes, 20 mil agendamentos/mês e 500 GB de anexos.
- Planejamento inicial em português brasileiro, moeda BRL e fusos IANA configuráveis por unidade. Armazenamento de instantes em UTC.
- Uso conectado à internet. A primeira versão não promete prontuário offline; rede indisponível deve ser sinalizada sem exibir uma falsa confirmação de salvamento.
- Os volumes são hipóteses de engenharia. Não são limites garantidos antes de testes de carga.

### 2.3 Fora do MVP

Aplicativos nativos, marketplace com compra automática, contabilidade fiscal completa, telemedicina, prescrição automatizada, interpretação diagnóstica de fotos, recomendações de doses ou planos anatômicos, recomendação automática de exames e integração com operadoras. Esses itens têm complexidade ou dependências que não devem atrasar a organização básica da clínica.

## 3. Usuários e jornadas

| Perfil | Necessidade principal | Limite de acesso padrão |
|---|---|---|
| Profissional assistencial | Atender, registrar, consultar evolução e organizar retorno | Pacientes sob sua responsabilidade ou compartilhados formalmente na clínica |
| Recepção | Cadastrar, agendar, confirmar presença e acompanhar retornos | Dados administrativos; sem anamnese, imagens ou evolução clínica |
| Gestor da clínica | Acompanhar operação, equipe, custos e receita | Indicadores e gestão; acesso clínico não é concedido apenas por ser proprietário |
| Financeiro | Contas a pagar/receber e conciliação | Valores e referências necessárias; sem conteúdo clínico |
| Estoquista/compras | Lotes, validade, movimentos e fornecedores | Estoque e compras; sem prontuário |
| Paciente/responsável | Preencher ficha e consultar orientações autorizadas | Apenas seus documentos e processos, em portal futuro |
| Suporte do SaaS | Resolver problemas operacionais | Metadados técnicos; acesso excepcional temporário e auditado |

**Jornada principal:** cadastrar paciente → verificar dados e preferências → agendar → confirmar → preencher anamnese → atendimento → finalizar evolução → registrar retorno → acompanhar situação → concluir acompanhamento.

**Jornada de retorno:** profissional define prazo ou data individualmente → sistema cria tarefa vinculada ao atendimento → recepção contata ou envia mensagem aprovada → paciente agenda → tarefa é concluída pelo evento correspondente. Uma tarefa de retorno não reserva horário automaticamente.

**Jornada de pós-procedimento:** profissional seleciona orientações e cronograma revisados → sistema cria tarefas e lembretes → respostas entram em fila de atendimento humano → equipe registra avaliação. Mensagens não equivalem a monitoramento de emergência.

## 4. Requisitos e rastreabilidade

| ID | Origem | Requisito e interpretação | Entrega |
|---|---|---|---|
| RF01 | U/T | Cadastro e busca de pacientes, responsáveis e contatos; alerta de possível duplicidade | F1 |
| RF02 | U | Agenda por profissional, unidade e sala; bloqueios, cancelamentos, faltas e reagendamento | F2 |
| RF03 | U/T | Lista de retornos com responsável, prazo, situação e histórico de contato | F2 |
| RF04 | U/T | Anamnese versionada, atendimentos, evolução e histórico longitudinal | F3 |
| RF05 | T | Fotos antes/depois com datas e visualização comparativa; interpretação por IA é outro requisito | F4; IA em F8 |
| RF06 | T | Voz e documentos geram rascunhos revisáveis de prontuário ou documento próprio | F8 |
| RF07 | T | Cronogramas, modelos de orientação e acompanhamento pós-procedimento | F4 |
| RF08 | T | Contas, pagamentos, vencimentos e fluxo de caixa por período e procedimento | F5 |
| RF09 | T | Cadastro e reconhecimento assistido de produtos; entradas, saídas, lotes e validade | F6; OCR em F8 |
| RF10 | T | Leitura de notas fiscais: XML quando disponível; OCR para imagem/PDF, com conferência | F6 manual/XML; F8 OCR |
| RF11 | T | Cadastro de fornecedores, cotações e comparação de custo total | F6/F7 |
| RF12 | T | Congressos, referências, técnicas e vídeos: biblioteca editorial com fontes e revisão | F7 |
| RF13 | T | Gráficos de resultados laboratoriais com unidade e referência da origem; sugestões clínicas separadas | F8 |
| RF14 | T | Projeções financeiras mensais/anuais por procedimento, com hipóteses visíveis | F5 básico; F8 estatístico |
| RF15 | U | Cloud, responsividade e segurança | Todas |
| RF16 | P | Exportação, auditoria, consentimentos, isolamento de clínicas e portabilidade | F0–F3 |
| RF17 | P | Portal do paciente e integrações externas | F7 |
| RF18 | P | Assinatura comercial, limites por plano e medição de consumo do SaaS | F7 |

**Pendências da transcrição:** os trechos “estímulos de acordo c/ o posto dele…”, “por quantos [ilegível]”, o verbo relacionado a “documento próprio” e “vídeos pós-[ilegível]” precisam de esclarecimento. Não se presume que “leitura de foto” signifique diagnóstico, nem que “contabilidade” implique escrituração fiscal. Para planejamento, foram separados comparação visual, gestão financeira e hipóteses futuras de inteligência.

## 5. Módulos e fronteiras

| Módulo | Responsabilidade | Interfaces principais |
|---|---|---|
| Identidade e organizações | Usuários, vínculos, papéis, unidades, sessões | Autorização para todos os módulos |
| Pacientes e relacionamento | Identificação, responsáveis, contatos, preferências | Agenda e prontuário referenciam o paciente |
| Agenda e retornos | Disponibilidade, reservas, tarefas e lembretes | Emite eventos de agendamento e cancelamento |
| Prontuário | Anamnese, atendimentos, evolução, documentos finalizados | Consome referências de paciente, procedimento e anexos |
| Mídia e documentos | Upload, quarentena, armazenamento e acesso | Entrega arquivos somente após autorização |
| Acompanhamento | Planos revisados, tarefas e respostas | Integra agenda, atendimento e comunicação |
| Financeiro | Recebíveis, despesas, parcelas, pagamentos e estornos | Vincula procedimento sem copiar narrativa clínica |
| Estoque e compras | Produtos, lotes, movimentos, cotações e fornecedores | Consumo vinculado ao atendimento e lançamento financeiro |
| Comunicação | Preferências, modelos, envio, retorno e falhas | Adaptadores para e-mail, WhatsApp e SMS |
| Inteligência assistida | Transcrição, extração e rascunhos | Nunca finaliza prontuários ou executa condutas autonomamente |
| Plataforma | Auditoria, cobrança SaaS, exportação, limites e operação | Serviços internos e painel restrito |

Cada módulo mantém suas regras e serviços de aplicação. A escrita em tabelas de outro módulo ocorre pela interface desse módulo, mesmo compartilhando o banco. Relatórios usam consultas controladas ou projeções; não contornam permissões.

## 6. Modelos de desenvolvimento e decisões arquiteturais

### 6.1 Abordagem de trabalho

Aplicar descoberta orientada a jornadas, priorização MoSCoW e entregas verticais: cada incremento inclui interface, regra, persistência, autorização e verificação. Usar ciclos de duas semanas, demonstração com um profissional e revisão do backlog. Histórias seguem critérios verificáveis no formato “dado/quando/então”.

Aplicar DDD de forma leve para separar domínios; arquitetura em camadas para domínio, casos de uso e infraestrutura; modelo C4 para diagramas; registros ADR para decisões; threat modeling para riscos; DevSecOps para integrar segurança ao fluxo de entrega. Não é necessário implementar todos os padrões de DDD em cada cadastro.

### 6.2 ADRs iniciais

| Decisão | Escolha | Motivo e condição de revisão |
|---|---|---|
| ADR01 | Monólito modular + worker | Simplifica transações e implantação. Extrair serviço quando carga ou equipes independentes justificarem |
| ADR02 | PostgreSQL relacional | Relações e integridade fortes; JSONB só para formulários flexíveis e snapshots |
| ADR03 | Banco compartilhado com tenant_id e RLS | Custo inicial adequado. Banco dedicado será opção futura para contratos com isolamento adicional |
| ADR04 | Web responsiva/PWA | Uma base para celular, tablet e PC. Aplicativo nativo somente por necessidade comprovada |
| ADR05 | AWS, região São Paulo para dados centrais | Facilita governança de localização; não elimina análise de transferências de terceiros |
| ADR06 | Identidade gerenciada via OIDC | Reduz construção de autenticação própria; autorização continua na aplicação |
| ADR07 | Outbox transacional + fila | Evita perder lembretes após gravação e permite tentativas seguras |
| ADR08 | Prontuário finalizado imutável | Retificação por adendo preserva autoria e evolução |
| ADR09 | IA desacoplada por adaptador | Permite comparar fornecedores, limitar custos e bloquear funcionalidades sem afetar o CRM |

Não adotar Kubernetes, Kafka, banco vetorial separado ou event sourcing geral no início. Histórico clínico e movimentos financeiros/estoque precisam de rastreabilidade, mas isso não exige reconstruir todo o sistema por eventos.

## 7. Arquitetura cloud proposta

### 7.1 Fluxo lógico

```text
PC / tablet / smartphone
          │ HTTPS
          ▼
DNS → proteção WAF → balanceador
          │
          ├── Web Next.js / BFF ── provedor OIDC + MFA
          │           │
          │           ▼
          └────── API NestJS (módulos de domínio)
                       ├── PostgreSQL/RDS privado
                       ├── S3 privado (documentos e fotos)
                       └── Outbox → SQS → worker
                                          ├── mensagens / integrações
                                          ├── exportações / varredura
                                          └── IA assistida (fase futura)
Controles transversais: KMS, segredos, auditoria, métricas e backups
```

### 7.2 Implantação

Contas ou ambientes isolados para desenvolvimento, homologação e produção. Dados de teste sintéticos. Infraestrutura declarada como código, com redes públicas para o balanceador e privadas para aplicação e banco. Em produção comercial, serviços web/API distribuídos em duas zonas, com pelo menos duas réplicas de cada serviço que atende requisições; worker escalável pela idade da fila. Banco RDS Multi-AZ antes de oferecer o nível comercial de disponibilidade.

Para o piloto, usar recursos menores e, se aprovado no marco técnico, banco Single-AZ com backups e indisponibilidade planejada explícita. Isso reduz custo, mas não tem a mesma resiliência do desenho comercial. Nenhum ambiente pode expor o banco à internet.

S3 privado recebe uploads por autorização temporária. Arquivos novos ficam em quarentena até validação de tipo, tamanho e varredura. Acesso a originais e versões depende do prontuário e da organização. Recursos estáticos podem usar CDN; respostas clínicas usam `Cache-Control: no-store` e ficam fora de caches compartilhados.

Egress por NAT ou endpoints privados conforme serviço; evitar um NAT único como dependência de produção entre zonas. Decidir o conjunto de endpoints com uma estimativa de custo: endpoints também podem gerar cobrança. Segredos no Secrets Manager, chaves gerenciadas por KMS, certificados TLS e acesso administrativo com MFA e funções temporárias.

### 7.3 Ferramentas de implementação

| Camada | Ferramenta proposta | Uso |
|---|---|---|
| Interface | TypeScript, React e Next.js | Layout responsivo, formulários e BFF de sessão |
| Design system | CSS tokens, componentes acessíveis e Storybook | Consistência de telas e estados |
| API | TypeScript, NestJS, OpenAPI | Casos de uso, validação e contratos documentados |
| Dados | PostgreSQL; Prisma para acesso e migrações | SQL específico para RLS, constraints e concorrência |
| Identidade | Amazon Cognito via OIDC | MFA, recuperação e ciclo de credenciais |
| Execução | Docker, ECS Fargate e ECR | Web, API e tarefas assíncronas |
| Fila | SQS + fila de falhas; agendador periódico | Lembretes, exportação, OCR e integrações |
| Arquivos | S3, KMS e scanner de malware | Fotos, documentos e artefatos de exportação |
| Observabilidade | OpenTelemetry e CloudWatch | Métricas, logs sanitizados e rastreamento |
| Desenvolvimento | Git, GitHub, Actions, editor e gestor de issues | Revisão, automação e rastreabilidade |
| Infraestrutura | OpenTofu ou Terraform, escolha registrada em ADR | Ambientes reproduzíveis e revisão de mudanças |
| Qualidade | Vitest/Jest, Playwright, axe-core e k6 | Regras, jornadas, acessibilidade e desempenho |
| Segurança | Scanner de dependências, segredos, imagens e OWASP ZAP | Detecção contínua e teste dinâmico |
| Design | Figma ou Penpot | Protótipos, validação e documentação de componentes |

Fixar versões estáveis suportadas no início da implementação, com lockfiles e rotina de atualização. A escolha não depende de uma versão “mais recente” imutável. Next.js admite diferentes formas de implantação; NestJS oferece módulos para organizar responsabilidades. Fontes: [Next.js](https://nextjs.org/docs/app/getting-started/deploying) e [NestJS](https://docs.nestjs.com/modules).

## 8. Modelo de dados

### 8.1 Entidades principais

Todas as entidades de negócio têm UUID, tenant_id, timestamps e autoria quando aplicável. Tabelas globais de identidade e catálogos explicitamente públicos são exceções documentadas.

| Entidade | Campos essenciais e relações |
|---|---|
| Organization / Unit | Nome, situação, fuso, configurações; organização 1:N unidades |
| User / Membership | Identidade externa; usuário N:N organizações por vínculo e papéis |
| Professional | Vínculo, conselho/registro, habilitações declaradas e unidades |
| Patient / Guardian | Nome, nascimento, contatos; CPF opcional; responsável e relação |
| CareTeamAssignment | Paciente, profissional/equipe, vigência e motivo do acesso |
| ContactPreference / Consent | Finalidade, canal, documento/versão, evidência, coleta e revogação |
| ProcedureCatalog | Nome, duração sugerida, preço de referência, versão e situação |
| Appointment / Reservation | Paciente, unidade, profissional, recurso, início/fim e situação |
| ReturnTask | Paciente, atendimento de origem, prazo, responsável e conclusão |
| Encounter | Paciente, profissional, agendamento opcional e situação |
| FormTemplateVersion / FormResponse | Esquema versionado, respostas JSONB e referência imutável à versão |
| ClinicalEntry / Addendum | Conteúdo, autor, instante, versão, finalização, hash e referência original |
| ProcedureExecution | Atendimento, procedimento, região registrada, produto/lote e quantidade informada |
| Attachment / Photo | Chave privada, checksum, MIME, tamanho, status de varredura, captura e finalidade |
| FollowUpPlan / Task | Modelo/versão, paciente, agenda, responsável e respostas |
| Receivable / Payable / Payment | Competência, vencimento, centavos, moeda, parcelas e conciliações |
| Product / Batch / StockMovement | Unidade de medida, lote, validade, entrada/saída, custo e referência |
| Supplier / Quotation / Purchase | Identificação, itens, frete, prazo e estado de aprovação |
| InvoiceImport | Documento original, chave quando houver, resultado extraído e conferência |
| Notification / IntegrationEvent | Destino mínimo, template, chave idempotente e estado externo |
| AuditEvent / OutboxEvent | Ator, recurso, ação, resultado, timestamp e identificador de correlação |
| Subscription / Usage | Plano SaaS, franquias, período e consumo; distinto do financeiro da clínica |
| AIJob | Tipo, versão de modelo/prompt, referências, custo, resultado e revisão humana |

**Relações centrais:** organização → pacientes → atendimentos → registros e anexos. Agenda referencia paciente e recursos. Atendimento pode originar retorno, consumo de estoque e recebível. Um pagamento liquida total ou parcialmente um recebível; não cria receita duplicada. Um produto tem vários lotes, cada um com movimentos imutáveis.

### 8.2 Integridade e concorrência

- Chaves estrangeiras compostas por tenant_id e id impedem relacionar um atendimento de uma clínica a um paciente de outra.
- Índices iniciam por tenant_id nos caminhos comuns: paciente/nome, agenda/profissional/período, retorno/status/prazo, atendimento/paciente/data e lote/produto/validade.
- CPF não será chave primária e não será obrigatório para cadastrar. Duplicidade por nome, nascimento e contato exige conferência humana; fusão preserva referências e auditoria.
- Agenda usa reservas de intervalos com constraint de exclusão para recursos ocupados e estados ativos. Reagendamento libera e reserva dentro de uma transação. Testar dois usuários disputando o mesmo horário.
- Edições usam versão otimista (`version`/ETag); conflito gera resposta 409 e comparação. Não adotar “última gravação vence” em registros clínicos.
- Valores financeiros em centavos ou decimal exato; quantidades de estoque em decimal com unidade explícita. Nunca usar ponto flutuante para dinheiro.
- Estoque é calculado por movimentos, com controle transacional para impedir saída indevida e estornos vinculados ao movimento original.
- Finalização do prontuário grava snapshot, autoria e evidência de integridade. Correções posteriores são adendos. Hash isolado não equivale a assinatura digital juridicamente qualificada.

### 8.3 Isolamento de clínicas

O backend obtém o tenant do vínculo autenticado, nunca confia apenas no tenant_id enviado pelo navegador. A política RLS deve cobrir leitura e escrita, com `USING` e `WITH CHECK`. A função de conexão da aplicação não pode ser proprietária das tabelas, superusuária ou possuir BYPASSRLS; migrações usam credencial separada. Empregar FORCE RLS onde aplicável e contexto de tenant local à transação, evitando vazamento entre conexões do pool.

A mesma fronteira vale para arquivos, filas, caches, exportações e busca. Jobs revalidam organização e autorização na execução. A documentação do PostgreSQL registra exceções de RLS para proprietários e papéis privilegiados; por isso a configuração de papéis faz parte dos testes de segurança. [PostgreSQL — Row Security](https://www.postgresql.org/docs/17/ddl-rowsecurity.html).

## 9. Regras funcionais essenciais

### 9.1 Agenda e retornos

Situações: agendado → confirmado → em atendimento → concluído; cancelado e falta são estados terminais alternativos. Reagendamento mantém o histórico. Conclusão administrativa do horário não finaliza automaticamente o conteúdo clínico.

Considerar duração, intervalo de preparação, horário de funcionamento e bloqueios por profissional e sala. Sobrerreserva, se adotada futuramente, exige permissão e justificativa. Retornos têm estados pendente, contatado, agendado, concluído e dispensado com motivo. Cancelamento do retorno reabre a tarefa quando necessário.

### 9.2 Anamnese e prontuário

Templates por área/procedimento, revisados pelo responsável clínico. Campos tipados, condicionais e obrigatórios conforme template; distinguir “não”, “não informado” e “não se aplica”. Registrar mudanças de alergias e outras informações relevantes sem apagar histórico.

Permitir rascunho com salvamento no servidor, indicador de última gravação e recuperação após reconexão. Finalização é ato explícito do profissional e exige revisão. Modelos novos não alteram fichas antigas. Importações e transcrições identificam sua origem. Permitir exportação legível com datas, autores, adendos e anexos autorizados.

### 9.3 Fotos e documentos

Registrar data de captura e upload separadamente, região corporal, contexto e vínculo ao atendimento. Preservar original e derivadas; remover geolocalização de cópias de exibição quando desnecessária. Não aplicar filtros de embelezamento à evidência clínica. Comparação lado a lado e alinhamento manual devem informar diferenças de iluminação/posição.

Uso assistencial e divulgação de imagem são finalidades distintas. O acesso clínico não autoriza publicação comercial. Exportação ou compartilhamento gera auditoria. Downloads temporários usam links curtos, revogáveis e escopo mínimo; não enviar arquivos clínicos como anexos automáticos em canais de mensagem.

### 9.4 Financeiro e estoque

Separar receita prevista, faturamento e caixa recebido. Relatórios por procedimento devem expor regra de rateio quando houver pacote, desconto ou parcelamento. Pagamentos parciais, inadimplência, estornos e cancelamentos mantêm rastreabilidade.

Estoque registra lote e validade, perdas, devoluções e ajustes justificados. Bloquear uso de lote vencido/quarentenado e mostrar vínculo lote → atendimentos para rastreio. Conversões entre embalagem, volume e unidades de aplicação precisam de configuração revisada; o sistema não deduz equivalência clínica entre produtos.

Importação XML/OCR de nota gera uma conferência antes de movimentar estoque e financeiro. Não confundir reconhecimento de documento com emissão fiscal ou validação tributária. Comparar fornecedores pelo custo total, prazo, validade e procedência documentada; a compra exige decisão humana.

## 10. Contratos de API e processamento

API REST versionada em `/v1`, com OpenAPI gerada, validação no servidor e paginação por cursor nas listas extensas. Erros padronizados incluem código, mensagem segura e correlation_id, sem conteúdo clínico em logs. Autenticação por sessão de servidor no BFF; cookies Secure, HttpOnly e SameSite, proteção CSRF e verificação de origem. Tokens de acesso não ficam em localStorage.

| Operação | Contrato resumido | Controle decisivo |
|---|---|---|
| POST /v1/patients | Cria paciente com campos mínimos | Papel, tenant e alerta de duplicidade |
| POST /v1/appointments | Reserva período e recursos | Transação, idempotência e conflito 409 |
| PATCH /v1/appointments/{id} | Atualiza ou reagenda | If-Match/version e histórico |
| POST /v1/encounters | Abre atendimento | Vínculo assistencial e profissional |
| POST /v1/clinical-entries/{id}/finalize | Finaliza uma versão revisada | Autoria, permissão e versão atual |
| POST /v1/clinical-entries/{id}/addenda | Acrescenta retificação | Original permanece íntegro |
| POST /v1/attachments/upload-intents | Autoriza upload restrito | MIME, tamanho, finalidade e expiração |
| POST /v1/exports | Cria job de exportação | Reautenticação quando sensível e auditoria |
| POST /v1/webhooks/{provider} | Recebe evento de integração | Assinatura, anti-replay e deduplicação |

Um comando grava a alteração e um evento outbox na mesma transação. O publicador envia à fila; o consumidor tolera entregas repetidas. Usar chave idempotente por organização/operação e unicidade do evento externo. Não prometer execução “exatamente uma vez”.

Lembretes são tarefas persistidas no banco, selecionadas periodicamente pelo vencimento. Antes do envio, revalidar agendamento, consentimento/preferência, janela de envio e versão do modelo. Implementar tentativas com atraso exponencial, fila de falhas e reprocessamento auditado. Webhook repetido não duplica pagamento nem altera duas vezes o agendamento.

## 11. Segurança e privacidade

### 11.1 Controles obrigatórios

| Risco | Controle preventivo/detectivo | Evidência de aceite |
|---|---|---|
| Acesso entre clínicas | RLS, autorização por recurso e chaves compostas | Testes negativos em API, jobs, busca e anexos |
| Conta comprometida | MFA para equipe, recuperação segura, revogação e limites de tentativas | Sessão revogada perde acesso; recuperação auditada |
| Recepção lendo prontuário | Papéis + vínculo assistencial; negação por padrão | Matriz de permissões testada |
| Vazamento de foto | S3 privado, escopo mínimo, URL curta e log | URL expira e tenant incorreto é negado |
| Alteração de histórico | Registros finalizados imutáveis e adendos | Original não muda após correção |
| Upload malicioso | Quarentena, assinatura real de arquivo, limite e scanner | Arquivo não liberado antes da verificação |
| Ransomware/perda | Backups isolados, versões e restauração ensaiada | Recuperação cronometrada e reconciliada |
| Vazamento em telemetria | Redação de dados, sem payloads clínicos ou gravação de sessão | Inspeção de logs e traces |
| Abuso administrativo | MFA, acesso temporário, motivo e revisão | Auditoria exportável e alertas |
| Dependência vulnerável | Inventário, scanner, atualização e revisão de imagens | Sem vulnerabilidade crítica explorável aberta |

Criptografia em trânsito e em repouso, incluindo backups. Princípio do menor privilégio em IAM, banco e aplicação. Política de sessão proposta: bloqueio por 15 minutos de inatividade em telas clínicas, duração máxima configurada e reautenticação em ações sensíveis. Validar usabilidade e não descartar rascunhos já salvos.

Auditar leitura de prontuário e foto, finalização, exportação, permissão, acesso de suporte e alterações financeiras. Log contém identidade, ação, recurso, momento, resultado e motivo quando necessário; não copia a narrativa clínica. Proteger contra alteração com destino separado e controles de retenção.

Adotar OWASP ASVS 5.0 nível 2 como base de verificação, com controles adicionais por risco para dados clínicos. Isso é uma meta de engenharia, não uma certificação automaticamente obtida. [OWASP ASVS](https://github.com/OWASP/ASVS).

### 11.2 LGPD e governança

Dados de saúde são sensíveis. Mapear finalidade e hipótese legal por tratamento; não usar um consentimento genérico para todas as operações. A clínica tende a controlar o tratamento assistencial; o SaaS tende a atuar como operador nesse serviço, podendo ser controlador de atividades próprias, como faturamento. Formalizar os papéis contratualmente. Separar assistência, comunicação operacional, marketing e uso de imagens. [LGPD](https://planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709compilado.htm).

Manter inventário de dados, fornecedores e subprocessadores, registro das operações, canal de direitos e avaliação de impacto proporcional ao risco. Exportação, correção, restrição e exclusão passam por identificação do solicitante e análise de obrigações de guarda.

Preferir São Paulo para banco, objetos e backups centrais. Serviços de autenticação, suporte, telemetria e IA também entram no mapa de fluxo: usar cloud no Brasil não garante ausência de transferência internacional. Avaliar mecanismos contratuais aplicáveis antes de enviar dados a outro país. [ANPD — transferências internacionais](https://www.gov.br/anpd/pt-br/acesso-a-informacao/institucional/atos-normativos/regulamentacoes_anpd/resolucao-cd-anpd-no-19-de-23-de-agosto-de-2024).

Para incidentes com risco ou dano relevante, preparar processo de avaliação e comunicação pelo controlador. A orientação oficial informa prazo de três dias úteis, ressalvadas regras específicas. Definir alerta interno imediato ao responsável, preservação de evidências e cooperação do operador para cumprir o prazo aplicável. [ANPD — comunicação de incidentes](https://www.gov.br/anpd/pt-br/canais_atendimento/agente-de-tratamento/comunicado-de-incidente-de-seguranca-cis?sck=direto).

### 11.3 Guarda, validade documental e responsabilidade clínica

A Lei 13.787/2018 trata de digitalização e guarda de prontuários e prevê prazo mínimo de 20 anos a partir do último registro para possível eliminação de prontuários em papel e digitalizados. Isso não deve virar uma rotina automática indiscriminada: validar aplicação ao documento nato digital, normas profissionais e impedimentos de descarte. Digitalização e assinatura têm requisitos próprios. [Texto oficial da Lei 13.787/2018](https://portalsinan.saude.gov.br/images/documentos/Legislacoes/LEI_13787_27_DEZEMBRO_2018.pdf).

Antes do uso clínico, o responsável jurídico e o responsável clínico devem aprovar matriz de retenção por classe, procedimentos de assinatura, acesso de responsáveis, tratamento de menores e regras profissionais aplicáveis. Não aplicar normas de um conselho automaticamente a todas as profissões. A habilitação do usuário e o escopo de atuação devem ser verificados pela clínica; uma seleção no software não constitui autorização profissional.

Política técnica proposta: backup recuperável por 35 dias; logs técnicos por 30–90 dias conforme minimização; auditoria e documentos conforme matriz aprovada; áudio bruto de IA por até 7 dias após revisão, quando permitido, com expiração configurável. Guardas legais prevalecem. Desativação de conta não apaga prontuários automaticamente. Restauração de backup reaplica registros de exclusão/restrição para não reativar dados eliminados legitimamente.

## 12. Requisitos não funcionais e operação

| ID | Meta proposta | Forma de medição |
|---|---|---|
| RNF01 | Disponibilidade mensal de 99,5% no piloto; 99,9% na versão comercial | Sonda externa das jornadas críticas e relatório de incidentes |
| RNF02 | API p95 até 500 ms em leituras e 1 s em escritas usuais, sem upload/terceiros | Teste com volume de referência, consultas representativas e 100 sessões simultâneas |
| RNF03 | Tela principal útil em até 3 s em rede móvel de teste definida | Aparelhos intermediários e medição sintética; definir RTT/banda no ensaio |
| RNF04 | RPO até 15 min e RTO até 4 h para recuperação de dados na região | Teste trimestral de restore; medir banco, objetos e permissões |
| RNF05 | Perda regional: RPO 24 h / RTO 24–48 h como objetivo inicial | Cópia para região de contingência aprovada e exercício anual; não prometido antes da implantação |
| RNF06 | WCAG 2.2 AA nas jornadas essenciais | Automação + teclado, leitor de tela e verificação manual |
| RNF07 | Zero vazamento entre tenants nos cenários testados | Suíte de autorização em toda entrega |
| RNF08 | Exportação de prontuário sem perda de autoria, versão ou anexos | Reconciliação e revisão de amostra |

Os números são metas, não SLAs contratuais já demonstrados. 99,9% corresponde a aproximadamente 43 minutos de indisponibilidade em 30 dias; 99,5%, a 3 h 36 min. Definir janela, exclusões contratuais e forma de medição antes de vender um SLA.

Monitorar taxa de erro, latência, conexões do banco, armazenamento, idade da fila, atraso de lembretes, falhas de upload, negações anormais de acesso e custo por organização. Alertas críticos precisam de responsável e escala de atendimento compatível com o SLA; monitoramento sem capacidade de resposta não garante disponibilidade.

Backup do banco com recuperação pontual, versionamento dos objetos e cópia protegida contra credenciais de produção comprometidas. Manter manifesto de anexos e reconciliar arquivos órfãos/ausentes após restore. Para desastre regional, selecionar região secundária e formalizar eventual transferência internacional antes de ativar a cópia; até isso ocorrer, o risco regional permanece registrado.

Runbooks obrigatórios: indisponibilidade, falha de banco, restauração, fila represada, provedor de mensagem fora do ar, credencial comprometida, vazamento e reversão de release. Em indisponibilidade, orientar contingência da clínica e posterior transcrição auditada; não simular atendimento salvo.

## 13. Experiência responsiva e acessibilidade

Arquitetura de informação: **Hoje, Pacientes, Agenda, Retornos, Atendimento, Financeiro, Estoque e Configurações**, com opções exibidas conforme papel e plano. O painel “Hoje” mostra próximos horários, retornos vencidos e tarefas; dados clínicos detalhados só aparecem quando o usuário abre o atendimento autorizado.

| Dispositivo | Comportamento proposto |
|---|---|
| Smartphone, 360–767 px | Lista diária de agenda; navegação curta; formulário em etapas; ações principais ao alcance do toque |
| Tablet, 768–1199 px | Agenda diária/semanal simplificada; paciente e formulário em duas áreas quando houver espaço |
| Desktop, a partir de 1200 px | Menu lateral, agenda ampliada e painéis comparativos |

Evitar arrastar como único modo de reagendar; oferecer edição por formulário. Botões de ação com alvo preferencial de 44×44 px, foco visível, rótulos e mensagens de erro associadas aos campos. Não usar apenas cor para indicar atraso, alergia ou status. Contraste mínimo de texto normal 4,5:1 e de texto grande 3:1; validar estados de hover, desabilitado e foco. Referência: [WCAG 2.2](https://www.w3.org/TR/wcag/).

PWA instalável opcionalmente, com cache apenas do shell público e recursos estáticos. Sem prontuário persistido em service worker/IndexedDB no MVP. Sessões, formulários e prévias são limpos ao sair. Fluxos longos exigem estados claros de “salvando”, “salvo” e “falha ao salvar”.

## 14. Marca e identidade visual

### 14.1 Nome recomendado

**Verdia** — nome curto, associado a verde, vitalidade e organização do dia. Descritor: **CRM e gestão clínica**. Assinatura proposta: **“Cuidado que acompanha.”** É uma hipótese de marca, sem verificação de disponibilidade jurídica, domínio ou redes sociais. Antes da escolha final, fazer busca de anterioridade no INPI, domínio e conflitos no segmento.

Alternativas: **EloVita**, com foco em vínculo e continuidade, e **Núcleo Verde**, com foco em organização e acolhimento. São propostas criativas, também sem disponibilidade verificada.

### 14.2 Paleta proposta

| Papel | Cor | Código | Aplicação |
|---|---|---|---|
| Primária | Verde floresta | #145C45 | Botões principais, navegação e marca |
| Secundária | Verde profundo | #0B3D2E | Títulos e áreas de destaque |
| Apoio | Sálvia | #A8C5B5 | Superfícies secundárias e elementos gráficos |
| Fundo suave | Menta clara | #EAF4EE | Destaques leves e cartões |
| Fundo base | Branco quente | #F7FAF8 | Páginas e formulários |
| Texto | Grafite esverdeado | #1F2933 | Texto principal |
| Atenção | Âmbar escuro | #8A4B08 | Pendências, sempre com ícone/texto |
| Erro | Vermelho escuro | #B42318 | Falhas e ações destrutivas |

Usar branco sobre verde floresta nos botões; sálvia deve ser fundo ou ornamento, não texto pequeno em branco. Tipografia sans-serif legível, preferencialmente Inter ou família de sistema, corpo de pelo menos 16 px em formulários móveis.

### 14.3 Três conceitos de logomarca

1. **Verdia / folha em V:** duas folhas geométricas formam um V; comunica vitalidade sem prometer resultados estéticos. Principal recomendação para ícone de app e marca horizontal.
2. **EloVita / elos de cuidado:** dois arcos entrelaçados formam um elo aberto; simboliza acompanhamento e relação profissional–paciente.
3. **Núcleo Verde / pontos orbitais:** núcleo com três pontos conectados; remete à organização do cuidado, equipe e informação.

O documento visual acompanha estudos vetoriais desses conceitos. São esboços de direção, não identidades finais. Refinar a marca escolhida em versões monocromática, reduzida, horizontal e vertical; testar legibilidade a 24 px e pesquisar semelhanças com marcas existentes. Evitar símbolos protegidos e aparência de certificação médica.

## 15. Fases e pequenas entregas

As durações são sequenciais para tornar o orçamento comparável; algumas atividades de design e segurança podem ocorrer em paralelo dentro da mesma equipe. A complexidade cresce por dependência. A base de segurança vem primeiro mesmo exigindo trabalho especializado.

### F0 — Descoberta e fundação · 2 semanas

**Incremento 0.1:** revisar jornadas com biomédico e recepção; esclarecer ilegíveis; definir campos mínimos, matriz de papéis e dados sintéticos. Entregar backlog priorizado, protótipo das telas principais e ADRs.

**Incremento 0.2:** repositório, pipeline, homologação cloud, autenticação/MFA, organização, permissões, logs sanitizados e backup. **Aceite:** duas clínicas de teste não acessam dados uma da outra; login/recuperação funcionam; restore de teste documentado. Dependência: responsáveis de produto e clínico disponíveis.

### F1 — Cadastro de pacientes · 2 semanas

**Incremento 1.1:** cadastro mínimo, busca, edição, responsáveis e avisos de possível duplicidade. **Incremento 1.2:** preferências de contato, importação CSV com prévia/erros e exportação administrativa controlada.

**Aceite:** recepção cadastra e encontra um paciente em teste de uso; importação repetida não duplica registros sem aviso; papéis e auditoria verificados. A liberação com dados reais depende dos controles de F0 e da governança correspondente.

### F2 — Agenda e retornos · 2–4 semanas

**Incremento 2.1:** agenda diária, recursos, bloqueios e reserva transacional. **Incremento 2.2:** confirmação manual, cancelamento, falta e reagendamento. **Incremento 2.3:** lista de retornos e tarefas de contato, com lembrete interno ou e-mail operacional quando contratado.

**Aceite:** tentativa simultânea de reservar o mesmo recurso retorna conflito; cancelamento remove lembrete pendente; tarefa de retorno permanece rastreável. **Marco:** piloto administrativo em 6–8 semanas acumuladas.

### F3 — Prontuário e anamnese · 4–6 semanas

**Incremento 3.1:** catálogo de procedimentos e templates de anamnese versionados. **Incremento 3.2:** atendimento, evolução e salvamento de rascunhos. **Incremento 3.3:** finalização, adendos, exportação e trilha de leitura/acesso. Incluir assinatura apropriada ao fluxo definido na análise documental; integração externa pode ser necessária nesta fase se exigida.

**Aceite:** atualizar template não altera resposta antiga; recepção não acessa narrativa clínica; adendo preserva original; exportação confere com fonte; responsável clínico aprova fluxo documental e regras de assinatura. **Marco:** MVP clínico em 10–14 semanas acumuladas.

### F4 — Fotos e acompanhamento · 3–4 semanas

**Incremento 4.1:** anexos com quarentena e galeria autorizada. **Incremento 4.2:** comparação visual antes/depois e vinculação ao atendimento. **Incremento 4.3:** modelos revisados de orientação, cronograma e fila de respostas.

**Aceite:** original preservado; arquivo malicioso bloqueado; link expirado não funciona; uso comercial depende de autorização específica; respostas são atribuídas a uma pessoa responsável. Acumulado: 13–18 semanas.

### F5 — Financeiro operacional · 4–6 semanas

**Incremento 5.1:** contas e parcelas com vencimento. **Incremento 5.2:** pagamentos, conciliação manual, estornos e alertas. **Incremento 5.3:** fluxo de caixa mensal/anual, receita por procedimento e projeção determinística de recebíveis.

**Aceite:** pagamento parcial não quita saldo restante; estorno não apaga histórico; totais do relatório reconciliam com lançamentos; permissões mantêm segregação. Acumulado: 17–24 semanas. Não inclui escrituração fiscal.

### F6 — Estoque e fornecedores · 6–8 semanas

**Incremento 6.1:** produtos, unidades, fornecedores, entradas e saídas. **Incremento 6.2:** lotes, validade, perdas e consumo por atendimento. **Incremento 6.3:** importação de XML compatível com conferência e prevenção de duplicidade. **Incremento 6.4:** cotações e comparação de custo total.

**Aceite:** saldo reconcilia com movimentos; duas saídas simultâneas não ultrapassam estoque; lote vencido bloqueado; nota repetida identificada; é possível localizar pacientes ligados a um lote. Acumulado: 23–32 semanas.

### F7 — Integrações e expansão · 4–8 semanas por pacote priorizado

**Pacote A:** WhatsApp oficial e portal para confirmação/ficha. **Pacote B:** pagamentos e conciliação por webhook. **Pacote C:** agenda externa, exportação contábil e biblioteca de referências/congressos. **Pacote D:** planos SaaS, cobrança e medição de consumo.

Cada pacote é contratado/priorizado separadamente; a duração não cobre todos simultaneamente. **Aceite comum:** autenticação e webhooks validados, eventos duplicados inofensivos, falha do fornecedor não bloqueia atendimento e dados compartilhados minimizados.

### F8 — Inteligência assistida · 6–12 semanas por capacidade

**Sequência:** OCR assistido → ditado e transcrição → rascunho estruturado → resumo com referência à fonte → gráficos laboratoriais e projeções estatísticas → eventual pesquisa de interpretação de imagens/recomendação clínica.

**Aceite:** avaliação em corpus autorizado, campos críticos conferidos, origem rastreável, custo limitado, possibilidade de desligar a função e nenhum registro finalizado sem revisão. Funções clínicas avançadas têm cronograma próprio após avaliação regulatória; não estão incluídas automaticamente nas 6–12 semanas.

## 16. Modelos de inteligência e limites de automação

| Capacidade | Modelo/técnica | Dados e validação |
|---|---|---|
| Reconhecer nota e documento | Parser XML primeiro; OCR quando necessário | Conferir emitente, itens, valores e duplicidade antes de gravar |
| Ditado clínico | Modelo de reconhecimento de fala em português | Medir erro geral e erro em termos críticos; revisar nomes, valores e negações |
| Estruturar prontuário | Modelo de linguagem com saída em esquema validado | Somente rascunho, com evidência de origem e campos desconhecidos explícitos |
| Buscar referências | Busca textual; RAG sobre coleção aprovada | Citar trecho, fonte, data e versão; respeitar isolamento por clínica |
| Comparar fotos | Visualização padronizada primeiro | IA quantitativa somente após validação específica; não inferir diagnóstico automaticamente |
| Projetar caixa | Regras de recebíveis; depois modelos de séries temporais | Backtesting, erro e intervalos; avaliar sazonalidade e amostra insuficiente |
| Gráficos laboratoriais | Série temporal com unidade e intervalo do laboratório | Conversões explícitas; não usar “parâmetro ideal” universal |

Não é necessário treinar um modelo próprio para iniciar. Selecionar fornecedor por avaliação de português, qualidade, privacidade, região, retenção, custo e contrato. Modelos específicos só serão fixados após esse benchmark; não se presume que qualquer serviço permita processar dados de pacientes nas condições necessárias.

Criar conjunto de avaliação sintético ou autorizado, representando sotaques, ruído, documentos ruins e negações. Para passagem a piloto de rascunhos, exigir ausência de erros críticos não detectados na amostra de validação e 100% de conferência humana dos campos críticos. Essa aprovação da amostra não é garantia estatística de erro zero em produção. Registrar edições e taxa de rejeição, sem expor dados em telemetria.

Documentos e áudios são conteúdo não confiável: instruções embutidas não podem comandar ferramentas, exportar dados ou mudar permissões. A IA não pode comprar produtos, enviar mensagens clínicas, prescrever ou finalizar registros por conta própria. Minimizar dados enviados, não usar conteúdo de pacientes para treinamento próprio por padrão e aplicar orçamento por tenant.

Software com finalidade de diagnóstico, monitoramento ou recomendação terapêutica pode exigir enquadramento como dispositivo médico. Revisão humana, sozinha, não determina dispensa regulatória. Definir finalidade de uso e avaliar enquadramento antes de implementar ou divulgar essas capacidades. [Anvisa — software como dispositivo médico](https://www.gov.br/anvisa/pt-br/assuntos/noticias-anvisa/2022/software-como-dispositivo-medico-perguntas-e-respostas/).

## 17. Integrações futuras

| Serviço | Benefício | Dependências e cuidados | Prioridade |
|---|---|---|---|
| WhatsApp Business Platform | Confirmação e lembretes | Modelos aprovados, preferências, opt-out, custos e entrega assíncrona | Alta após F3 |
| E-mail transacional | Convites, recuperação e lembretes | SPF/DKIM/DMARC, reputação e conteúdo mínimo | Alta |
| Google/Microsoft Calendar | Visão externa de disponibilidade | OAuth, sincronização e conflitos; enviar bloqueios genéricos sem narrativa clínica | Média |
| PSP com Pix/cartão | Cobrança e conciliação | Checkout hospedado, webhooks, estornos e taxas; não armazenar cartão | Alta após F5 |
| Assinatura eletrônica/digital | Formalização de documentos | Modalidade adequada, evidência e validação jurídica do fluxo | Conforme F3 |
| Contabilidade/ERP | Evitar digitação duplicada | Exportação mapeada e validação pelo contador | Média |
| Serviços fiscais | NFS-e e importação fiscal | Município, provedor, contrato e regras tributárias | Média |
| Laboratórios/FHIR | Receber resultados estruturados | Acordo com origem, identificação e mapeamento de unidades | Posterior |
| Fornecedores | Catálogo, cotação e pedido assistido | API/licença, qualidade da fonte e aprovação de compra | Posterior |
| Eventos e biblioteca | Congressos, vídeos e referências atualizadas | Curadoria, direitos de uso, fonte e data de revisão | Média |

FHIR é uma opção de interoperabilidade quando o parceiro oferecer suporte; não pressupõe conexão automática com laboratórios, RNDS ou outros sistemas. Evitar scraping como dependência central. “Fornecedor confiável” deve ter critérios documentados e evidências, não selo automático gerado por IA.

## 18. Equipe, esforço e investimento de desenvolvimento

Equipe-base proposta: 1 líder técnico/backend, 1 desenvolvedor full-stack, designer de produto parcial, QA parcial, DevOps/segurança parcial e profissional clínico como especialista de domínio. Um responsável de produto decide prioridades. Jurídico/privacidade entra em marcos específicos. O tempo clínico e de homologação precisa estar reservado.

Estimativas próprias de planejamento, em horas totais de equipe; não são pesquisa salarial ou proposta de fornecedor. Taxa média hipotética de **R$ 140–220/h**, incluindo a composição das funções técnicas. A execução por equipe interna ou pelo fundador altera desembolso, mas não elimina esforço, revisão e operação.

| Fase | Horas estimadas | Custo pela faixa hipotética |
|---|---:|---:|
| F0 | 140–220 | R$ 19.600–48.400 |
| F1 | 120–180 | R$ 16.800–39.600 |
| F2 | 180–280 | R$ 25.200–61.600 |
| F3 | 280–420 | R$ 39.200–92.400 |
| F4 | 220–340 | R$ 30.800–74.800 |
| F5 | 260–420 | R$ 36.400–92.400 |
| F6 | 360–560 | R$ 50.400–123.200 |
| F7, por pacote | 240–420 | R$ 33.600–92.400 |
| F8, por capacidade | 360–640 | R$ 50.400–140.800 |

**MVP F0–F3:** 720–1.100 h, R$ 100.800–242.000; com reserva de 20%, **R$ 120.960–290.400**. **Operação ampliada F0–F6:** 1.560–2.420 h, R$ 218.400–532.400; com reserva de 20%, **R$ 262.080–638.880**. F7/F8 não estão nesses totais.

Prazo e horas não são equivalentes: homologação, aprovação de fornecedores e disponibilidade do especialista afetam o calendário. Reservas adicionais, a cotar: revisão jurídica/privacidade R$ 5–20 mil e teste independente de segurança R$ 8–25 mil. Esses valores são provisões propostas, não preços de mercado verificados. Não incluem certificação, regularização de dispositivo médico, tributos da contratação ou pesquisa clínica.

Para reduzir investimento inicial, contratar apenas F0–F2, validar uso com clínicas e só então executar F3. Não reduzir isolamento, backup ou autorização para encaixar módulos extras no mesmo orçamento.

## 19. Custos recorrentes e premissas de orçamento

### 19.1 Infraestrutura cloud

Valores abaixo são **faixas de reserva estimadas**, não soma de preços regionais cotados em calculadora. Referência de consulta: 05/10/2026. Antes da contratação, gerar cotação na região São Paulo com tamanhos, horas, tráfego e impostos; guardar o resultado como anexo deste SDD. Nenhum crédito gratuito foi considerado.

Hipótese do piloto: serviços pequenos, uma zona para banco, 50 GB em objetos, 50 GB de banco provisionado, cerca de 100 GB/mês de saída, até 10 GB/mês de logs sanitizados e homologação reduzida. Hipótese comercial: duas réplicas web e duas de API, worker, RDS Multi-AZ, 100 GB de banco provisionado, 500 GB em objetos, até 500 GB de saída e 30 GB de logs. Retenção e cópias aumentam os volumes faturados.

| Componente mensal | Piloto, US$ | Comercial inicial, US$ |
|---|---:|---:|
| Aplicação web/API/worker em containers | 60–140 | 180–380 |
| PostgreSQL gerenciado e armazenamento básico | 70–150 | 220–450 |
| Balanceador, WAF, rede, NAT/endpoints e IPv4 | 60–140 | 130–280 |
| Objetos, versões, backup e transferência | 15–45 | 50–150 |
| Logs, métricas, chaves, segredos, DNS e autenticação | 25–75 | 60–180 |
| Homologação e execução de CI | 25–70 | 50–140 |
| **Total estimado** | **255–620** | **690–1.580** |

Usando **R$ 5,50/US$ apenas como cenário**, e não cotação cambial, os totais seriam R$ 1.402,50–3.410/mês no piloto e R$ 3.795–8.690/mês na operação comercial. Acrescentar 20% de reserva resulta em aproximadamente **R$ 1.683–4.092** e **R$ 4.554–10.428**, respectivamente. Tributos e variação cambial devem ser calculados conforme contratação. Contingência regional completa e suporte premium de cloud exigem cotação adicional.

Fargate cobra por recursos alocados e tempo; banco e armazenamento têm componentes próprios, e rede pode gerar custos adicionais. Fontes de precificação: [Fargate](https://aws.amazon.com/fargate/pricing/?nc1=h_ls), [RDS PostgreSQL](https://aws.amazon.com/rds/postgresql/pricing/), [S3](https://aws.amazon.com/s3/pricing/), [VPC](https://aws.amazon.com/vpc/pricing/) e [Cognito](https://aws.amazon.com/cognito/pricing/).

### 19.2 Outros custos

| Item | Tratamento no orçamento |
|---|---|
| Domínio | Reserva proposta de R$ 50–150/ano; verificar extensão e disponibilidade |
| Git, editor, bibliotecas e testes locais | Opções sem licença paga; verificar licenças e necessidades comerciais |
| GitHub, design e gestão de projetos | Reserva de R$ 0–800/mês para equipe pequena, conforme planos escolhidos |
| Mensageria | Quantidade faturável × tarifa por país/categoria + taxa do intermediário, se houver |
| OCR, voz e LLM | Páginas/minutos/tokens × tarifa contratada, com cota por clínica |
| Assinatura digital | Custo por documento, usuário ou pacote; cotar conforme fluxo |
| Pagamentos | Tarifa do meio de pagamento, conciliação e eventuais estornos |
| Operação e manutenção | 40–80 h/mês × R$ 140–220 = R$ 5.600–17.600/mês, hipótese sem plantão 24×7 |

Mensagens não têm um custo único universal; validar tabela vigente e condições de categoria/destino. [WhatsApp Business — precificação](https://whatsappbusiness.com/es-la/products/platform-pricing/).

Exemplo de sensibilidade, sem representar tarifa real: 10 mil mensagens a R$ 0,10 geram R$ 1.000; a R$ 0,50, R$ 5.000. Para IA, usar a fórmula: minutos de áudio × preço/minuto + páginas × preço/página + tokens de entrada/saída × tarifas + armazenamento. Habilitar limite rígido e aviso por clínica antes do consumo.

**Custo anual do piloto**, somente cloud com reserva e manutenção da faixa acima: R$ 87.396–260.304. Exclui desenvolvimento, mensagens, IA, jurídico, testes externos, licenças adicionais e tributos. Não confundir custo da infraestrutura com custo total de manter um SaaS.

FinOps: medir custo por clínica e por capacidade; alertas a 50%, 80% e 100% do orçamento; expiração de exportações; limites de upload; retenção de logs; verificação mensal de recursos ociosos. Não usar bloqueio financeiro para impedir exportação e guarda legal de dados.

## 20. Testes, entrega e critérios de liberação

### 20.1 Estratégia de qualidade

- Testes unitários para regras de agenda, retorno, versões, cálculos, estoque e estados.
- Integração com PostgreSQL real para RLS, chaves compostas, transações e concorrência; mocks não demonstram isolamento do banco.
- Testes ponta a ponta das jornadas críticas em desktop, tablet e smartphone, incluindo Safari/iOS e Chrome/Android.
- Contratos e reentrega de webhooks, falha de provedor, repetição de jobs e expiração de links.
- Validação manual de acessibilidade, uso em rede lenta, sessão expirada e interrupção durante salvamento.
- Testes de segurança independentes antes da abertura comercial, mais restauração e carga no perfil declarado.

### 20.2 Pipeline e reversão

Pull request → revisão → análise estática e dependências → testes → build de imagem → homologação → smoke tests → aprovação de release por responsável → produção gradual → observação. Alterações de infraestrutura também passam por revisão. IA de programação pode apoiar desenvolvimento com dados sintéticos e revisão de código, sem receber prontuários reais.

Migrações seguem expansão/contração: adicionar estrutura compatível, migrar dados, trocar leitura/escrita e remover somente em release posterior. O rollback da aplicação deve continuar compatível com o banco; não presumir que toda migração tem reversão destrutiva segura. Mudanças arriscadas usam feature flags e backup verificado.

### 20.3 Definition of Done

História aceita pelo produto, fluxo validado, permissões testadas, isolamento preservado, observabilidade disponível, documentação atualizada e migração/rollback preparados quando necessários. Não liberar falha crítica explorável, vazamento entre clínicas, corrupção de histórico ou incapacidade de restaurar dados. Achados menores exigem responsável e prazo registrados.

Portões de liberação: F2 exige teste de concorrência e controles administrativos; F3 exige aprovação documental/clínica e exportação; F4 exige proteção de mídia; F5/F6 exigem reconciliação; F7 exige falhas de integração testadas; F8 exige avaliação específica e governança de dados/modelos.

## 21. Importação, portabilidade e saída do serviço

Importação inicial por CSV com mapeamento, prévia, relatório de rejeição e identificador de lote. Fazer ensaio com dados sintéticos, backup antes do lote real e conciliação de contagens. Prontuários de sistemas antigos exigem mapeamento específico; ausência de autoria/data deve ficar explícita, sem criar assinatura retroativa.

Exportação por paciente e por organização, conforme permissão: documento legível e pacote estruturado CSV/JSON com anexos, versões, autores e manifesto/checksums. Registrar quem solicitou e baixou. Exportações volumosas são assíncronas, cifradas e temporárias; expiração proposta de 24 horas após geração, configurável.

Na saída de uma clínica, executar exportação conferida, plano de guarda/transferência e encerramento de acessos. A política de eliminação respeita obrigações, litígios e contratos. Testar restauração por tenant em ambiente isolado: um backup compartilhado não permite devolver dados de outras clínicas.

## 22. Indicadores de sucesso e riscos

### 22.1 Indicadores de produto

Medir linha de base no piloto e depois estabelecer metas finais. Hipóteses iniciais: cadastro simples em até 2 minutos; 90% dos atendimentos elegíveis com retorno registrado quando indicado; redução de 20% no tempo administrativo em 60 dias; diminuição de faltas após lembretes. Esses são objetivos a testar, não resultados prometidos.

Definir métricas com denominadores claros: faltas/horários não cancelados; retornos agendados/tarefas de retorno elegíveis; registros finalizados/atendimentos concluídos; horas de operação e incidentes. Evitar usar conclusão de procedimento como indicador automático de qualidade clínica.

### 22.2 Riscos principais

| Risco | Mitigação | Responsável proposto |
|---|---|---|
| Escopo cresce antes da validação | Financiar por fases e manter backlog de descoberta separado | Produto |
| Campos e fluxos inadequados | Homologação semanal com profissional e recepção | Produto + clínico |
| Vazamento entre organizações | Defesa em profundidade e testes negativos sistemáticos | Liderança técnica |
| Perda ou adulteração de prontuário | Imutabilidade, autoria, backups e restore ensaiado | Técnico + clínico |
| Uso de IA com finalidade regulada | Avaliar finalidade de uso antes do desenvolvimento/comercialização | Jurídico/regulatório |
| Custo de mensagens e arquivos cresce | Cotas, precificação por uso e alarmes | Produto + operação |
| Integração indisponível | Fila, repetição segura e operação manual documentada | Técnico |
| Marca indisponível | Busca jurídica e de domínio antes de investir na identidade | Produto |
| Equipe sem plantão para SLA | Adequar SLA, escala e orçamento antes de vender | Operação |

## 23. Decisões a validar na descoberta

1. O lançamento atende profissionais autônomos, clínicas ou ambos? Há múltiplas unidades desde o início?
2. Quais profissões e habilitações estarão no piloto e quem aprova os templates clínicos?
3. Quais trechos ilegíveis da transcrição representam necessidades adicionais?
4. Qual é o fluxo de assinatura/documentação exigido e qual política de guarda será adotada?
5. O sistema substituirá um prontuário existente ou começará por agenda/CRM administrativo?
6. Quantos usuários, pacientes, fotos por atendimento e mensagens por mês são esperados?
7. Qual orçamento e qual equipe estão disponíveis para F0–F3 e para operação?
8. A clínica já possui WhatsApp oficial, provedor de pagamentos e sistema contábil?
9. Quais dados precisam ser migrados e em quais formatos existem?
10. Qual nível de disponibilidade e atendimento de suporte será vendido?

Essas decisões não impedem o desenho proposto, mas podem alterar esforço, contratos e portões de liberação. Registrar mudanças em nova versão do SDD, preservando histórico das decisões.

## 24. Plano de início e entregáveis de F0

**Semana 1:** entrevistar profissional e recepção; validar a transcrição; desenhar cadastro, agenda e retorno; priorizar RF01–RF04; mapear dados, papéis e responsabilidades; selecionar fluxo de assinatura para análise.

**Semana 2:** validar protótipo em celular/tablet/PC; implementar fundação em homologação com dados sintéticos; testar isolamento e restore; registrar versões de ferramentas; produzir cotação regional da cloud; converter F1/F2 em histórias com critérios de aceite.

Ao concluir F0, devem existir: backlog ordenado, protótipo navegável, matriz de acesso, mapa de dados, ADRs, diagrama de implantação, pipeline funcional, evidência de backup/restore, orçamento revisado e responsável definido para cada risco. O desenvolvimento segue por pequenas entregas demonstráveis e aprovações de produto em cada fase.

## 25. Referências e natureza das estimativas

Fontes consultadas em 05/10/2026 e vinculadas às seções correspondentes: legislação LGPD; Lei 13.787/2018; orientações ANPD de incidentes e transferências; material Anvisa sobre SaMD; PostgreSQL RLS; Next.js; NestJS; OWASP ASVS; WCAG 2.2; páginas de preços AWS e WhatsApp Business.

A arquitetura, o cronograma, as metas, as faixas de orçamento, a paleta e os conceitos de marca são propostas deste SDD. A classificação regulatória, a adequação documental, as tarifas regionais, a disponibilidade do nome e os resultados de desempenho dependem de validação antes de contratação ou lançamento. Este documento não afirma certificação, registro de marca, disponibilidade de domínio ou homologação clínica já obtidos.
